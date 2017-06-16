## Private cloud

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
        stage ('Build & Test') {
            steps {
                sh 'mvn clean build'
            }
            post {
                success {
                    junit 'target/surefire-reports/**/*.xml' 
                }
            }
        }
        stage ('Deploy to Stage') {
            when {
                branch 'master'
            }
            steps {
               sh 'mvn deploy -DskipTests'
               sh 'cf api https://api.local.pcfdev.io'
               sh 'cf auth user pass
               sh 'cf target -o pcfdev-org -s pcfdev-stage'
               sh 'cf push stage-my-company-monolith -p target/*.jar --no-start'
               sh 'cf bind-service stage-my-company-monolith mysql-stage'
               sh 'cf restart stage-my-company-monolith'
            }
        }
        stage ('Deploy to Production') {
            when {
                branch 'production'
            }
            steps {
               sh 'mvn package -DskipTests'
               sh 'cf api https://api.local.pcfdev.io'
               sh 'cf auth user pass
               sh 'cf target -o pcfdev-org -s pcfdev-prod'
               sh 'cf push prod-my-company-monolith -p target/*.jar --no-start'
               sh 'cf bind-service prod-my-company-monolith mysql-prod'
               sh 'cf restart prod-my-company-monolith'
            }

        }
    }
}
```

Pipeline contains stages that will run sequentially \(and conditionally\):

* Build & Test - will run `mvn verify`  \(on any branch\)
* Deploy to Stage  - will run `mvn deploy`  to upload artifact on maven repository \(only on _**master**_ branch\)
* Deploy to Stage  - will push the application on 'pcfdev-stage' environment \(only on _**master**_ branch\)
  * Connect to PCF via CloudFoundry CLI: `cf api https://api.local.pcfdev.io`
  * Authenticate to PCF: `cf auth user pass`
  * Set Organization to '`pcfdev-org`' and space to 'pcfdev-stage': `cf target -o pcfdev-org -s pcfdev-stage`
  * Push the application. No start:`cf push stage-my-company-monolith -p target/*.jar --no-start`
  * Bind mysql service to the application \(service have to be created first\):  `cf bind-service stage-my-company-monolith mysql-stage`
  * Start the application: `cf restart stage-my-company-monolith`
* Deploy to Production  - will run `mvn package`  to create artifact in the target folder \(only on _**production**_ branch\)
* Deploy to Production  - will push the application on 'pcfdev-prod' environment \(only on _**production**_ branch\)
  * Connect to PCF via CloudFoundry CLI: `cf api https://api.local.pcfdev.io`
  * Authenticate to PCF: `cf auth user pass`
  * Set Organization to '`pcfdev-org`' and space to 'pcfdev-prod': `cf target -o idugalic -s Prod`
  * Push the application. No start:`cf push prod-my-company-monolith -p target/*.jar --no-start`
  * Bind mysql service to the application \(service have to be created first\):  `cf bind-service prod-my-company-monolith mysql-prod`
  * Start the application: `cf restart prod-my-company-monolith`

Every push to **master** branch will trigger the pipeline and the application will be deployed to PCF on '**pcfdev-stage**' space:![](/assets/Screen Shot 2017-06-16 at 8.52.20 PM.png)

Once you are ready to deploy to **production** you should **merge master branch into production branch** \(you should create pull request for that\). This will trigger the pipeline and the application will be deployed to PCF on '**pcfdev-prod**' space:![](/assets/Screen Shot 2017-06-16 at 9.02.42 PM.png)

### Requirements

For the pipeline to work you have to create two spaces\(environments\) on PCF:

* pcfdev-stage
* pcfdev-prod

On each space you have to create instance of PCF Dev MySQL service \(database\):

* On pcfdev-stage space: mysql-stage
* On pcfdev-prod space: mysql-prod



