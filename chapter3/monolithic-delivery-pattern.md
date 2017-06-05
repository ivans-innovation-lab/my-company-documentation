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
      - mvn cf:push -s .circleci.settings.xml -DskipTests -Dcf.appname=stage-my-company-monolith -Dcf.space=Stage -Dcf.services=sql
  production:
    branch: production
    commands:
      - mvn cf:push -s .circleci.settings.xml -DskipTests -Dcf.appname=prod-my-company-monolith -Dcf.space=Prod -Dcf.services=sql
```

Pipeline contains stages that will run sequentially \(and conditionally\):

* Dependencies - will download all maven dependencies and save them in the cache for latter
* Test - will run `mvn test` 
* Deployment  - will run `mvn deploy`  to upload artifacts on maven repository \(only on master branch\)
* Deployment  - will run mvn cf:push to deploy application on PCF 'stage' environment \(only on master branch\)
* Deployment  - will run mvn cf:push to deploy application on PCF 'production' environment \(only on production branch\)



