---
title: 'Docker Compose'
---

# Docker Compose
Docker is a great way to containerize and run an application. However, these days, applications are complicated and may involve several components and dependent applications. Communicating these dependencies and providing instructions to run them can be tricky at times. Docker Compose provides a way to define and run multi-container applications easily. It involves putting all the required information in a predefined format in a file called `docker-compose.yml` and then calling the `docker-compose up` which reads the compose file and create the containers.

## Installing Docker Compose
Docker Compose is distributed as a binary and can be downloaded from Docker's Github account. Instruction to download and run the docker-compose binary can be obtained from the [releases page](https://github.com/docker/compose/releases/latest). Usually just downloading the binary and setting executable permission works. Let us write a simple compose file.
```
owncloud9: 
  image: adimania/owncloud9-centos7
  ports: 
    - 80:80
  links:
    - mysql
mysql: 
  image: mysql
  environment:
    - MYSQL_ROOT_PASSWORD=my-secret-pw
    - MYSQL_DATABASE=owncloud
```
This compose file uses two images, anchorcms and mysql. The anchorcms container is linked with mysql container. So the mysql container would be reachable from the anchorcms container via hostname `mysql`.  Let us execute this compose file
```bash
$ sudo docker-compose -f docker-compose.yml up
```
This will start both the containers with right parameters. An exhaustive documentation on Docker Compose is available on [Docker's website](https://docs.docker.com/compose/compose-file/).