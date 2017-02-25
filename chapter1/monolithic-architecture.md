# Monolithic Architecture

Once we understand the architecture pattern language and we know how to visualize our architecture, we can apply a pattern language to architect an concrete application.

[Monolithic Architecture pattern](http://microservices.io/patterns/monolithic.html) and the [Microservice Architecture pattern](http://microservices.io/patterns/microservices.html) are alternative ways of architecting an application. You pick one or the other.

I strongly believe that one should start with Monolithic pattern first, and once the drawbacks of this approach overruns the benefits you should switch to Microservices pattern. For this to happen smoothly you should design your monolith in a way so you can easily switch to microservices: 

* Make your components more loosely coupled. 
* Consider applying Domain Driven Design with Event Sourcing and CQRS.

## Key Benefits of a monolithic architecture

* Simple to develop - the goal of current development tools and IDEs is to support the development of monolithic applications
* Simple to deploy - you simply need to deploy the WAR file \(or directory hierarchy\) on the appropriate runtime
* Simple to scale - you can scale the application by running multiple copies of the application behind a load balancer

## Key Drawbacks of a monolithic architecture







