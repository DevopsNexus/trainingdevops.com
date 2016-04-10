---
title: 'Building Docker Images'
---

#Docker Images

Docker images are the read-only templates from which a container is created. In previous sections, we have seen how to pull and list the docker images. Let us learn how to build them.

## Building Docker images by using docker commit
A lot of Docker's command line sub-commands are inspired by Git, including this one. Docker provides a very simple way to create images. The workflow goes like this:

 - Create a container from a Docker image
 - Make required changes, like installing a webserver or adding a user
 - On command line execute the following
```bash
$ sudo docker commit <container_id> <optional_tag>
```
While this presents a very simple way to create images, this is not a good way to do so. Creating images this way is not very reproducible and hence not recommended.

## Building Docker images by using a Dockerfile
Docker provides an easy way to build images from a description file. This file is known as Dockerfile. A simple Dockerfile will look like this:
```
FROM centos
MAINTAINER Aditya Patawari <aditya@adityapatawari.com>

RUN yum -y update && yum clean all
RUN yum -y install httpd
EXPOSE 80
CMD ["/usr/sbin/httpd", "-D", "FOREGROUND"]
```
Dockerfile supports various commands. I have used a few of them and I am going to describe them below:

- FROM: This defines the base image for the image that would be created
- MAINTAINER: This defines the name and contact of the person that has created the image.
- RUN: This will run the command inside the container.
- EXPOSE: This informs that the specified port is to be bind inside the container. 
- CMD: Command to be executed when the container starts.

A comprehensive list and description of all the supported commands can be found in the [documentation](https://docs.docker.com/engine/reference/builder/).
Let us create a file with the above lines and build an image from this.
```
$ sudo docker build -f <path_of_dockerfile> . 
```
Images created this way document the steps involved clearly and hence using Dockerfile is a good way to build reproducible images.
It is also worth nothing that each line in Dockerfile create a new layer in Docker image. So often, we will club statements using and operator (&&) like this:
```bash
RUN yum -y update && yum clean all
```