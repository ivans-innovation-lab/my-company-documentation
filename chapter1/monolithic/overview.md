# Monolithic Architecture Overview

## Patterns and techniques

Patterns/techniques that are used:

- Domain Driven Design
- Command and Query Responsibility Separation (CQRS)
- Event Sourcing

<iframe src="https://structurizr.com/embed/22311?diagram=Context&diagramSelector=false" width="600" height="425" marginwidth="0" marginheight="0" frameborder="0" scrolling="no" allowfullscreen="true"></iframe>

## System Context

We are developing a server-side enterprise application. It will support a variety of different clients including desktop browsers, mobile browsers and native mobile applications. This clients will enable employees to manage blog posts, projects information, customers and other data, and it will enable customers to browse the news and submit requests for new interesting projects. The application will also expose an API for 3rd parties (partners) to consume and support B2B. It will also integrate with other systems (Github, LinkedIn, Twitter) via web services to enrich and to share relevant data with them.

## Containers

- Web Application (Tomcat)
- Database (HSQLDB)

## Components

The domain is literally split into a command-side component and a query-side component (this is [CQRS](http://microservices.io/patterns/data/cqrs.html) in its most literal form).
Communication between the two components is event-driven and the demo uses simple event store (Database in this case - JPA - HSQLDB) as a means of passing the events between components.

The command-side processes commands. Commands are actions which change state in some way. The execution of these commands results in Events being generated which are persisted by Axon (using SQL DB - HSQLDB) and propagated out to components. In [event-sourcing](http://microservices.io/patterns/data/event-sourcing.html), events are the sole records in the system. They are used by the system to describe and re-build aggregates on demand, one event at a time.

The query-side is an event-listener and processor. It listens for the Events and processes them in whatever way makes the most sense. In this application, the query-side just builds and maintains a materialized view which tracks the state of the individual aggregates (Product, Blog, ...).

Every component is a separate [maven](https://maven.apache.org/what-is-maven.html) project/library:

- Project - Command side: https://github.com/ivans-innovation-lab/my-company-project-domain
- Blog Posts - Command side: https://github.com/ivans-innovation-lab/my-company-blog-domain

- Project - Query side: https://github.com/ivans-innovation-lab/my-company-project-materialized-view

- Blog Posts - Query side: https://github.com/ivans-innovation-lab/my-company-blog-materialized-view


The command-side and the query-side do not have REST API's.
This is why we need another component - web component, which will expose capabilities of all other components via REST API, and package application in one _'.war'_ archive by including other components as depended libraries. 
- Web application: https://github.com/ivans-innovation-lab/my-company-monolith 

### Technologies

- [Spring Boot](http://projects.spring.io/spring-boot/) (v1.4.1.RELEASE)
- [Spring Data](http://projects.spring.io/spring-data/)
- [Spring Data REST](http://projects.spring.io/spring-data-rest/)
- [Axon Framework](http://www.axonframework.org/) (v3.0-RC1)
- My SQL / HSQLDB


