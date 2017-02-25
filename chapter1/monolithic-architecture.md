# Monolithic Architecture

Once we understand the architecture pattern language and we know how to visualize our architecture, we can apply a pattern language to architect a concrete application.

[Monolithic Architecture pattern](http://microservices.io/patterns/monolithic.html) and the [Microservice Architecture pattern](http://microservices.io/patterns/microservices.html) are alternative ways of architecting an application. You pick one or the other.

I strongly believe that one should start with Monolithic pattern first, and once the drawbacks of this approach overrun the benefits you should switch to Microservices pattern. For this to happen smoothly you should design your monolith in a way so you can easily switch to microservices:

* Make your components more loosely coupled. 
* Consider applying Domain Driven Design with Event Sourcing and CQRS.

## Benefits of a monolithic architecture

* Simple to develop - the goal of current development tools and IDEs is to support the development of monolithic applications
* Simple to deploy - you simply need to deploy the WAR file \(or directory hierarchy\) on the appropriate runtime
* Simple to scale - you can scale the application by running multiple copies of the application behind a load balancer

## Drawbacks of a monolithic architecture

* The large monolithic code base intimidates developers, especially ones who are new to the team. The application can be difficult to understand and modify. As a result, development typically slows down. Also, because there are not hard module boundaries, modularity breaks down over time. Moreover, because it can be difficult to understand how to correctly implement a change the quality of the code declines over time. It’s a downwards spiral.
* Continuous deployment is difficult - a large monolithic application is also an obstacle to frequent deployments. In order to update one component you have to redeploy the entire application. This will interrupt background tasks \(e.g. Quartz jobs in a Java application\), regardless of whether they are impacted by the change, and possibly cause problems. There is also the chance that components that haven’t been updated will fail to start correctly. As a result, the risk associated with redeployment increases, which discourages frequent updates. This is especially a problem for user interface developers, since they usually need to iterative rapidly and redeploy frequently.
* Scaling the application can be difficult - a monolithic architecture is that it can only scale in one dimension. On the one hand, it can scale with an increasing transaction volume by running more copies of the application. Some clouds can even adjust the number of instances dynamically based on load. But on the other hand, this architecture can’t scale with an increasing data volume. Each copy of application instance will access all of the data, which makes caching less effective and increases memory consumption and I/O traffic. Also, different application components have different resource requirements - one might be CPU intensive while another might memory intensive. With a monolithic architecture we cannot scale each component independently
* Obstacle to scaling development - A monolithic application is also an obstacle to scaling development. Once the application gets to a certain size its useful to divide up the engineering organization into teams that focus on specific functional areas. For example, we might want to have the UI team, accounting team, inventory team, etc. The trouble with a monolithic application is that it prevents the teams from working independently. The teams must coordinate their development efforts and redeployments. It is much more difficult for a team to make a change and update production.
* Requires a long-term commitment to a technology stack - a monolithic architecture forces you to be married to the technology stack \(and in some cases, to a particular version of that technology\) you chose at the start of development . With a monolithic application, can be difficult to incrementally adopt a newer technology. For example, let’s imagine that you chose the JVM. You have some language choices since as well as Java you can use other JVM languages that inter-operate nicely with Java such as Groovy and Scala. But components written in non-JVM languages do not have a place within your monolithic architecture. Also, if your application uses a platform framework that subsequently becomes obsolete then it can be challenging to incrementally migrate the application to a newer and better framework. It’s possible that in order to adopt a newer platform framework you have to rewrite the entire application, which is a risky undertaking.



