# Visualize Your Architecture

The [C4 software architecture model](https://structurizr.com/help/c4) is a simple hierarchical way to think about the static structures of a **software system** in terms of **containers**, **components** and **classes** \(or **code**\).

> A **software system** is made up of one or more **containers** \(web applications, mobile apps, standalone applications, databases, file systems, etc\), each of which contains one or more **components**, which in turn are implemented by one or more **classes**.

Visualising this hierarchy is then done by creating a collection of **System Context**, **Container**, **Component** and \(optionally\) **Class** diagrams. [Structurizr ](https://github.com/ivans-innovation-lab/my-company/tree/28b2881e0ab092a89b8449019c2f06e0f15946b9/www.structurizr.com)is an implementation of the System Context, Container and Component diagrams.

## System Context Diagram

A System Context diagram can be a useful starting point for diagramming and documenting a software system, allowing you to step back and look at the big picture. Draw a simple block diagram showing your system as a box in the centre, surrounded by its users and the other systems that it interfaces with.

Detail isn't important here as this is your zoomed out view showing a big picture of the system landscape. The focus should be on people \(actors, roles, personas, etc\) and software systems rather than technologies, protocols and other low-level details. It's the sort of diagram that you could show to non-technical people.

## Container Diagram

Once you understand how your system fits in to the overall IT environment with a System Context diagram, a really useful next step can be to illustrate the high-level technology choices with a Container diagram. A "container" is something like a web server, application server, desktop application, mobile app, database, file system, etc. Essentially, a container is anything that can execute code or host data.

The Container diagram shows the high-level shape of the software architecture and how responsibilities are distributed across it. It also shows the major technology choices and how the containers communicate with one another. It's a simple, high-level technology focussed diagram that is useful for software developers and support/operations staff alike.

## Component Diagram

Following on from a Container diagram showing the high-level technology decisions, you can then start to zoom in and decompose each container further. However you decompose your system is up to you, but this is about identifying the major logical structural building blocks and their interactions.

The Component diagram shows how a container is divided into [components](https://structurizr.com/help/components-vs-classes), what each of those components are, their responsibilities and the technology/implementation details.

