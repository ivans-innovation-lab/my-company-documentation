# Public Cloud

Every service has a pipeline/workflow:

* [https://circleci.com/gh/ivans-innovation-lab/workflows/my-company-project-materialized-view-microservice](https://circleci.com/gh/ivans-innovation-lab/workflows/my-company-project-materialized-view-microservice)
* [https://circleci.com/gh/ivans-innovation-lab/workflows/my-company-project-domain-microservice](https://circleci.com/gh/ivans-innovation-lab/workflows/my-company-project-domain-microservice)
* [https://circleci.com/gh/ivans-innovation-lab/workflows/my-company-blog-materialized-view-microservice](https://circleci.com/gh/ivans-innovation-lab/workflows/my-company-blog-materialized-view-microservice)
* [https://circleci.com/gh/ivans-innovation-lab/workflows/my-company-blog-domain-microservice](https://circleci.com/gh/ivans-innovation-lab/workflows/my-company-blog-domain-microservice)
* [https://circleci.com/gh/ivans-innovation-lab/workflows/my-company-api-gateway-backingservice](https://circleci.com/gh/ivans-innovation-lab/workflows/my-company-api-gateway-backingservice/)
* [https://circleci.com/gh/ivans-innovation-lab/workflows/my-company-adminserver-backingservice](https://circleci.com/gh/ivans-innovation-lab/workflows/my-company-adminserver-backingservice)
* [https://circleci.com/gh/ivans-innovation-lab/workflows/my-company-registry-backingservice](https://circleci.com/gh/ivans-innovation-lab/workflows/my-company-registry-backingservice)
* [https://circleci.com/gh/ivans-innovation-lab/workflows/my-company-configuration-backingservice](https://circleci.com/gh/ivans-innovation-lab/workflows/my-company-configuration-backingservice)

This "Pipeline as Code" is written to a [.circleci/config.yml](https://github.com/ivans-innovation-lab/my-company-project-materialized-view-microservice/blob/master/.circleci/config.yml) and checked into a project’s source control repository:

```text
defaults: &defaults
  working_directory: /home/circleci/my-company-project-materialized-view-microservice
  docker:
    - image: circleci/openjdk:8-jdk-browsers

version: 2
jobs:
  build:
    <<: *defaults
    steps:

      - checkout

      - restore_cache:
          key: my-company-project-materialized-view-microservice-{{ checksum "pom.xml" }}

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
          key: my-company-project-materialized-view-microservice-{{ checksum "pom.xml" }}

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
      - run:
          name: Deploy to Staging - PWS CLoudFoundry
          command: |
            cf api https://api.run.pivotal.io
            cf auth idugalic@gmail.com $CF_PASSWORD
            cf target -o idugalic -s Stage
            cf push stage-my-company-project-materialized-view-microservice -p workspace/*.jar --no-start
            cf bind-service stage-my-company-project-materialized-view-microservice my-company-configuration-backingservice
            cf bind-service stage-my-company-project-materialized-view-microservice my-company-registry-backingservice
            cf bind-service stage-my-company-project-materialized-view-microservice mysql-stage
            cf bind-service stage-my-company-project-materialized-view-microservice rabbit-stage
            cf restart stage-my-company-project-materialized-view-microservice


  # A very simple e2e test    
  staging-e2e:
    <<: *defaults
    steps:
      - attach_workspace:
          at: workspace/

      - run: 
          name: End to end test on Staging
          command: curl -i https://stage-my-company-project-materialized-view-microservice.cfapps.io/health


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
          name: Deploy to Production - PWS CLoudFoundry
          command: |
            cf api https://api.run.pivotal.io
            cf auth idugalic@gmail.com $CF_PASSWORD
            cf target -o idugalic -s Prod
            cf push prod-my-company-project-materialized-view-microservice -p workspace/*.jar --no-start
            cf bind-service prod-my-company-project-materialized-view-microservice my-company-configuration-backingservice
            cf bind-service prod-my-company-project-materialized-view-microservice my-company-registry-backingservice
            cf bind-service prod-my-company-project-materialized-view-microservice mysql-prod
            cf bind-service prod-my-company-project-materialized-view-microservice rabbit-prod
            cf restart prod-my-company-project-materialized-view-microservice


workflows:
  version: 2
  my-company-project-materialized-view-microservice-workflow:
    jobs:
      - build
      - staging:
          requires:
            - build
          filters:
            branches:
              only: master
      - staging-e2e:
          requires:
            - staging
          filters:
            branches:
              only: master
      - approve-production:
          type: approval
          requires:
            - staging-e2e
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

The following example shows a [workflow](https://circleci.com/gh/ivans-innovation-lab/workflows/my-company-project-materialized-view-microservice) with five sequential jobs. The jobs run according to configured requirements, each job waiting to start until the required job finishes successfully. This workflow is configured to wait for manual approval of a job 'approve-production' before continuing by using the`type: approval`key. The`type: approval`key is a special job that is only ****added under your`workflow`key![](../../../.gitbook/assets/screen-shot-2017-07-17-at-9.03.13-am.png)

### Stage

Every push to **master** branch \(every time you merge a feature branch\) will trigger the pipeline and the applications will be deployed to PWS on '**Stage**' space:![](../../../.gitbook/assets/screen-shot-2017-07-16-at-9.53.39-pm.png)

### Production

Once you are ready to deploy to **production** you should manually approve deployment to production in your CircleCI workflow. This will trigger next job \(production\), and the application will be deployed to PWS on '**Prod**' space

![](../../../.gitbook/assets/screen-shot-2017-07-17-at-2.19.24-pm.png)

## Requirements

For the pipeline to work you have to create two spaces \(environments\) on PWS:

* Stage
* Prod

Deploy backing-services first:

1. [my-company-configuration-backingservice](https://github.com/ivans-innovation-lab/my-company-configuration-repository)
2. [my-company-registry-backingservice](https://github.com/ivans-innovation-lab/my-company-registry-backingservice)

On each space you have to create instance of ClearDB MySQL service \(database\), cloudamqp \(RabbitMQ\), [my-company-configuration-backingservice](https://circleci.com/gh/ivans-innovation-lab/workflows/my-company-configuration-backingservice), [my-company-registry-backingservice](https://circleci.com/gh/ivans-innovation-lab/workflows/my-company-registry-backingservice):

```text
cf api https://api.run.pivotal.io
cf auth EMAIL PASSWORD

cf target -o idugalic -s Stage
cf create-user-provided-service my-company-configuration-backingservice -p '{"uri":"https://stage-my-company-configuration-backingservice.cfapps.io"}'
cf create-user-provided-service my-company-registry-backingservice -p '{"uri":"https://stage-my-company-registry-backingservice.cfapps.io"}'
cf create-service cleardb spark mysql-stage
cf create-service cloudamqp lemur rabbit-stage


cf t -s Prod
cf create-user-provided-service my-company-configuration-backingservice -p '{"uri":"https://prod-my-company-configuration-backingservice.cfapps.io"}'
cf create-user-provided-service my-company-registry-backingservice -p '{"uri":"https://prod-my-company-registry-backingservice.cfapps.io"}'
cf create-service cleardb spark mysql-prod
cf create-service cloudamqp lemur rabbit-prod
```

## ![](../../../.gitbook/assets/screen-shot-2017-07-16-at-10.04.32-pm.png)Metrics

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

![](../../../.gitbook/assets/screen-shot-2017-07-17-at-11.36.58-am.png)

## Autoscaler

[App Autoscaler](https://docs.run.pivotal.io/appsman-services/autoscaler/using-autoscaler.html) is a marketplace service that ensures app performance and helps control the cost of running apps.

To balance app performance and cost, Space Developers and Space Managers can use App Autoscaler to do the following:

* Configure rules that adjust instance counts based on metrics thresholds such as CPU Usage
* Modify the maximum and minimum number of instances for an app, either manually or following a schedule

## Spring Boot Actuator

Adding Actuator to your Spring Boot application deployed on Pivotal Cloud Foundry gets you the following production-ready features:

* Health Check column & expanded information in Instances section
* git commit id indicator, navigable to your git repo
* Summary git info under Settings tab \(also navigable to repo\)
* Runtime adjustment of logging levels, exposed via Actuator endpoints
* Heap Dump\*
* View Trace\*
* View Threads, dump/download for further analysis\*

![](../../../.gitbook/assets/screen-shot-2017-07-17-at-2.25.20-pm.png)![](../../../.gitbook/assets/screen-shot-2017-07-17-at-2.26.33-pm.png)![](../../../.gitbook/assets/screen-shot-2017-07-17-at-2.28.36-pm.png)![](../../../.gitbook/assets/screen-shot-2017-07-17-at-2.30.09-pm.png)

## Blue-Green Deployment

[Blue-green deployment](https://docs.run.pivotal.io/devguide/deploy-apps/blue-green.html) is a release technique that reduces downtime and risk by running two identical production environments called Blue and Green.

At any time, only one of the environments is live, with the live environment serving all production traffic. For this example, Blue is currently live and Green is idle.

As you prepare a new release of your software, deployment and the final stage of testing takes place in the environment that is\_not\_live: in this example, Green. Once you have deployed and fully tested the software in Green, you switch the router so all incoming requests now go to Green instead of Blue. Green is now live, and Blue is idle.

This technique can eliminate downtime due to application deployment. In addition, blue-green deployment reduces risk: if something unexpected happens with your new release on Green, you can immediately roll back to the last version by switching back to Blue.

Blue-green deployment is not implemented in this lab. You should consider it.

