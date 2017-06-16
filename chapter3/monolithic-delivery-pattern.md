# Monolithic Deployment Pipeline

## The scenario - Private cloud

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

## The scenario - Public cloud

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
  * Push the application. No start:`cf push stage-my-company-monolith -p target/*.jar --no-start`
  * Bind mysql service to the application \(service have to be created first\):  `cf bind-service stage-my-company-monolith mysql-stage`
  * Start the application: `cf restart stage-my-company-monolith`
* Deployment  - will run `mvn package`  to create artifact in the target folder \(only on _**production**_ branch\)
* Deployment  - will push the application on PWS 'Prod' environment \(only on _**production**_ branch\)
  * Connect to PWS via CloudFoundry CLI: `cf api https://api.run.pivotal.io`
  * Authenticate to PWS: `cf auth idugalic@gmail.com $CF_PASSWORD`
  * Set Organization to 'idugalic' and space to 'Prod': `cf target -o idugalic -s Prod`
  * Push the application. No start:`cf push prod-my-company-monolith -p target/*.jar --no-start`
  * Bind mysql service to the application \(service have to be created first\):  `cf bind-service prod-my-company-monolith mysql-prod`
  * Start the application: `cf restart prod-my-company-monolith`

Every push to **master** branch will trigger the pipeline and the application will be deployed to PWS on '**Stage**' space:![](/assets/Screen Shot 2017-06-05 at 11.00.03 PM.png)

Once you are ready to deploy to **production** you should **merge master branch into production branc**h \(you should create pull request for that\). This will trigger the pipeline and the application will be deployed to PWS on '**Prod**' space:![](/assets/Screen Shot 2017-06-05 at 10.59.41 PM.png)

### Requirements

For the pipeline to work you have to create two spaces\(environments\) on PWS:

* Stage
* Prod

On each space you have to create instance of ClearDB MySQL service \(database\):

* On Stage space: mysql-stage
* On Prod space: mysql-prod

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

### Blue-Green Deployment

[Blue-green deployment](https://docs.run.pivotal.io/devguide/deploy-apps/blue-green.html) is a release technique that reduces downtime and risk by running two identical production environments called Blue and Green.

At any time, only one of the environments is live, with the live environment serving all production traffic. For this example, Blue is currently live and Green is idle.

As you prepare a new release of your software, deployment and the final stage of testing takes place in the environment that is\_not\_live: in this example, Green. Once you have deployed and fully tested the software in Green, you switch the router so all incoming requests now go to Green instead of Blue. Green is now live, and Blue is idle.

This technique can eliminate downtime due to application deployment. In addition, blue-green deployment reduces risk: if something unexpected happens with your new release on Green, you can immediately roll back to the last version by switching back to Blue.

