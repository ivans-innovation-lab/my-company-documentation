# Monolithic Deployment Pipeline

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

## The scenario '_Cloud-based'_

Typically, this "Pipeline as Code" would be written to a [circle.yml](https://github.com/ivans-innovation-lab/my-company-monolith/blob/master/circle.yml) and checked into a project’s source control repository:

* [my-company-monolith](https://github.com/ivans-innovation-lab/my-company-monolith/blob/master/circle.yml)

```
machine:
  services:
    - docker

dependencies:
  override:
    - mvn dependency:resolve -s .circleci.settings.xml

test:
  override:
    - mvn test -s .circleci.settings.xml
  post:
    - mkdir -p $CIRCLE_TEST_REPORTS/junit/
    - find . -type f -regex ".*/target/surefire-reports/.*xml" -exec cp {} $CIRCLE_TEST_REPORTS/junit/ \;

deployment:
  master:
    branch: master
    commands:
      - mvn deploy -s .circleci.settings.xml -DskipTests -P idugalic-cloud
      - mvn docker:build -s .circleci.settings.xml -DpushImage
      
```

Pipeline contains stages that will run sequentially \(and conditionally\):

* Dependencies - will download all maven dependencies and save them in the cache for latter
* Test - will run `mvn test` 
* Deployment  - will run `mvn deploy`  to upload artifacts on maven repository \(only on master branch\)
* Deployment  - will run `mvn docker:build` to build and push docker image to public docker hub \(only on master branch\)



