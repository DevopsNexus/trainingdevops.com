---
title: 'Docker Remote API'
---

#Docker Remote Api
Docker provides a [rich set of APIs](https://docs.docker.com/engine/reference/api/docker_remote_api/) which can be used to do manage containers. This opens up a huge opportunity of creating automation to improve user experience and scalability. For this lab, we will bind the Docker daemon to a TCP port.

```
$ sudo docker daemon -H :1234
```

Let us try doing some basic operations using the Remote API. 

- Listing the running containers
```
$ curl http://localhost:1234/containers/json
```
- Pulling a Docker image
```bash
$ curl -d '' http://localhost:1234/images/create?fromImage=alpine
```
- Creating a container
```bash
$ curl -H "Content-Type: application/json" -d '{"Image":"alpine", "Cmd":["sleep", "10000"]}' http://localhost:1234/containers/create
```
- Running a container
```bash
$ curl -d '' http://localhost:1234/containers/<container_id>/start
```