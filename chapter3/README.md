# Chapter 3 - Process

The idea of automated deployment is important. Indeed, if you take automating the deployment process to its logical conclusion, you could push every build that passes the necessary automated tests into production. The practice of automatically deploying every successful build directly into production is generally known as Continuous Deployment.

However, a pure Continuous Deployment approach is not always appropriate for everyone. For example, many users would not appreciate new versions falling into their laps several times a week, and prefer a more predictable \(and transparent\) release cycle. Commercial and marketing considerations might also play a role in when a new release should actually be deployed. The notion of Continuous Delivery is a variation on the idea of Continuous Deployment that takes into account these considerations.

Two biggest enablers to continuous delivery are:

* architecture, 
* and organizational structure and culture.![](/assets/monolithicToMicroservices.jpg)

In the context of architecture there are typically multiple attributes we are concerned about, for example availability, security, performance, usability and so forth. In continuous delivery, we introduce two new architectural attributes: _testability_ and _deployability_.

In a _testable_ architecture, we design our software such that most defects can \(in principle, at least\) be discovered by developers by running automated tests on their workstations. We shouldn’t need to depend on complex, integrated environments in order to do the majority of our acceptance and regression testing.

In a _deployable_ architecture, deployments of a particular product or service can be performed independently and in a fully automated fashion, without the need for significant levels of orchestration. Deployable systems can typically be upgraded or reconfigured with zero or minimal downtime.

Where testability and deployability are not prioritized, we find that much testing requires the use of complex, integrated environments, and deployments are “big bang” events that require that many services are released at the same time due to complex interdependencies. These “big bang” deployments require many teams to work together in a carefully orchestrated fashion with many hand-offs, and dependencies between hundreds or thousands of tasks. Such deployments typically take many hours or even days, and require scheduling significant downtime.

Designing for testability and deployability starts with ensuring our products and services are composed of loosely-coupled, well-encapsulated components or modules \(in the context of object-oriented programming, such systems follow the principles of [SOLID design](https://en.wikipedia.org/wiki/SOLID_%28object-oriented_design%29)\).

We can define a well-designed modular architecture as one in which it is possible to test or deploy a single component or service on its own, with any dependencies replaced by a suitable [test double](http://martinfowler.com/bliki/TestDouble.html), which could be in the form of a virtual machine, a stub, or a mock. Each component or service should be deployable in a fully automated fashion on developer workstations, test environments, or in production. In a well-designed architecture, it is possible to get a high level of confidence the component is operating properly when deployed in this fashion.

