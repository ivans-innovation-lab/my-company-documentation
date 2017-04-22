# Monolithic Deployment Pipeline

Jenkins Pipeline is a suite of plugins which supports implementing and integrating continuous delivery pipelines into Jenkins. Pipeline provides an extensible set of tools for modeling simple-to-complex delivery pipelines "as code" via the [Pipeline DSL](https://jenkins.io/doc/book/pipeline/syntax/).\[[1](https://jenkins.io/doc/book/pipeline/#_footnote_1)\]

Typically, this "Pipeline as Code" would be written to a[`Jenkinsfile`](https://jenkins.io/doc/book/pipeline/jenkinsfile/)and checked into a project’s source control repository, for example:

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
* _Deploy to Cloud Foundry - test environment_
* _Deploy to Cloud Foundry - staging environment_
* _Deploy to Cloud Foundry - production environment_

'Build' stage will be triggered for 'feature/\*' branches only. Build & Deploy artifact will be triggered for 'master' branch only. We want to deploy artifact on maven repository \(Artifactory\) from 'master' branch only, and make sure that 'production ready' code is deployed.

_TODO - deployment to Cloud Foundry_

