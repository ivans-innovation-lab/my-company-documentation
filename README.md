# Introduction

Today, there are several trends that are forcing application architectures to evolve. Users expect a rich, interactive and dynamic user experience on a wide variety of clients including mobile devices. Applications must be highly scalable, highly available and run on cloud environments. Organizations often want to frequently roll out updates, even multiple times a day. Consequently, it’s no longer adequate to develop simple, monolithic web applications that serve up HTML to desktop browsers.

## **Microservices**

The microservice architectural style \(in opsite of monolithic\) is an approach to developing a single application as a suite of small services, each running in its own process and communicating with lightweight mechanisms, often an HTTP resource API \(RESTful\). These services are built around business capabilities and independently deployable by fully automated deployment machinery. There is a bare minimum of centralized management of these services, which may be written in different programming languages and use different data storage technologies.

A Microservices architecture style brings a lot of operations overhead. Where a monolithic application might have been deployed to a small application server cluster, you now have tens of separate services to build, test, deploy and run, potentially in polyglot languages and environments. All of these services potentially need clustering for failover and resilience, turning your single monolithic system into, say, 20 services consisting of 40-60 processes after we have added resilience.

## **Cloud**

If you look at the concerns typically expressed about microservices, you will find that they are exactly the challenges that a PaaS \(Platform As A Service\) is intended to address. So while microservices do not necessarily imply cloud \(and vice versa\), there is in fact a symbiotic relationship between the two, with each approach somehow compensating for the limitations of the other, much like the practices of eXtreme Programming do the same.

PaaS offerings like Cloud Foundry have raised the level of abstraction to a focus on an ecosystem of applications and services. Cloud Foundry is open source and it can be deployed on private or public \(IaaS\) infrastructure.

Linux container technology, such as Docker, can be used to streamline the development, testing and deployment experience. The Docker platform empowers you to build a CaaS \(Containers as a service\) that fits your business requirements. Whether that means locking down access, enforcing the use of trusted content or the ability to move workloads from cloud to cloud. The Docker CaaS approach focuses on the applications, not having a pre-baked package or a vertically locked stack.

## **Cloud-native**

A cloud-native application is an application that has been designed and implemented to run on a PaaS or CaaS installation, and to embrace horizontal elastic scaling.

### **Twelve-factor app**

The twelve-factor app is a methodology for building cloud-native applications that:

* Use declarative formats for setup automation, to minimize time and cost for new developers joining the project;

* Have a clean contract with the underlying operating system, offering maximum portability between execution environments;

* Are suitable for deployment on modern cloud platforms, obviating the need for servers and systems administration;

* Minimize divergence between development and production, enabling continuous deployment for maximum agility;

* And can scale up without significant changes to tooling, architecture, or development practices.This file file serves as your book's preface, a great place to describe your book's content and ideas.

## Continuous delivery and deployment

The microservice architecture is merely a means to an end. The ultimate goal is to deliver better software faster. Today, that invariably means continuous delivery – for an installed product – or continuous deployment for an -aaS product.

![](/assets/successtriangle.png)

The microservice architecture enables teams to be agile and autonomous. Together, the team of teams and the microservice architecture  enable continuous delivery/deployment.

