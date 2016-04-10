---
title: 'Docker Swarm'
---

#Docker Swarm
Docker Swarm is native clustering system provided by Docker itself. It groups all the Docker hosts running Swarm in a single entity which can be used to run containers. Swarm has a simple master-slave like architecture. Swarm's primary master, called manager, schedules containers runs across the slaves, called nodes. It uses a consul for service discovery. So let us build a simple Swarm cluster.

##Setting up Swarm Manager 
First of all, we need to install the Docker engine. Since we have already done that, we will skip this step. We will, however, stop our running Docker engine since we need to start it with specific network binding.
```bash
$ nohup sudo docker daemon -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock &
```
Here, we have bound the docker daemon to port 2375 as well as to the docker.sock file. Swarm will use port 2375 to communicate among other members. 
We also need to run consul for service discovery.
```bash
$ sudo docker run -d -p 8500:8500 --name=consul progrium/consul -server -bootstrap
```
At this point fire up `ifconfig` or `ip addr` and note down the IP of your machine. For me, it is 172.31.10.244. Now we will run swarm manager on this machine. Note that swarm manager and nodes are distributed as Docker containers themselves.
```bash
$ sudo docker run -d -p 4000:4000 swarm manage -H :4000 --replication --advertise 172.31.10.244:4000  consul://172.31.10.244:8500
```
In a production grade cluster, there should be several swarm managers running. For that, just run the swarm container using the above command but change the IP in the `--advertise` flag with the IP of the additional managers. Remember that the IP of the consul should be same across the swarm cluster. 
docker -H :4000 info

##Setting up Swarm Nodes
Running node is as simple as running the container like we did above with slightly different arguments. But before that, we should run the Docker engine with different network binds as we did for the manager.
```bash
$ nohup sudo docker daemon -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock &
```
Now, get the IP of the node by using `ifconfig` or `ip addr`. For me, it came out to be 172.31.10.245. We will run the swarm container with keeping the consul discovery backend as same.
```bash
$ sudo docker run -d swarm join --advertise=172.31.10.245:2375 consul://172.31.10.244:8500
```
More nodes can be added to the Swarm cluster by running the swarm container as above, changing the `--advertise` flag with the IP of the additional nodes. Remember that the IP of the consul should be same across the swarm cluster. 

##Verify the setup
To verify that our setup is running good, we can checkout if our Swarm manager is able to detect the Swarm node. Fire the command below on Swarm manager:
```bash
$ sudo docker -H :4000 info
```
The command should list the node and the resources available to run the containers.

##Running the containers
Let us run a container on this cluster and see where it runs. On the manager, we will run the following command.
```bash
$ sudo docker -H :4000 run -d -p 80:80 adimania/owncloud9-centos7 /opt/run.sh
```
This should schedule and run the container on the node.