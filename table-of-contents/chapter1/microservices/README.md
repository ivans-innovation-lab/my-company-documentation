# Microservice Architecture

We started with the Monolithic pattern first, and the [drawbacks ](../monolithic/)of this approach have overrun the [benefits](../monolithic/).

After one year in production \(with monolithic pattern\) it is very difficult to:

* scale application
* scale development
* long-term commitment to a technology stack is a problem at this point \(we need No SQL database for some of the parts\)
* continuous deployment is difficult

## Benefits

This microservices architecture has a number of benefits:

* Each microservice is relatively small
  * Easier for a developer to understand
  * The IDE is faster making developers more productive
  * The application starts faster, which makes developers more productive, and speeds up deployments
* Each service can be deployed independently of other services - easier to deploy new versions of services frequently
* Easier to scale development. It enables you to organize the development effort around multiple teams. Each \(two pizza\) team is owns and is responsible for one or more single service. Each team can develop, deploy and scale their services independently of all of the other teams.
* Improved fault isolation. For example, if there is a memory leak in one service then only that service will be affected. The other services will continue to handle requests. In comparison, one misbehaving component of a monolithic architecture can bring down the entire system.
* Each service can be developed and deployed independently
* Eliminates any long-term commitment to a technology stack. When developing a new service you can pick a new technology stack. Similarly, when making major changes to an existing service you can rewrite it using a new technology stack.

## Drawbacks

The microservices architecture has a number of drawbacks:

* Developers must deal with the additional complexity of creating a distributed system.
  * Developer tools/IDEs are oriented on building monolithic applications and don’t provide explicit support for developing distributed applications.
  * Testing is more difficult
  * Developers must implement the inter-service communication mechanism.
  * Implementing use cases that span multiple services without using distributed transactions is difficult
  * Implementing use cases that span multiple services requires careful coordination between the teams
* Deployment complexity. In production, there is also the operational complexity of deploying and managing a system comprised of many different service types.
* Increased memory consumption. The microservice architecture replaces N monolithic application instances with NxM services instances. If each service runs in its own JVM \(or equivalent\), which is usually necessary to isolate the instances, then there is the overhead of M times as many JVM runtimes. Moreover, if each service runs on its own VM \(e.g. EC2 instance\), as is the case at Netflix, the overhead is even higher.

