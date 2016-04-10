---
title: 'Installing Docker'
---

# How to install Docker?
Installing Docker is easy on most of the popular Linux based operating systems. Docker itself provides deb and rpm repositories to do so. A full list of supported Linux flavors and respective repository settings can be found at [Docker documentation](https://docs.docker.com/engine/installation/). 
However, if you are a Mac OS X or Windows user, things will become a little tricky for you since Docker needs some of the Linux Kernel's functionalities to run. A simple way to run Docker on your Mac or Windows it to enable virtualization and then using [Docker Toolbox](https://www.docker.com/toolbox).

For the training session, we would use CentOS Linux to run Docker engine. CentOS is one of the market leaders for providing enterprise grade Linux distribution and is an ideal choice to run on production.

**Step 1:** Add the repository
```bash
$ sudo vim /etc/yum.repos.d/dockerrepo.repo
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/$releasever/
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
```
**Step 2:** Install Docker engine
```
$ sudo yum install docker-engine
```
**Step 3:** Start the Docker service
```
$ sudo systemctl start docker
```
In three simple steps, we installed Docker on our machine. Let us verify that our Docker setup is actually running
```
$ sudo docker ps
```
Since we have just installed Docker, the above command will show that there are no containers running at the moment. If we do not see any error, then we can assume that our setup was successful.