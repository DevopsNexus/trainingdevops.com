---
title: 'Docker Networking'
---

# Docker Networking
Connecting containers is very important for most of the applications out there. A classic web application consists of at least a web server, an application server and a database server. The web server needs to talk to the application server and the application server would need to talk to database server. A legacy, but popular way to do so is via passing `--link` flag to `docker run` command.
```bash
$ sudo docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw mysql
$ sudo docker run --link mysql -i -t centos /bin/bash
```
Currently, Docker has a very elaborated network suite to cater to a wide variety of use-cases. By default it comes with three network pre-configured:

 - **none:** Containers in this network have only local interface. They have no external connectivity. So this is ideal for applications that do not need network but may contain sensitive info 
 - **host:** This network copies the host network. So the containers running in this network basically have identical network as the host.
 - **bridge:** This the default network in which Docker will boot up containers. This creates a bridge (docker0 interface) with a gateway and other required network components. Containers running in this network can talk to the other containers in the same network but they cannot talk to to any other containers in any other network. 

Please take moment to run a container in all these network and run `ifconfig` or `ip addr`.

```bash
$ sudo docker run --net=<type_of_the_network> -i -t centos /bin/bash
```
Docker does not limit us to these three networks only. We can create additional networks as and when required. This helps in maintaining isolation between networks and improves security. Docker lets us create two kinds of user-defined networks:

 - **Bridge:** It is similar to default bridge network with minor differences, like --link flag is not available for user-defined networks. Since bridge networks are defined per host, all the containers in this network must reside on the same host. Multiple networks on the same host are isolated from each other, however, a container can be a part of multiple network and facilitate inter-network communication.
Let us create and run a container in user-defined bridge network.
```bash
$ sudo docker network create --driver bridge isolated_nw
$ sudo docker run --net=isolated_nw -i -t centos /bin/bash
```
Try pinging containers running in other networks.

 - **Overlay:** Overlay network is Docker's way to simplify multi-host networking. An overlay network can span to several hosts and maintain network level isolation. It uses libnetwork and libkv. To run libkv, we need to install one of the key value stores supported by Docker which are Consul, Etcd, and ZooKeeper. For this session, we will go ahead with etcd.
 So let us start by installing etcd on all the servers that are going to run the Docker daemon.
 
 ```bash
 $ sudo yum -y install etcd
 ```
 Designate one of the servers as etcd master and configure it to listen to the non-local IP address.

 ```
 ETCD_LISTEN_CLIENT_URLS="http://<ip-addr>:2379"
 ETCD_ADVERTISE_CLIENT_URLS="http://<ip-addr>:2379"
 ```
 Now restart the etcd service.

 ```bash
 $ sudo systemctl restart etcd
```
Once our etcd service is up, we can start the Docker daemon with some additional parameters. 

 ```bash
$ nohup sudo docker daemon  --cluster-store=etcd://<ip-addr>:2379 --cluster-advertise=eth0:2376 &
 ```
 Now let us create an overlay network, named `overlay-net1` on one of the machines.
 ```bash
 $ sudo docker network create --driver overlay overlay-net1
 ```
 And then let us check this on the other machine.
 ```bash
 $ sudo docker network ls
 ```
 This is it! Now any container in the overlay network will be able to communicate to each other regardless of what host it is running on.