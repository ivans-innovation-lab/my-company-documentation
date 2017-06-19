## Public cloud

Artifactory, CircleCI and PWS are on the public cloud, and you don't need to install them or to manage them.

* [Artifactory](https://www.jfrog.com/artifactory/) as Maven repository on AWS
  * [http://maven.idugalic.pro](http://maven.idugalic.pro)
* [CircleCI](https://circleci.com/) as continuous integration and delivery** **platform
  * [https://circleci.com/gh/ivans-innovation-lab/my-company-common](https://circleci.com/gh/ivans-innovation-lab/my-company-common)
  * [https://circleci.com/gh/ivans-innovation-lab/my-company-blog-materialized-view](https://circleci.com/gh/ivans-innovation-lab/my-company-blog-materialized-view)
  * [https://circleci.com/gh/ivans-innovation-lab/my-company-project-materialized-view](https://circleci.com/gh/ivans-innovation-lab/my-company-project-materialized-view)
  * [https://circleci.com/gh/ivans-innovation-lab/my-company-blog-domain](https://circleci.com/gh/ivans-innovation-lab/my-company-blog-domain)
  * [https://circleci.com/gh/ivans-innovation-lab/my-company-project-domain](https://circleci.com/gh/ivans-innovation-lab/my-company-project-domain)
* [PWS](http://run.pivotal.io/) - an instance of the Cloud Foundry platform-as-a-_service _ operated by Pivotal Software, Inc.

### Deployment Pipelines

Each maven project/repository defines its own pipeline in a[ circle.yml](https://github.com/ivans-innovation-lab/my-company-monolith/blob/master/circle.yml) file. Pipeline is using maven, and maven settings file \([.circleci.settings.xml](https://github.com/ivans-innovation-lab/my-company-monolith/blob/master/.circleci.settings.xml)\) is included. Make sure that your environment variables \(MAVEN\_PASSWORD, CF\_PASSWORD\) are configured in every 'build' project on CircleCI.![](/assets/Screen Shot 2017-06-19 at 11.23.43 AM.png)

Any project that has circle.yml configured will be build automatically by CircleCi on every push to master branch:

* [my-company-common](https://github.com/ivans-innovation-lab/my-company-common)
* [my-company-blog-domain](https://github.com/ivans-innovation-lab/my-company-blog-domain)
* [my-company-project-domain](https://github.com/ivans-innovation-lab/my-company-project-domain)
* [my-company-blog-materialized-view](https://github.com/ivans-innovation-lab/my-company-blog-materialized-view)
* [my-company-project-materialized-view](https://github.com/ivans-innovation-lab/my-company-project-materialized-view)

![](/assets/Screen Shot 2017-06-17 at 11.13.45 AM.png)

Artifacts are deployed on [Artifactory instance](http://maven.idugalic.pro/artifactory/webapp/#/home) in the cloud \(AWS hosted\). Parent maven [pom](https://github.com/ivans-innovation-lab/my-company-common/blob/master/pom.xml) file is configured to use this instance with maven profile 'idugalic-cloud'. Make sure that your parent maven pom file is configured by your needs. You have to change every [.circleci.settings.xml](https://github.com/ivans-innovation-lab/my-company-monolith/blob/master/.circleci.settings.xml) accordingly.![](/assets/Screen Shot 2017-06-17 at 2.00.12 PM.png)

### Adopt it

#### For open projects

[CircleCI](https://circleci.com/) offer a total of four free linux containers \($2400 annual value\) for open-source projects. Simply keeping your project public will enable this for you! You can consider [Travis CI](https://travis-ci.org/), it is free for open source as well.

#### For closed projects

You can use:

* CircleCI and Travis for private projects
* [Bitbucket](https://bitbucket.org/product) and [Bitbucket Pipelines](https://bitbucket.org/product/features/pipelines). Atlassian products are mature and very good integrated. 
* [Gitlab](https://about.gitlab.com/) unifies issues, code review, CI and CD into a single UI. GitLab provides efficient platform for software development and delivery, covering the entire lifecycle from idea to production.

##### Gitlab

The Gitlab has proven as best option by my opinion _\(the best value for the price - for closed projects in the public cloud\)._

* I have cloned Github organization to Gitlab group \(all repos included\)
* I added CI pipeline \(.gitlab-ci.yml\) for every project/repo \(used shared runners and created my [specific runners](https://docs.gitlab.com/ee/ci/runners/README.html) as well\)
* Configured secret \(env\) variables \(MAVEN\_PASSWORD, CF\_PASSWORD\) for every build in Gitlab CI settings
* All green in 1 hour.

![](/assets/Screen Shot 2017-06-19 at 11.58.12 AM.png)

Example of my-company-monolith pipeline \([.gitlab-ci.yml](https://gitlab.com/snippets/1665449)\):

```
image: docker:latest
services:
  - docker:dind

stages:
  - build
  - deploy

maven-build:
  image: maven:3-jdk-8
  stage: build
  script:
    - "mvn package -s .circleci.settings.xml -B"
  artifacts:
    paths:
      - target/*.jar

maven-deploy:
  image: maven:3-jdk-8
  stage: deploy
  script: "mvn -s .gitlab-ci.settings.xml -DskipTests deploy -P idugalic-cloud"
  only:
    - master

# Master branch will be deployed to PWS(CloudFoundry) on "Stage" env. Deployment will be triggered on every push to master branch.
# It is required to have
#   - organzation 'idugalic' created on PWS
#   - space 'Stage' created within 'idugalic' organization on PWS
# NOTE: CF_PASSWORD is a secret variable. You have to set it on Gitlab (Settings->CI/CD Pipelines->Secret Variables->Add Variable)
cf-deploy-stage:
  image: governmentpaas/cf-cli:latest
  stage: deploy
  script:
    - cf api https://api.run.pivotal.io
    - cf auth idugalic@gmail.com $CF_PASSWORD
    - cf target -o idugalic -s Stage
    - cf push stage-my-company-monolith -p target/*.jar --no-start
    - cf bind-service stage-my-company-monolith mysql-stage
    - cf restart stage-my-company-monolith
  environment:
    name: staging
    url: https://stage-my-company-monolith.cfapps.io
  only:
    - master

# The 'when': manual action exposes a play button in GitLab's UI and the cf-deploy-prod job will only be triggered if and when we click that play button.
# https://docs.gitlab.com/ce/ci/environments.html#manually-deploying-to-environments
cf-deploy-prod:
  image: governmentpaas/cf-cli:latest
  stage: deploy
  script:
    - cf api https://api.run.pivotal.io
    - cf auth idugalic@gmail.com $CF_PASSWORD
    - cf target -o idugalic -s Prod
    - cf push prod-my-company-monolith -p target/*.jar --no-start
    - cf bind-service prod-my-company-monolith mysql-prod
    - cf restart prod-my-company-monolith
  environment:
    name: production
    url: https://prod-my-company-monolith.cfapps.io
  when: manual
  only:
    - master
```

![](/assets/Screen Shot 2017-06-19 at 11.18.50 PM.png)![](/assets/Screen Shot 2017-06-19 at 11.23.49 PM.png)

