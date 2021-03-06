# Chapter 3 - Process

The idea of automated deployment is important. Indeed, if you take automating the deployment process to its logical conclusion, you could push every build that passes the necessary automated tests into production. The practice of automatically deploying every successful build directly into production is generally known as Continuous Deployment.

However, a pure Continuous Deployment approach is not always appropriate for everyone. For example, many users would not appreciate new versions falling into their laps several times a week, and prefer a more predictable \(and transparent\) release cycle. Commercial and marketing considerations might also play a role in when a new release should actually be deployed. The notion of Continuous Delivery is a variation on the idea of Continuous Deployment that takes into account these considerations.

Two biggest enablers to continuous delivery are:

* architecture, 
* and organizational structure and culture.![](../../.gitbook/assets/monolithictomicroservices.jpg)

## Process and Architecture

In the context of architecture there are typically multiple attributes we are concerned about, for example availability, security, performance, usability and so forth. In continuous delivery, we introduce two new architectural attributes: _testability_ and _deployability_.

In a _testable_ architecture, we design our software such that most defects can \(in principle, at least\) be discovered by developers by running automated tests on their workstations. We shouldn’t need to depend on complex, integrated environments in order to do the majority of our acceptance and regression testing.

In a _deployable_ architecture, deployments of a particular product or service can be performed independently and in a fully automated fashion, without the need for significant levels of orchestration. Deployable systems can typically be upgraded or reconfigured with zero or minimal downtime.

Where testability and deployability are not prioritized, we find that much testing requires the use of complex, integrated environments, and deployments are “big bang” events that require that many services are released at the same time due to complex interdependencies. These “big bang” deployments require many teams to work together in a carefully orchestrated fashion with many hand-offs, and dependencies between hundreds or thousands of tasks. Such deployments typically take many hours or even days, and require scheduling significant downtime.

Designing for testability and deployability starts with ensuring our products and services are composed of loosely-coupled, well-encapsulated components or modules \(in the context of object-oriented programming, such systems follow the principles of [SOLID design](https://en.wikipedia.org/wiki/SOLID_%28object-oriented_design%29)\).

We can define a well-designed modular architecture as one in which it is possible to test or deploy a single component or service on its own, with any dependencies replaced by a suitable [test double](http://martinfowler.com/bliki/TestDouble.html), which could be in the form of a virtual machine, a stub, or a mock. Each component or service should be deployable in a fully automated fashion on developer workstations, test environments, or in production. In a well-designed architecture, it is possible to get a high level of confidence the component is operating properly when deployed in this fashion.

## Process and Organization

Westrum’s research emphasizes the importance of creating a culture where new ideas are welcomed, people from across the organization collaborate in the pursuit of common goals, where we train people to bring bad news so we can act on it, and where failures and accidents are treated as opportunities to learn how to improve rather than witch-hunts.

The DevOps movement has always emphasized the primary importance of culture, with a particular focus on effective collaboration between development teams and IT operations teams. Research shows that a win-win relationship between development and ops is a significant predictor of IT performance. Practitioners in the DevOps movement have also used a number of tools to help organizations process information more effectively, such as [chatops](https://www.youtube.com/watch?v=NST3u-GjjFw), [blameless post-mortems](http://www.slideshare.net/danmil30/how-to-run-a-postmortem-with-humans-not-robots-velocity-2013), and [comprehensive configuration management](https://continuousdelivery.com/foundations/configuration-management/).

Indeed the highest-performing companies don’t wait for bad things to happen in order to learn how to improve, they [create \(controlled\) accidents on a regular basis](http://queue.acm.org/detail.cfm?id=2371516)so as to learn more quickly than the competition. Netflix took this to a new level with their Simian Army, which is constantly breaking their infrastructure in order to continuously test the resilience of their systems.

The common theme that can be observed in high performing organizations is that they are always trying to get better. Rather than waiting for the environment to shift around them or comparing themselves against others, [they treat themselves as the standard to beat](https://continuousdelivery.com/2013/01/on-antifragility-in-systems-and-organizational-architecture/).

## Delivery Pipeline

The key pattern introduced in continuous delivery is the _deployment pipeline._

In the deployment pipeline pattern, every change in version control triggers a process \(usually in a [CI](https://continuousdelivery.com/foundations/continuous-integration/) server\) which creates deployable packages and runs automated unit tests and other validations such as static code analysis. This first step is optimized so that it takes only a few minutes to run. If this initial \_commit stage \_fails, the problem must be fixed immediately—nobody should check in more work on a broken commit stage. Every passing commit stage triggers the next step in the pipeline, which might consist of a more comprehensive set of automated tests. Versions of the software that pass all the automated tests can then be deployed on demand to further stages such as exploratory testing, performance testing, staging, and production, as shown below.

![](../../.gitbook/assets/pipeline-branching.png)

Deployment pipelines tie together configuration management, continuous integration and test and deployment automation in a holistic, powerful way that works to improve software quality, increase stability, and reduce the time and cost required to make incremental changes to software, whatever domain you’re operating in. When building a deployment pipeline, we’ve found the following practices valuable:

* **Only build packages once**

  We want to be sure the thing we’re deploying is the same thing we’ve tested throughout the deployment pipeline, so if a deployment fails we can eliminate the packages as the source of the failure.

* **Deploy the same way to every environment**—including development. This way, we test the deployment process many, many times before it gets to production, and again, we can eliminate it as the source of any problems.
* **Smoke test your deployments**

  Have a script that validates all your application’s dependencies are available, at the location you have configured your application. Make sure your application is running and available as part of the deployment process.

* **Keep your environments similar**

  Although they may differ in hardware configuration, they should have the same version of the operating system and middleware packages, and they should be configured in the same way. This has become much easier to achieve with modern virtualization and container technology.

With the advent of infrastructure as code, it has became possible to use deployment pipelines to create a fully automated process for taking all kinds of changes—including database and infrastructure changes—from version control into production in a controlled, repeatable and auditable way.

