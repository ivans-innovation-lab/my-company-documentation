## Public cloud

This "Pipeline as Code" is written to a [.circleci/config.yml](https://github.com/ivans-innovation-lab/my-company-monolith/blob/master/.circleci/config.yml) and checked into a project’s source control repository:

```
defaults: &defaults
  working_directory: /home/circleci/my-company-monolith
  docker:
    - image: circleci/openjdk:8-jdk-browsers

version: 2
jobs:
  # Build and test with maven
  build:
    <<: *defaults
    steps:

      - checkout
      # Caching is one of the most effective ways to make jobs faster on CircleCI.
      # Downloading from a Remote Repository in Maven is triggered by a project declaring a dependency that is not present in the local repository (or for a SNAPSHOT, when the remote repository contains one that is newer).
      # Do not overwrite your release (not snapshots) artifacts (my-company-blog-domain, my-company-blog-materialized-view, my-company-project-domain, my-company-project-materialized-view) on remote maven repository, othervise the cache will become stale.
      - restore_cache:
          key: my-company-monolith1-{{ checksum "pom.xml" }}

      - run: 
          name: Install maven artifact
          command: |
            if [ "${CIRCLE_BRANCH}" != "master" ]; then
              mvn -s .circleci/maven.settings.xml install -P idugalic-cloud
            fi

      - deploy:
          name: Deploy maven artifact
          command: |
            if [ "${CIRCLE_BRANCH}" == "master" ]; then
              mvn -s .circleci/maven.settings.xml deploy -P idugalic-cloud
            fi

      - save_cache:
          paths:
            - ~/.m2
          key: my-company-monolith1-{{ checksum "pom.xml" }}

      - run:
          name: Collecting test results
          command: |
            mkdir -p junit/
            find . -type f -regex ".*/target/surefire-reports/.*xml" -exec cp {} junit/ \;
          when: always

      - store_test_results:
          path: junit/

      - store_artifacts:
          path: junit/

      - run:
          name: Collecting artifacts
          command: |
            mkdir -p artifacts/
            find . -type f -regex ".*/target/.*jar" -exec cp {} artifacts/ \;

      - store_artifacts:
          path: artifacts/

      - persist_to_workspace:
          root: artifacts/
          paths:
            - .
  # Deploying build artifact on staging environment for testing.
  staging:
    <<: *defaults
    steps:
      - attach_workspace:
          at: workspace/
      - run:
          name: Install CloudFoundry CLI
          command: |
            curl -v -L -o cf-cli_amd64.deb 'https://cli.run.pivotal.io/stable?release=debian64&source=github'
            sudo dpkg -i cf-cli_amd64.deb
            cf -v
      - deploy:
          name: Deploy to Staging - PWS CLoudFoundry (CF_PASSWORD variable required)
          command: |
            cf api https://api.run.pivotal.io
            cf auth idugalic@gmail.com $CF_PASSWORD
            cf target -o idugalic -s Stage
            cf push stage-my-company-monolith -p workspace/*.jar --no-start
            cf bind-service stage-my-company-monolith mysql
            cf restart stage-my-company-monolith

  # Deploying current production on staging environemnt for DB schema backward compatibility testing.
  staging-prod:
    <<: *defaults
    steps:
      - attach_workspace:
          at: workspace/

      - run:
          name: Install CloudFoundry CLI
          command: |
            curl -v -L -o cf-cli_amd64.deb 'https://cli.run.pivotal.io/stable?release=debian64&source=github'
            sudo dpkg -i cf-cli_amd64.deb
            cf -v

      - run: 
          name: Install AWS CLI
          command: sudo apt-get update && sudo apt-get install -y awscli

      - run:
          name: Download latest production artifact from AWS S3 (AWS Permissions on CircleCI required)
          command: aws s3 sync s3://my-company-production . --delete --region eu-central-1

      - deploy:
          name: Deploy latest production application to Staging - PWS CLoudFoundry (CF_PASSWORD variable required)
          command: |
            if [ -e "./my-company-monolith-1.0.0-SNAPSHOT.jar" ]; then
              cf api https://api.run.pivotal.io
              cf auth idugalic@gmail.com $CF_PASSWORD
              cf target -o idugalic -s Stage
              cf push stage-my-company-monolith-prod -p ./*.jar --no-start
              cf bind-service stage-my-company-monolith-prod mysql
              cf restart stage-my-company-monolith-prod 
            else 
              echo "Artifact does not exist, deploying current build"
              cf api https://api.run.pivotal.io
              cf auth idugalic@gmail.com $CF_PASSWORD
              cf target -o idugalic -s Stage
              cf push stage-my-company-monolith-prod -p workspace/*.jar --no-start
              cf bind-service stage-my-company-monolith-prod mysql
              cf restart stage-my-company-monolith-prod
            fi 


  # A very simple e2e test on staging environemnt
  staging-e2e:
    <<: *defaults
    steps:
      - run: 
          name: End to end test on Staging
          command: |
            curl -I "https://stage-my-company-monolith.cfapps.io/health"
            curl -I "https://stage-my-company-monolith.cfapps.io/api/blogposts"
            curl -I "https://stage-my-company-monolith.cfapps.io/api/projects"

  # A very simple e2e test of the currnet production application on staging environment.
  # DB schema backward compatibility testing. Testing if the latest DB schema is compatible with latest production application.
  staging-prod-e2e:
    <<: *defaults
    steps: 
      - run: 
          name: End to end test on Staging of current production
          command: |
            curl -I "https://stage-my-company-monolith-prod.cfapps.io/health"
            curl -I "https://stage-my-company-monolith-prod.cfapps.io/api/blogposts"
            curl -I "https://stage-my-company-monolith-prod.cfapps.io/api/projects"

  # Deploying build artifact on production environment with Blue-Green deployment strategy and the rollback posibility.
  # Artifact is uploaded to AWS s3 as the latest production artifact as well. It is used within 'staging-prod' and 'staging-prod-e2e' jobs to test DB schema backward compatibility
  production:
    <<: *defaults
    steps:
      - attach_workspace:
          at: workspace/

      - run:
          name: Install CloudFoundry CLI
          command: |
            curl -v -L -o cf-cli_amd64.deb 'https://cli.run.pivotal.io/stable?release=debian64&source=github'
            sudo dpkg -i cf-cli_amd64.deb
            cf -v

      - run: 
          name: Install AWS CLI
          command: sudo apt-get update && sudo apt-get install -y awscli

      - deploy:
          name: Deploy to Production - PWS CLoudFoundry (CF_PASSWORD variable required)
          command: |
            cf api https://api.run.pivotal.io
            cf auth idugalic@gmail.com $CF_PASSWORD
            cf target -o idugalic -s Prod

            cf push prod-my-company-monolith-B -p workspace/*.jar --no-start
            cf bind-service prod-my-company-monolith-B mysql
            cf restart prod-my-company-monolith-B
            # Map Original Route to Green
            cf map-route prod-my-company-monolith-B cfapps.io -n prod-my-company-monolith
            # Unmap temp Route to Green
            cf unmap-route prod-my-company-monolith-B cfapps.io -n prod-my-company-monolith-B
            # Unmap Route to Blue (only if Blue exist)
            cf app prod-my-company-monolith && cf unmap-route prod-my-company-monolith cfapps.io -n prod-my-company-monolith
            # Stop and rename current Blue in case you need to roll back your changes (only if Blue exist)
            cf app prod-my-company-monolith && cf stop prod-my-company-monolith
            cf app prod-my-company-monolith && cf rename prod-my-company-monolith prod-my-company-monolith-old
            # Rename Green to Blue
            cf rename prod-my-company-monolith-B prod-my-company-monolith

      - deploy:
          name: Upload latest production artifact to AWS S3 (AWS Permissions on CircleCI required)
          command: aws s3 sync workspace/ s3://my-company-production --delete --region eu-central-1


workflows:
  version: 2
  my-company-monolith-workflow:
    jobs:
      - build
      - staging:
          requires:
            - build
          filters:
            branches:
              only: master
      - staging-prod:
          requires:
            - staging
          filters:
            branches:
              only: master
      - staging-e2e:
          requires:
            - staging
          filters:
            branches:
              only: master
      - staging-prod-e2e:
          requires:
            - staging-prod
          filters:
            branches:
              only: master
      - approve-production:
          type: approval
          requires:
            - staging-e2e
            - staging-prod-e2e
          filters:
            branches:
              only: master
      - production:
          requires:
            - approve-production
          filters:
            branches:
              only: master
```

The following example shows a [workflow](https://circleci.com/gh/ivans-innovation-lab/workflows/my-company-monolith) with seven sequential jobs. The jobs run according to configured requirements, each job waiting to start until the required job finishes successfully. This workflow is configured to wait for manual approval of a job 'approve-production' before continuing by using the`type: approval`key. The`type: approval`key is a special job that is only** **added under your`workflow`key

![](/assets/Screen Shot 2017-08-04 at 1.13.39 PM.png)

#### Stage

Every push to **master** branch \(every time you merge a feature branch\) will trigger the pipeline and the application will be deployed to PWS on '**Stage**' space:![](/assets/Screen Shot 2017-06-21 at 1.28.42 PM.png)Additionally, a current production artifact will be deployed on Stage by 'staging-prod' job for DB schema backward compatibility testing \(we will test old application against new DB schema\)

#### Production

Once you are ready to deploy to **production** you should manually approve deployment to production in you CircleCI workflow. This will trigger next job \(production\) and the application will be deployed to PWS on '**Prod**' space:![](/assets/Screen Shot 2017-06-21 at 1.28.58 PM.png)Additionally we will upload an artifact to AWS S3 so we can test DB schema backward compatibility on Stage environment.

### Requirements

For the pipeline to work you have to create two spaces \(environments\) on PWS:

* Stage
* Prod

On each space you have to create instance of ClearDB MySQL service \(database\):

```
cf api https://api.run.pivotal.io
cf auth EMAIL PASSWORD
cf target -o idugalic -s Stage
cf create-service cleardb spark mysql
cf t -s Prod
cf create-service cleardb spark mysql
```

NOTE: Instructions to install CloudFoundry CLI \(cf\): [https://docs.cloudfoundry.org/cf-cli/install-go-cli.html](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html)

### Metrics

PCF Metrics helps you understand and troubleshoot the health and performance of your apps by displaying the following:

* [Container Metrics](http://docs.run.pivotal.io/metrics/using.html#container)
   A graph of CPU, memory, and disk usage percentages
* [Network Metrics](http://docs.run.pivotal.io/metrics/using.html#network)
   A graph of requests, HTTP errors, and response times
* [App Events](http://docs.run.pivotal.io/metrics/using.html#events)
   A graph of update, start, stop, crash, SSH, and staging failure events
* [Logs](http://docs.run.pivotal.io/metrics/using.html#logs)
   A list of app logs that you can search, filter, and download
* [Trace Explorer](http://docs.run.pivotal.io/metrics/using.html#trace)
   A graph that traces a request as it flows through your apps and their endpoints, along with the corresponding logs

![](/assets/Screen Shot 2017-06-16 at 11.45.31 AM.png)

### Autoscaler

[App Autoscaler](https://docs.run.pivotal.io/appsman-services/autoscaler/using-autoscaler.html) is a marketplace service that ensures app performance and helps control the cost of running apps.

To balance app performance and cost, Space Developers and Space Managers can use App Autoscaler to do the following:

* Configure rules that adjust instance counts based on metrics thresholds such as CPU Usage
* Modify the maximum and minimum number of instances for an app, either manually or following a schedule

### Spring Boot Actuator

Adding Actuator to your Spring Boot application deployed on Pivotal Cloud Foundry gets you the following production-ready features:

* Health Check column & expanded information in Instances section
* git commit id indicator, navigable to your git repo
* Summary git info under Settings tab \(also navigable to repo\)
* Runtime adjustment of logging levels, exposed via Actuator endpoints
* Heap Dump\*
* View Trace\*
* View Threads, dump/download for further analysis\*

### Blue-Green Deployment

[Blue-green deployment](https://docs.run.pivotal.io/devguide/deploy-apps/blue-green.html) is a release technique that reduces downtime and risk by running two identical production environments called Blue and Green.

At any time, only one of the environments is live, with the live environment serving all production traffic. For this example, Blue is currently live and Green is idle.

As you prepare a new release of your software, deployment and the final stage of testing takes place in the environment that is\_not\_live: in this example, Green. Once you have deployed and fully tested the software in Green, you switch the router so all incoming requests now go to Green instead of Blue. Green is now live, and Blue is idle.

This technique can eliminate downtime due to application deployment. In addition, blue-green deployment reduces risk: if something unexpected happens with your new release on Green, you can immediately roll back to the last version by switching back to Blue.

Blue-green deployment is implemented by 'production' job in the [workflow](https://github.com/ivans-innovation-lab/my-company-monolith/blob/master/.circleci/config.yml).

Doing Blue-green deployment with database schema changing is not easy. We have to change the schema in such a way that Blue-green deployment and roll back to the last version are possible, usually by making DB changes backward compatible. For this we need schema versioning first \([Flyway](http://flywaydb.org/)\). I was inspired with this [blog post](https://spring.io/blog/2016/05/31/zero-downtime-deployment-with-a-database). There you can find more details.

You can design your database in the 6th normal form an make you scheme more adaptable and your process more agile. I was inspired with this [blog post](https://blog.codecentric.de/en/2017/07/agile-database-design-using-anchor-modeling/ ).

