---
title: 'Introduction to Docker'
---

# Introduction to Docker
Docker is a containerization engine. It helps in containerizing your applications, along with the environments.
##What are Containers?
 - A way to package your application and environments.
 - A way to ship your application with ease.
 - A way to isolate resources in a shared system.

Containers have been around much before Docker. OpenVZ and LXC projects came along about half a decade before Docker. So all Docker containers and containers but not all containers are Docker containers. 

## What is Docker?
**Official definition of Docker**
> Docker allows you to package an application with all of its
> dependencies into a standardized unit for software development.

**How does a Docker containers looks like?**

 - Docker runs on host operating system.
 - The kernel is shared among all the containers but the resources are in appropriate namespaces and cgroups.
 - Typical container consists of operating system libraries and the application code, much of which is shared.
 - No emulation of hardware.

![A Docker container](https://www.dropbox.com/s/o8850vv9o1f2lna/container.png?dl=1)

**How is this different that a Virtual machine?**

 - Additional hypervisor layer.
 - Each guest OS comes with its own Kernel and libraries.
 - Hardware is emulated.
 - Much more resource intensive.
 - Difficult to share a virtual machine.

![enter image description here](https://www.dropbox.com/s/sao37n5evgyg4gw/vm.png?dl=1)

##Basic Docker Terminologies
 - Docker image
 Docker images are *read-only* frozen templates which may consists of system libraries, application code and dependencies. 
 - Docker container
A *stateful and running instance* of a Docker image is called a Docker container. 
Another way to look at it is that a running Docker container is a Docker image with a writable layer on top of it.
![enter image description here](https://www.dropbox.com/s/75kllfrh4gcncb1/docker-filesystem.png?dl=1)
 - Docker registry
A Docker registry is a collection or repository of Docker images. They can be hosted publicly or privately in on-premise setup.
 - Docker Hub
The default Docker registry operated by Docker Inc. is called Docker Hub.

##How this all works together?
A typical Docker workflow looks like this.
![enter image description here](https://www.dropbox.com/s/8745b49s9pkukb5/docker-arch.png?dl=1)
 
Docker client gives a call to Docker host to run a container from a particular image. If Docker host has that image, it run a container from it. If it does not have the image, it looks for it in the registry and pulls it on the host and then runs a container from it.
Note that any of these three components can be on the same machine or on different machines.
