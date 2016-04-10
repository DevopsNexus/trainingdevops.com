---
title: 'Basic Docker Commands'
---

# Basic Docker commands
Here is a list of ten basic and easiest Docker commands. This will help you in getting started with Docker

- List all running containers
```
$ sudo docker ps
```
-  List all running and stopped containers
```
$ sudo docker ps -a
```
- List all Docker images
```
$ sudo docker images
```
- Pull a Docker image
```
$ sudo docker pull alpine
```
- Run a Docker container
```
$ sudo docker run -i -t alpine /bin/sh
```
- Check logs of a Docker container
```
$ sudo docker logs <container_id>
```
- Run a command in a running Docker container
```
$ sudo docker exec -i -t <container_id> /bin/sh
```
- Stop a Docker container
```
$ sudo docker stop <container_id>
```
- Delete a Docker container
```
$ sudo docker rm <container_id>
```
- Delete a Docker image
```
$ sudo docker rmi <image_id>
```

**Bonus command**
```
$ sudo docker --help
```