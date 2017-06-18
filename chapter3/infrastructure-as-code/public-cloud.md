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

Each maven project/repository defines its own pipeline in a[ circle.yml](https://github.com/ivans-innovation-lab/my-company-monolith/blob/master/circle.yml) file:

* [my-company-common](https://github.com/ivans-innovation-lab/my-company-common)
* [my-company-blog-domain](https://github.com/ivans-innovation-lab/my-company-blog-domain)
* [my-company-project-domain](https://github.com/ivans-innovation-lab/my-company-project-domain)
* [my-company-blog-materialized-view](https://github.com/ivans-innovation-lab/my-company-blog-materialized-view)
* [my-company-project-materialized-view](https://github.com/ivans-innovation-lab/my-company-project-materialized-view)

![](/assets/Screen Shot 2017-06-17 at 11.13.45 AM.png)

Artifacts are deployed on Artifactory instance in the cloud \(AWS hosted\). Parent maven [pom](https://github.com/ivans-innovation-lab/my-company-common/blob/master/pom.xml) file is configured to use this instance with maven profile 'idugalic-cloud'.![](/assets/Screen Shot 2017-06-17 at 2.00.12 PM.png)

### Adopt it

#### For public projects

[CircleCI](https://circleci.com/) offer a total of four free linux containers \($2400 annual value\) for open-source projects. Simply keeping your project public will enable this for you! You can consider [Travis CI](https://travis-ci.org/), it is free for open source as well.

#### For private projects

You can use CircleCI and Travis for private projects, but I would use [Bitbucket](https://bitbucket.org/product) and [Bitbucket Pipelines](https://bitbucket.org/product/features/pipelines). This Atlassian products are mature and very good integrated with Jira. I have to mention that [Gitlab](https://about.gitlab.com/) is coming strong as well. GitLab unifies issues, code review, CI and CD into a single UI. GitLab provides efficient platform for software development and delivery, covering the entire lifecycle from idea to production.

