# Infrastructure as code

This chapter will lead you through installing an instance of Jenkins and Artifactory on a system. For this purposes we will utilize Docker to run containers with Jenkins and Artifactory already configured.

* Jenkins is the open source continuous integration server
* Artifactory is the open source maven repository
* Docker is an open platform for developers and sysadmins to build, ship, and run distributed applications, whether on laptops, data center VMs, or the cloud.

You can find the source code here: https://github.com/ivans-innovation-lab/my-company-infrastructure

### The scenario:

* all jobs are under version control and described via [Job-DSL](https://github.com/jenkinsci/job-dsl-plugin/wiki), see the [my-company-ci-jobs](https://github.com/ivans-innovation-lab/my-company-ci-jobs) repo
* there is a [seed-job](https://github.com/ivans-innovation-lab/my-company-infrastructure/blob/master/seedJob.xml) which runs periodically to ensure the aforementioned multi-branch jobs exist in Jenkins
* each project to be built by these jobs defines its own pipeline via [Pipeline-DSL](https://jenkins.io/doc/book/pipeline/syntax/) in a`Jenkinsfile,(`Pipeline as Code`)` see the:
  * [my-company-monolith](https://github.com/ivans-innovation-lab/my-company-monolith)
  * [my-company-common](https://github.com/ivans-innovation-lab/my-company-common)
  * [my-company-blog-domain](https://github.com/ivans-innovation-lab/my-company-blog-domain)
  * [my-company-project-domain](https://github.com/ivans-innovation-lab/my-company-project-domain)
  * [my-company-blog-materialized-view](https://github.com/ivans-innovation-lab/my-company-blog-materialized-view)
  * [my-company-project-materialized-view](https://github.com/ivans-innovation-lab/my-company-project-materialized-view)

* artifacts are deployed on Artifactory instance. Parrent maven [pom](https://github.com/ivans-innovation-lab/my-company-common/blob/master/pom.xml) file is configured to use the Artifactory.

## Running instructions

```
$ ./start.sh gituser gitpassword

```

Once Jenkins is started you should see at least the seed-job on [http://localhost:9090](http://localhost:9090/).

If it has not run yet, simply trigger it and see how the actual jobs/pipelines get created.

Artifactory is available on [http://localhost:9091](http://localhost:9091/)

Please note that this scenario can be adopted to your needs:

* You can use some other virtualization tool
* You want to save Jenkins configuration \(docker volumes\)
* You want to create Jenkins slaves
* You like more Nexus then Artifactory
* You don't want to create and manage multi-branch jobs by yourself \([my-company-ci-jobs](https://github.com/ivans-innovation-lab/my-company-ci-jobs)\). You want to use [Github Organization Folder Plugin](https://github.com/jenkinsci/github-organization-folder-plugin)
* You want to use Groove \(script\) style of Jenkinsfile rather then declarative style.

  




