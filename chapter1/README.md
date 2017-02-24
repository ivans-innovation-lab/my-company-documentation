# Part I - The A**rchitecture**

The microservice architectural style is an approach to developing a single application as a suite of small services, each running in its own process and communicating with lightweight mechanisms, often an HTTP resource API \(RESTful\). These services are built around business capabilities and independently deployable by fully automated deployment machinery. There is a bare minimum of centralized management of these services, which may be written in different programming languages and use different data storage technologies.

When organizations make the choice to put a digital platform in place, a discussion on Microservices is never far behind. By putting a Microservices layer in place, an organization creates the springboard to launch into the digital future, whether that involves apps, rich Web clients, or IoT devices such as in-store beacons. Individual Microservices, or orchestrated groups of Microservices, serve as the foundation for this innovation. The data being passed to and from Microservices also serves as the basis for behavioral analytics and Big Data, allowing organizations to tailor their digital services based on their users.

A Microservices architecture style brings a lot of operations overhead. Where a monolithic application might have been deployed to a small application server cluster, you now have tens of separate services to build, test, deploy and run, potentially in polyglot languages and environments.

#### Cloud {#cloud}

If you look at the concerns typically expressed about microservices, you will find that they are exactly the challenges that a PaaS \(Platform As A Service\) is intended to address.

PaaS offerings like [Cloud Foundry](https://www.cloudfoundry.org/) have raised the level of abstraction to a focus on an ecosystem of applications and services. Cloud Foundry is open source and it can be deployed on private or public \(IaaS\) infrastructure.

Linux container technology, such as [Docker](https://www.docker.com/), can be used to streamline the development, testing and deployment experience. The Docker platform empowers you to build a [CaaS](https://blog.docker.com/2016/02/containers-as-a-service-caas/) \(Containers as a service\) that fits your business requirements.

#### Cloud native {#cloud-native}

A cloud-native application is an application that has been designed and implemented to run on a PaaS or CaaS installation, and to embrace horizontal elastic scaling.

