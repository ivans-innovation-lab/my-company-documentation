# Architecture Patterns And Best Practices

A [**pattern language**](https://en.wikipedia.org/wiki/Pattern_language)** **is a method of describing good design practices or patterns of useful organization within a field of expertise.

![](/assets/MicroservicePatternLanguage.jpg)

## The Architecture Pattern Language

The architecture pattern language consists of numerous groups of patterns. **The value of a pattern language exceeds the sum of it’s individual patterns because it defines these relationships between the patterns:**

* Predecessor – a predecessor pattern is a pattern that motivates the need for this pattern. For example, the Microservice Architecture pattern is the predecessor to the rest of the patterns in the pattern language except the monolithic architecture pattern.

* Successor – a pattern that solves an issue that is introduced by this pattern. For example, if you apply the Microservice Architecture pattern you must then apply numerous successor patterns including service discovery patterns and the Circuit Breaker pattern.

* Alternative – a pattern that provides an alternative solution to this pattern. For example, the Monolithic Architecture pattern and the Microservice Architecture pattern are alternative ways of architecting an application. You pick one or the other. These relationships provide valuable guidance when using a pattern language. Applying a pattern creates issues that you must then address by applying successor patterns. The selection of patterns continuously recursively until you reach patterns with no successor. If two or more patterns are alternatives then you must typically pick just one. In many ways, this is similar to traversing a graph.

### Apply The Microservice Architecture Pattern Language - Example

Let’s look at how you can apply the microservice architecture pattern language to architect your application. We will look at 3 critical decisions you must make.

1. The first decision you must make is whether to use a [Monolithic architecture pattern](http://microservices.io/patterns/monolithic.html) or the [Microservice architecture pattern](http://microservices.io/patterns/microservices.html). **If you pick the Microservice architecture pattern you must choose numerous other patterns to deal with the consequences of your decision.**

2. If you have decided to use the microservice architecture you must define your services. There are two main options,

   * [Decompose by business capability](http://microservices.io/patterns/decomposition/decompose-by-business-capability.html) – define services corresponding to business capabilities
   * [Decompose by subdomain](http://microservices.io/patterns/decomposition/decompose-by-subdomain.html) – define services corresponding to DDD subdomains

   This patterns yield equivalent results: a set of services organized around business concepts rather than technical concepts.

3. A key feature of the microservice is the [Database per Service pattern](http://microservices.io/patterns/data/database-per-service.html). It’s alternative, the [Shared Database pattern](http://microservices.io/patterns/data/shared-database.html) is essentially an anti-pattern and best avoided. The Database per service pattern dramatically changes how you maintain data consistency and perform queries. You will need to use one of [event-driven architecture](http://microservices.io/patterns/data/event-driven-architecture.html) patterns such as [Transaction Log tailing](http://microservices.io/patterns/data/transaction-log-tailing.html) or [Event Sourcing](http://microservices.io/patterns/data/event-sourcing.html) to maintain data consistency. You will often need to implement queries using the [Command Query Responsibility Segregation \(CQRS\)](http://microservices.io/patterns/data/cqrs.html) pattern.

Visit the [Microservices.io](http://microservices.io/) for more details about architecture patterns and how to apply them. It is brought to you by Chris Richardson.



