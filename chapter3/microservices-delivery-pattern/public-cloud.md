## Public cloud

This "Pipeline as Code" is written to a [.circleci/config.yml](https://github.com/ivans-innovation-lab/my-company-project-materialized-view-microservice/blob/master/.circleci/config.yml) and checked into a project’s source control repository:

```
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

The following example shows a [workflow](https://circleci.com/gh/ivans-innovation-lab/workflows/my-company-project-materialized-view-microservice) with five sequential jobs. The jobs run according to configured requirements, each job waiting to start until the required job finishes successfully. This workflow is configured to wait for manual approval of a job 'approve-production' before continuing by using the`type: approval`key. The`type: approval`key is a special job that is only** **added under your`workflow`key![](/assets/Screen Shot 2017-07-17 at 9.03.13 AM.png)

#### Stage

Every push to **master** branch \(every time you merge a feature branch\) will trigger the pipeline and the applications will be deployed to PWS on '**Stage**' space:![](/assets/Screen Shot 2017-07-16 at 9.53.39 PM.png)

#### Production

Once you are ready to deploy to **production** you should manually approve deployment to production in you CircleCI workflow. This will trigger next job \(production\) and the application will be deployed to PWS on '**Prod**' space

### Requirements

For the pipeline to work you have to create two spaces \(environments\) on PWS:

* Stage
* Prod

On each space you have to create instance of ClearDB MySQL service \(database\), cloudamqp \(RabbitMQ\), my-company-configuration-backingservice, my-company-registry-backingservice:

```
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

### ![](/assets/Screen Shot 2017-07-16 at 10.04.32 PM.png)



