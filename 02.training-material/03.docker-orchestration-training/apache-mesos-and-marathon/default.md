---
title: 'Apache Mesos and Marathon'
---

#Introduction to Apache Mesos
Apache Mesos is a tool which abstracts system resources of individual systems and makes them available as a pool of resources. Mesos can, then, run and control the life of various applications that are deployed against these pool of resources. 

So, in practice, instead of running an application on a particular machine, one would tell mesos about the resource requirement to run the application and Mesos will figure out the most appropriate machine to run the application or task on.
  
## Mesos Components and Architecture
Mesos world consists of three significant entities.
 
 - **Zookeeper:** Apache zookeeper is an independent project which helps in coordinating among various pieces of software. Most notable uses of zookeeper is service discovery. Zookeeper is distributed and highly available.
 - **Mesos Master:** Mesos Master (or Master cluster) is the used to manage the slaves. It also collects information about resources and tasks to be executed from one entity and passes on to the other entity.
 - **Mesos Framework:** Mesos Framework have two important components, scheduler and executor. Framework Scheduler receives a resource offer from master and it can accept or reject it based on its requirement and algorithm. Framework Executor receives task information from Mesos master and executes the task on the slave. Most popular frameworks are Marathon and Chronos. We will use Marathon for this session.
 - **Mesos Slave:** These are the servers that actually runs the tasks. 

![enter image description here](https://www.dropbox.com/s/mmo10usprdkeq8f/mesos.png?dl=1)

## Introduction to Marathon
A cluster-wide init and control system for services in cgroups or Docker containers. It is a framework for Apache Mesos. Marathon exposes a rest api for managing tasks. We can also use Marathon to run and manage other frameworks. Marathon will receive a resource offer from Mesos and, if it accepts the offer, it will provide Mesos information about the task which will be passed on to the Marathon Executor running on the slave.  

##Installing and Setting up Mesos and other components
Mesosphere, the creators of DataCenter Operating System, have official package repositories for Mesos and related tools and frameworks. So, let us first install the repository and then the Mesos package. 
```bash
$ sudo rpm -Uvh http://repos.mesosphere.io/el/7/noarch/RPMS/mesosphere-el-repo-7-1.noarch.rpm
```
###Installing and Configuring Zookeeper
```bash
$ sudo yum -y install mesosphere-zookeeper
```
Each Zookeeper node needs to have a unique id, which is an integer between 1 to 255. The id is defined in `/var/lib/zookeeper/myid`. We will use only one zookeeper in our setup.
```bash
$ sudo bash -c "echo 1 > /var/lib/zookeeper/myid"
```
We also need to change the zookeeper configuration (`/etc/zookeeper/conf/zoo.cfg`) to reflect the binding address.

```
server.1=<ip_address>:2888:3888
```
Now start the zookeeper server.
```bash
$ systemctl start zookeeper
```
###Installing and Configuring Mesos Master
We can get the Mesos from the mesosphere repository which we setup in the previous step.
```bash
$ sudo yum -y install mesos
```
Now we need to point the Mesos to Zookeeper. 
```bash
$ sudo bash -c "echo zk://<zookeeper_ip>:2181/mesos > /etc/mesos/zk"
```
We also need to tell the Mesos about quorum size. It should always be greater than half of the size of Mesos master cluster. Since we are running with one master, we will go with 1.
```bash
$ sudo bash -c "echo 1 > /etc/mesos-master/quorum"
```
Mesos is uses the hostname to communicate with everyone. So either we have to make it routable via DNS or we should tell mesos to use IP address.
```bash
$ sudo bash -c "echo <ip-address> > /etc/mesos-master/hostname"
```
Now, let us start the Mesos master
```bash
$ sudo systemctl restart mesos-master
```
### Installing and Configuring Marathon
Install Marathon package from the mesosphere repository
```bash
$ sudo yum -y install marathon
```
Like Mesos, Marathon is also hostname sensitive. So we should set the hostname to the IP address.
```bash
$ sudo mkdir -p /etc/marathon/conf/
$ sudo bash -c "echo <ip-address> > /etc/marathon/conf/hostname"
```
Now start the marathon
```bash
$ sudo systemctl restart marathon
```
###Installing and Configuring Mesos Slave
On the slave, we will first install the mesos package and we will stop the master service.
```bash
$ sudo yum -y install mesos
$ sudo systemctl stop mesos-master
```
We need this slave to be able to run Docker. So let us configure that.
```bash
$ sudo bash -c "echo 'docker,mesos' > /etc/mesos-slave/containerizers"
```
Since there can be delays to pull Docker images at times, we should increase that timeout.
```bash
$ sudo bash -c "echo '15mins' > /etc/mesos-slave/executor_registration_timeout"
``` 
Just like the master, we need to tell the slave about the zookeeper and then change the hostname to point to IP address.
```bash
$ sudo bash -c "echo zk://<zookeeper_ip>:2181/mesos > /etc/mesos/zk"
$ sudo bash -c "echo <ip-address> > /etc/mesos-slave/hostname"
```
Now let us restart the Mesos slave.
```bash
$ sudo systemctl restart mesos-slave
```

Great! Both Mesos master and slave are up and we have Marathon running too. We can verify the setup by opening the web interface of both Mesos (port 5050) and Marathon (port 8080). Now let us deploy an application via marathon now.

Application or Docker containers can be deployed via Marathon framework by using web interface or [REST API](https://mesosphere.github.io/marathon/docs/rest-api.html). For this lab, we will see a mix of both where we will create the application using web interface but scale it using the REST API.

So let us go to Marathon web interface and click on `Create` button on upper left corner. A UI will open where we can put the relevant details and deploy the application. We will expand `Docker container settings` to supply details like image, network and port mappings. 

![enter image description here](https://www.dropbox.com/s/n3atbf4oviheqkf/marathon-newapp.png?dl=1)

Let us scale the cluster up to 2 instances by using the REST API.
```bash
$ curl -H 'Content-Type: application/json' -X PUT -d '{"instances":2}' http://<marathon_ip>:8080/v2/apps/<app-id>
```
API is very useful in environments where we want to take automate the scaling depending up on parameters like load, traffic and resource consumption.  