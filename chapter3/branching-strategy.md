# Branching Strategy

There are many git branching strategies out there, such as [git flow](http://nvie.com/posts/a-successful-git-branching-model/) and [github flow](http://scottchacon.com/2011/08/31/github-flow.html). I think there is still room for improvement. Git flow is complicated, and not so much in-line with continuous integration. GitHub flow does assume you are able to deploy to production every time you merge a feature branch. This is possible for SaaS applications but there are many cases where this is not possible. One would be a situation where you are not in control of the exact release moment. In these cases you can make a manual step in you pipeline to deploy your artifacts to production environment or maven repository.

* [Monolithic Deployment Pipeline](/chapter3/monolithic-delivery-pattern.md) is using [github flow](http://scottchacon.com/2011/08/31/github-flow.html) with addition of manual step in pipeline to control production deployment.
* [Microservices Deployment Pipeline](/chapter3/microservices-delivery-pattern.md) is using [github flow](http://scottchacon.com/2011/08/31/github-flow.html) with addition of manual step in pipeline to control production deployment.
* Core libraries have to be released to internal teams. This libraries will be used by monolithic application or misroservices. In this case we are using [github flow](https://www.gitbook.com/book/ivans-innovation-lab/my-company/edit#). Every push to master \(every time you merge a feature branch\) will trigger deployment of artifacts to maven repository.



