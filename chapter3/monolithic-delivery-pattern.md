# Monolithic Deployment Pipeline

Jenkins Pipeline is a suite of plugins which supports implementing and integrating continuous delivery pipelines into Jenkins. Pipeline provides an extensible set of tools for modeling simple-to-complex delivery pipelines "as code" via the [Pipeline DSL](https://jenkins.io/doc/book/pipeline/syntax/).\[[1](https://jenkins.io/doc/book/pipeline/#_footnote_1)\]

Typically, this "Pipeline as Code" would be written to a[`Jenkinsfile`](https://jenkins.io/doc/book/pipeline/jenkinsfile/)and checked into a project’s source control repository, for example:

* [my-company-monolith](https://github.com/ivans-innovation-lab/my-company-monolith)
* [my-company-common](https://github.com/ivans-innovation-lab/my-company-common)
* [my-company-blog-domain](https://www.gitbook.com/book/ivans-innovation-lab/my-company/edit#)
* [my-company-project-domain](https://www.gitbook.com/book/ivans-innovation-lab/my-company/edit#)
* [my-company-blog-materialized-view](https://www.gitbook.com/book/ivans-innovation-lab/my-company/edit#)
* [my-company-project-materialized-view](https://www.gitbook.com/book/ivans-innovation-lab/my-company/edit#)

[https://github.com/ivans-innovation-lab/my-company-monolith/edit/master/Jenkinsfile](https://github.com/ivans-innovation-lab/my-company-monolith/edit/master/Jenkinsfile)

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
                script{
                    sh 'git config --global user.email "idugalic@gmail.com"'
                    sh 'git config --global user.name "jenkins"'
                    def pom = readMavenPom file: 'pom.xml' 
                    def version = pom.version.replace("-SNAPSHOT", ".${currentBuild.number}")
                    sh "mvn -DreleaseVersion=${version} -DdevelopmentVersion=${pom.version} release:clean release:prepare release:perform -B"
                }
            }
        }
    }
}
```

TODO - deployment to Cloud Foundry 

