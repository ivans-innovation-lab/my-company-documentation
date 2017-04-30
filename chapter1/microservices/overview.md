# Microservice Architecture Overview

## Patterns and techniques

Patterns/techniques that are used:

- Domain Driven Design
- Command and Query Responsibility Separation (CQRS)
- Event Sourcing
- Microservice Architecture
- Service instance per Container
- Externalized configuration
- API gateway
- Client-side discovery
- Service registry
- Circuit Breaker
- Access Token
- Application metrics


## System Context

<iframe src="https://structurizr.com/embed/22311?diagram=Context&diagramSelector=false" width="600" height="425" marginwidth="0" marginheight="0" frameborder="0" scrolling="no" allowfullscreen="true"></iframe>

We are developing a server-side enterprise application. It will support a variety of different clients including desktop browsers, mobile browsers and native mobile applications. This clients will enable employees to manage blog posts, projects information, customers and other data, and it will enable customers to browse the news and submit requests for new interesting projects. The application will also expose an API for 3rd parties (partners) to consume and support B2B. It will also integrate with other systems (Github, LinkedIn, Twitter) via web services to enrich and to share relevant data with them.

## Containers

<iframe src="https://structurizr.com/embed/22311?diagram=Containers&diagramSelector=false" width="600" height="425" marginwidth="0" marginheight="0" frameborder="0" scrolling="no" allowfullscreen="true"></iframe>

The domain is literally split into a *command-side* microservice application/container and a *query-side* microservice application/container (this is CQRS in its most literal form).

Communication between the two microservices is `event-driven` and the demo/lab uses RabbitMQ messaging as a means of passing the events between processes (VM's).

The **command-side** processes commands. Commands are actions which change state in some way. The execution of these commands results in `Events` being generated which are persisted by Axon  and propagated out to other VM's (as many VM's as you like) via RabbitMQ messaging. In event-sourcing, events are the sole records in the system. They are used by the system to describe and re-build aggregates on demand, one event at a time.

The **query-side** is an event-listener and processor. It listens for the `Events` and processes them in whatever way makes the most sense. In this application, the query-side just builds and maintains a *materialised view* which tracks the state of the individual agregates (Product, Blog, Customer, ...). The query-side can be replicated many times for scalability and the messages held by the RabbitMQ queues are durable, so they can be temporarily stored on behalf of the event-listener if it goes down.

The command-side and the query-side containers both have REST API's which can be used to access their capabilities.

### [Backend Microservices](https://github.com/search?q=topic%3Amicroservice+org%3Aivans-innovation-lab&type=Repositories)

![](/assets/microservices.png)

While the backing services in the middle layer are still considered to be microservices, they solve a set of concerns that are purely operational and security-related. The business logic of this application sits almost entirely in our bottom layer.

#### [Blog Microservices](https://github.com/search?utf8=%E2%9C%93&q=topic%3Amicroservice+topic%3Ablog+org%3Aivans-innovation-lab&type=Repositories)

Two Blog micro-services are used for managing (command) and querying the blog posts: 

- Command side: https://github.com/ivans-innovation-lab/my-company-blog-materialized-view-microservice
- Query side: https://github.com/ivans-innovation-lab/my-company-blog-domain-microservice



#### [Project Microservices](https://github.com/search?utf8=%E2%9C%93&q=topic%3Amicroservice+topic%3Aproject+org%3Aivans-innovation-lab&type=Repositories)

Two Project micro-services are used for managing (command) and querying the projects:

- Command side: https://github.com/ivans-innovation-lab/my-company-project-materialized-view-microservice
- Query side: https://github.com/ivans-innovation-lab/my-company-project-domain-microservice


### [Backing services](https://github.com/search?q=topic%3Abacking-service+org%3Aivans-innovation-lab&type=Repositories)

The premise is that there are third-party service dependencies that should be treated as attached resources to your cloud native applications. The key trait of backing services are that they are provided as bindings to an application in its deployment environment by a cloud platform. Each of the backing services must be located using a statically defined route

#### [Admin server](https://github.com/ivans-innovation-lab/my-company-adminserver-backingservice)

Spring Boot Admin is a simple application to manage and monitor your Spring Boot services. The services are discovered using Spring Cloud (e.g. Eureka). The UI is just an Angular.js application on top of the Spring Boot Actuator endpoints. In case you want to use the more advanced features (e.g. jmx-, loglevel-management), Jolokia must be included in the client services.

- https://github.com/ivans-innovation-lab/my-company-adminserver-backingservice

#### [Registry (Eureka)](https://github.com/ivans-innovation-lab/my-company-registry-backingservice)

Netflix Eureka is a service registry. It provides a REST API for service instance registration management and for querying available instances. Netflix Ribbon is an IPC client that works with Eureka to load balance(client side) requests across the available service instances.

- https://github.com/ivans-innovation-lab/my-company-registry-backingservice

#### [Authorization server (Oauth2)](https://github.com/ivans-innovation-lab/my-company-authserver-backingservice)

For issuing tokens and authorize requests.

- https://github.com/ivans-innovation-lab/my-company-authserver-backingservice

#### [Configuration server](https://github.com/ivans-innovation-lab/my-company-configuration-backingservice)

The configuration service is a vital component of any microservices architecture. Based on the twelve-factor app methodology, configurations for your microservice applications should be stored in the environment and not in the project.

- https://github.com/ivans-innovation-lab/my-company-configuration-backingservice
- [Configuration repository](https://github.com/ivans-innovation-lab/my-company-configuration-repository)

### Databases

Each micro-service can use one database instance for that service only. This can be easily configured:

 - One database instance for all services (simple - hard to scale)
 - One database for all 'command side' microservices, and one database per 'query side' service (not so complicated - scales better)
 - One database per service (complicated - scales good)
 
### Message broker

 - [RabbitMQ](https://www.rabbitmq.com/) to pass events between micro-services. We'll deliver a event to multiple consumers. This pattern is known as "publish/subscribe". 'Command side' service will send events to exchange. Each 'query side' service will bind a queue to that exchange and receive (process) events in that way. Events are durable and this is making the system more (highly) available and fault isolated. The 'query  side' doesn't have to be online, as long as events are durable. Once the 'query side' is online it will process all events in concrete queue with correct order.
 

## Components

Each of the containers (microservices) is constructed of two main components:
- domain/materialized view component
- web (REST) component

### Project

#### Command side
[commandSideProjectComponent](https://github.com/ivans-innovation-lab/my-company-project-domain)+commandSideProjectWebComponent=[commandSideProjectMicroservice](https://github.com/ivans-innovation-lab/my-company-project-domain-microservice)

#### Query side
[querySideProjectComponent](https://github.com/ivans-innovation-lab/my-company-project-materialized-view)+querySideProjectWebComponent=[querySideProjectMicroservice](https://github.com/ivans-innovation-lab/my-company-project-materialized-view-microservice)

### Blog

#### Command side
[commandSideBlogComponent](https://github.com/ivans-innovation-lab/my-company-blog-domain)+commandSideBlogWebComponent=[commandSideBlogMicroservice](https://github.com/ivans-innovation-lab/my-company-blog-domain-microservice)

#### Query side
[querySideBlogComponent](https://github.com/ivans-innovation-lab/my-company-blog-materialized-view)+querySideBlogWebComponent=[querySideBlogMicroservice](https://github.com/ivans-innovation-lab/my-company-blog-materialized-view-microservice)






### Technologies

- [Spring Boot](http://projects.spring.io/spring-boot/) (v1.4.1.RELEASE)
- [Spring Data](http://projects.spring.io/spring-data/)
- [Spring Data REST](http://projects.spring.io/spring-data-rest/)
- Spring Cloud
- [Axon Framework](http://www.axonframework.org/) (v3.0-RC1)
- My SQL / HSQLDB
- RabbitMQ









 


