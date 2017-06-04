# Microservices Deployment Pipeline

## The scenario '_On-premises'_

Typically, this "Pipeline as Code" would be written to a[`Jenkinsfile`](https://jenkins.io/doc/book/pipeline/jenkinsfile/)and checked into a project’s source control repository:

TODO

## The scenario '_Cloud-based'_

Typically, this "Pipeline as Code" would be written to a circle.yml and checked into a project’s source control repositories:

* [my-company-blog-domain-microservice](https://github.com/ivans-innovation-lab/my-company-blog-domain-microservice/blob/master/circle.yml) / [build](https://circleci.com/gh/ivans-innovation-lab/my-company-blog-domain-microservice)
* [my-company-blog-materialized-view-microservice](https://github.com/ivans-innovation-lab/my-company-blog-materialized-view-microservice/blob/master/circle.yml) / [build](https://circleci.com/gh/ivans-innovation-lab/my-company-blog-materialized-view-microservice)
* [my-company-project-domain-microservice](https://github.com/ivans-innovation-lab/my-company-project-domain-microservice/blob/master/circle.yml) / [build](https://circleci.com/gh/ivans-innovation-lab/my-company-project-domain-microservice)
* [my-company-project-materialized-view-microservice](https://github.com/ivans-innovation-lab/my-company-project-materialized-view-microservice/blob/master/circle.yml) / [build](https://circleci.com/gh/ivans-innovation-lab/my-company-project-materialized-view-microservice)
* [my-company-configuration-backingservice](https://github.com/ivans-innovation-lab/my-company-configuration-backingservice/blob/master/circle.yml) / [build](https://circleci.com/gh/ivans-innovation-lab/my-company-configuration-backingservice)
* [my-company-registry-backingservice](https://github.com/ivans-innovation-lab/my-company-registry-backingservice/blob/master/circle.yml)  / [build](https://circleci.com/gh/ivans-innovation-lab/my-company-registry-backingservice)
* [my-company-api-gateway-backingservice](https://github.com/ivans-innovation-lab/my-company-api-gateway-backingservice/blob/master/circle.yml) / [build](https://circleci.com/gh/ivans-innovation-lab/my-company-api-gateway-backingservice)
* [my-company-adminserver-backingservice](https://github.com/ivans-innovation-lab/my-company-adminserver-backingservice/blob/master/circle.yml) / [build](https://circleci.com/gh/ivans-innovation-lab/my-company-adminserver-backingservice)

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



