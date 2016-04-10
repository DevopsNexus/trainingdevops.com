---
title: 'Docker Registry'
---

# Docker Registry
Docker registry is a repository of Docker images. It enables users to share images publicly and privately. Docker registry can be hosted on-premise, and can be password protected which makes it a great tool for any corporate environment where security and privacy are important. 

##How to setup Docker Registry?
Docker has containerize the registry which makes the installation extremely easy. At the moment, registry:2 is the latest registry. So let us pull the image first
```bash
$ sudo docker pull registry:2
```

As and when we push images to the registry, registry will store the data. We want to ensure that our data is safe, even if the container running registry dies. The easiest way to achieve that is to mount a directory from the host to the container which will store the data. So let us create a directory.

```bash
$ sudo mkdir /opt/registry-data
```
Now will mount this directory inside the registry container.
```bash
$ sudo docker run -d -p 5000:5000 -v /opt/registry-data:/var/lib/registry registry:2
```
Let us verify that the container is actually running
```bash
$ sudo docker ps
```
Once we confirm that the container is running, we should tag an image and try to push it.
```bash
$ sudo docker tag <image_id> localhost:5000/myimage
$ sudo docker push localhost:5000/myimage
```
