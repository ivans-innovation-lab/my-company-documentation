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


<iframe src="https://structurizr.com/embed/22311?diagram=Context&diagramSelector=false" width="600" height="425" marginwidth="0" marginheight="0" frameborder="0" scrolling="no" allowfullscreen="true"></iframe>

## System Context

We are developing a server-side enterprise application. It will support a variety of different clients including desktop browsers, mobile browsers and native mobile applications. This clients will enable employees to manage blog posts, projects information, customers and other data, and it will enable customers to browse the news and submit requests for new interesting projects. The application will also expose an API for 3rd parties (partners) to consume and support B2B. It will also integrate with other systems (Github, LinkedId, Twitter) via web services to enrich and to share relevant data with them.

## Containers

The domain is literally split into a *command-side* microservice application/container and a *query-side* microservice application/container (this is CQRS in its most literal form).

Communication between the two microservices is `event-driven` and the demo/lab uses RabbitMQ messaging as a means of passing the events between processes (VM's).

The **command-side** processes commands. Commands are actions which change state in some way. The execution of these commands results in `Events` being generated which are persisted by Axon  and propagated out to other VM's (as many VM's as you like) via RabbitMQ messaging. In event-sourcing, events are the sole records in the system. They are used by the system to describe and re-build aggregates on demand, one event at a time.

The **query-side** is an event-listener and processor. It listens for the `Events` and processes them in whatever way makes the most sense. In this application, the query-side just builds and maintains a *materialised view* which tracks the state of the individual agregates (Product, Blog, Customer, ...). The query-side can be replicated many times for scalability and the messages held by the RabbitMQ queues are durable, so they can be temporarily stored on behalf of the event-listener if it goes down.

The command-side and the query-side containers both have REST API's which can be used to access their capabilities.

<iframe src="https://structurizr.com/embed/22311?diagram=Containers&diagramSelector=false" width="600" height="425" marginwidth="0" marginheight="0" frameborder="0" scrolling="no" allowfullscreen="true"></iframe>

