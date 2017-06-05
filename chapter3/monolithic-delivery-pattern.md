# Monolithic Deployment Pipeline

There are many git branching strategies out there, such as [git flow](http://nvie.com/posts/a-successful-git-branching-model/) and [GitHub flow](http://scottchacon.com/2011/08/31/github-flow.html). I think there is still room for improvement. Git flow is complicated, and not so much in-line with continuous integration. GitHub flow does assume you are able to deploy to production every time you merge a feature branch. This is possible for SaaS applications but there are many cases where this is not possible. One would be a situation where you are not in control of the exact release moment. In these cases you can make a production branch that reflects the deployed code. You can deploy a new version by merging in master to the production branch. If you need to know what code is in production you can just checkout the production branch to see. The approximate time of deployment is easily visible as the merge commit in the version control system. This time is pretty accurate if you automatically deploy your production branch. If you need a more exact time you can have your deployment script create a tag on each deployment. This flow prevents the overhead of releasing, tagging and merging that is common to git flow.

This feature-master-production strategy is known as** **[**Gitlab flow**](https://about.gitlab.com/2014/09/29/gitlab-flow)** **and** **we are going to apply it within this lab. Only in case you need to release software to the outside world you need to work with [release branches](https://docs.gitlab.com/ee/workflow/gitlab_flow.html#release-branches-with-gitlab-flow).

## The scenario '_On-premises'_

Jenkins Pipeline is a suite of plugins which supports implementing and integrating continuous delivery pipelines into Jenkins. Pipeline provides an extensible set of tools for modeling simple-to-complex delivery pipelines "as code" via the [Pipeline DSL](https://jenkins.io/doc/book/pipeline/syntax/).\[[1](https://jenkins.io/doc/book/pipeline/#_footnote_1)\]

Typically, this "Pipeline as Code" would be written to a[`Jenkinsfile`](https://jenkins.io/doc/book/pipeline/jenkinsfile/)and checked into a project’s source control repository:

* [my-company-monolith](https://github.com/ivans-innovation-lab/my-company-monolith/blob/master/Jenkinsfile)

```
pipeline {
    agent any
    tools { 
        maven 'maven-3' 
    }
    stages {
        stage ('Build') {
            when {
                branch 'feature/*'
            }
            steps {
                sh 'mvn clean install'
            }
            post {
                success {
                    junit 'target/surefire-reports/**/*.xml' 
                }
            }
        }
        stage ('Build & Deploy artifact') {
            when {
                branch 'master'
            }
            steps {
                sh 'mvn clean deploy'
            }
            post {
                success {
                    junit 'target/surefire-reports/**/*.xml' 
                }
            }
        }
    }
}
```

Pipeline contains stages that will run sequentially \(and conditionally\):

* Build - will run `mvn install` \(only on feature branches\)
* Build & Deploy artifact - will run `mvn deploy`  \(only on master branch\)

'Build' stage will be triggered for 'feature/\*' branches only. 'Build & Deploy artifact' stage will be triggered for 'master' branch only. We want to deploy artifact on maven repository \(Artifactory\) from 'master' branch only, and make sure that 'production ready' code is deployed.

TODO adopt 'production' branch

## The scenario '_Cloud-based'_

Typically, this "Pipeline as Code" would be written to a [circle.yml](https://github.com/ivans-innovation-lab/my-company-monolith/blob/master/circle.yml) and checked into a project’s source control repository:

* [my-company-monolith](https://github.com/ivans-innovation-lab/my-company-monolith/blob/master/circle.yml)

```
machine:
  services:
    - docker

dependencies:
  pre:
    - curl -v -L -o cf-cli_amd64.deb 'https://cli.run.pivotal.io/stable?release=debian64&source=github'
    - sudo dpkg -i cf-cli_amd64.deb
    - cf -v
  override:
    - mvn dependency:resolve -s .circleci.settings.xml

test:
  override:
    - mvn test -s .circleci.settings.xml
  post:
    - mkdir -p $CIRCLE_TEST_REPORTS/junit/
    - find . -type f -regex ".*/target/surefire-reports/.*xml" -exec cp {} $CIRCLE_TEST_REPORTS/junit/ \;

deployment:
  stage:
    branch: master
    commands:
      - mvn deploy -s .circleci.settings.xml -DskipTests -P idugalic-cloud
#      - mvn docker:build -s .circleci.settings.xml -DpushImage
#      - mvn cf:push -s .circleci.settings.xml -DskipTests -Dcf.appname=stage-my-company-monolith -Dcf.space=Stage -Dcf.services=sql
      - cf api https://api.run.pivotal.io
      - cf auth idugalic@gmail.com $CF_PASSWORD
      - cf target -o idugalic -s Stage
      - cf push stage-my-company-monolith -p target/*.jar --no-start
      - cf bind-service stage-my-company-monolith mysql-stage
      - cf restart stage-my-company-monolith
  production:
    branch: production
    commands:
      - mvn package -s .circleci.settings.xml -DskipTests
#      - mvn cf:push -s .circleci.settings.xml -DskipTests -Dcf.appname=prod-my-company-monolith -Dcf.space=Prod -Dcf.services=sql
      - cf api https://api.run.pivotal.io
      - cf auth idugalic@gmail.com $CF_PASSWORD
      - cf target -o idugalic -s Prod
      - cf push prod-my-company-monolith -p target/*.jar --no-start
      - cf bind-service prod-my-company-monolith mysql-prod
      - cf restart prod-my-company-monolith
      
      

```

Pipeline contains stages that will run sequentially \(and conditionally\):

* Dependencies - will download all maven dependencies and save them in the cache for latter
* Test - will run `mvn test` 
* Deployment  - will run `mvn deploy`  to upload artifact on maven repository \(only on _**master**_ branch\)
* Deployment  - will push the application on PWS 'Stage' environment \(only on _**master**_ branch\)
  * Connect to PWS via CloudFoundry CLI: `cf api https://api.run.pivotal.io`
  * Authenticate to PWS: `cf auth idugalic@gmail.com $CF_PASSWORD`
  * Set Organization to 'idugalic' and space to 'Stage': `cf target -o idugalic -s Stage`
  * Push the application. No start:` cf push stage-my-company-monolith -p target/*.jar --no-start`
  * Bind mysql service to the application \(service have to be created first\):  `cf bind-service stage-my-company-monolith mysql-stage`
  * Start the application: `cf restart stage-my-company-monolith`
* Deployment  - will run `mvn package`  to create artifact in the target folder \(only on _**production**_ branch\)
* Deployment  - will push the application on PWS 'Prod' environment \(only on _**production**_ branch\)
  * Connect to PWS via CloudFoundry CLI: `cf api https://api.run.pivotal.io`
  * Authenticate to PWS: `cf auth idugalic@gmail.com $CF_PASSWORD`
  * Set Organization to 'idugalic' and space to 'Prod': `cf target -o idugalic -s Prod`
  * Push the application. No start:` cf push prod-my-company-monolith -p target/*.jar --no-start`
  * Bind mysql service to the application \(service have to be created first\):  `cf bind-service prod-my-company-monolith mysql-prod`
  * Start the application: `cf restart prod-my-company-monolith`

Every push to master branch will trigger the pipeline and the application will be deployed to PWS on '**Stage**' space:![](/assets/Screen Shot 2017-06-05 at 11.00.03 PM.png)

Once you are ready to deploy to production you should merge master branch into production branch \(you should create pull request for that\). This will trigger the pipeline and the application will be deployed to PWS on '**Prod**' space:![](/assets/Screen Shot 2017-06-05 at 10.59.41 PM.png)For the pipeline to work you have to create two spaces\(environments\) on PWS:

* Stage
* Prod

On each space you have to create instance of ClearDB MySQL service \(database\):

* On Stage space: mysql-stage
* On Prod space: mysql-prod

You can go further, and use [Blue-Green deployment](https://docs.pivotal.io/pivotalcf/1-7/devguide/deploy-apps/blue-green.html) to reduce downtime and risk.





