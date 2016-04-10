---
title: Kubernetes
---

# Introduction to Kubernetes
Kubernetes is an open source solution for managing container based deployment, scaling and operations. Kubernetes was open-sourced by Google and is a very active project with contributions from Google, Red Hat, IBM, Microsoft and many others. Kubernetes is build specially for containers and has a very feature rich and scalable architecture. 

## How to Setup Kubernetes?
Kubernetes has extensive [documentation](http://kubernetes.io/v1.1/docs/getting-started-guides/README.html) on how to install it on various operating systems. We will cover installation on CentOS 7 here, but Kubernetes can work with other Linux flavors as well.

For this lab, we will setup one master and one node. So let us boot up two CentOS 7 (virtual) machines. If there are enough resources available, we can setup as many nodes as we want. Let us name the Kubernetes master as *master* and the kubernetes node as *node1*. To ensure that these machines are reachable by these names, we should change the /etc/hosts of both the master and the node virtual machines to add the lines below. Please note that IP addresses in your environment might be different:
```
192.168.122.1 master
192.168.122.11 node1
```
Kubernetes is available in CentOS's default rpm repository which eases the installation process.

##Setting up Kubernetes Master
Let us install the required packages from the CentOS repository.
```bash
$ sudo yum -y install kubernetes etcd
```
Once we got the rpm packages installed, we need to make some config changes. Our Kubernetes master will consists of etcd, apiserver, schedular and controller-manager. Technically, etcd is not a Kubernetes component but it is required for Kubernetes to work. First of all, we need to bind etcd to the interface with 192.168.122.1 assigned. For the sake of convenience, we can make etcd listen to 0.0.0.0. This can be done by editing the etcd configuration file located at `/etc/etcd/etcd.conf`. After this start the etcd service:
```bash
$ sudo systemctl start etcd
```
Now let us configure the apiserver. Edit `/etc/kubernetes/apiserver` to look like this:

```
# The address on the local server to listen to.
KUBE_API_ADDRESS="--address=0.0.0.0"

# Comma separated list of nodes in the etcd cluster
KUBE_ETCD_SERVERS="--etcd_servers=http://master:2379"

# Address range to use for services
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.10.1.0/24"

# default admission control policies
KUBE_ADMISSION_CONTROL="--admission_control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota"

# Add your own!
KUBE_API_ARGS="--service_account_key_file=/etc/kubernetes/kubernetes.key"
```
While most of the configuration is default, I have changed the cluster IP range to 10.10.1.0/24. These IP addresses will be used to create service endpoints. So these should be routable. I will talk a bit more on routing in a while. I have also generated an RSA key pair and supplied it as an additional argument to the configuration. This is to make ServiceAccount work, however we won't be using it in our lab.

After we are done with apiserver, we need to configure some general parameters for Kubernetes. These are located in `/etc/kubernetes/config`. We will use the default config here but we will point KUBE_MASTER to the right host. It should look like the config given below:

```
# logging to stderr means we get it in the systemd journal
KUBE_LOGTOSTDERR="--logtostderr=true"

# journal message level, 0 is debug
KUBE_LOG_LEVEL="--v=0"

# Should this cluster be allowed to run privileged docker containers
KUBE_ALLOW_PRIV="--allow_privileged=false"

# How the controller-manager, scheduler, and proxy find the apiserver
KUBE_MASTER="--master=http://master:8080"
```
For controller-manager, we just need to tell the location of our key file which we created previously.
```
KUBE_CONTROLLER_MANAGER_ARGS="--service_account_private_key_file=/etc/kubernetes/kubernetes.key"
```
Since this is Kubernetes master, we don't need to do anything with kublet or proxy configuration. Schedular would work with default configuration so we don't need to change that.
Now let us start all the services.
```
$ for SERVICES in etcd kube-apiserver kube-controller-manager kube-scheduler; do
  sudo systemctl restart $SERVICES;
  sudo systemctl enable $SERVICES;     
  sudo systemctl status $SERVICES ; 
done
```
That is all. Our Kubernetes master is up!

## Setting up Kubernetes Node(s)
Let us install the node package from the CentOS repository.
```bash
$ sudo yum -y install kubernetes-node
```
**Note: The command above will pull Docker as a dependency. Do not start Docker at this moment. During its first run, Docker creates a bridge network which we want to alter. So we would start Docker only after we have made the required changes.**

Kubernetes node will run kubelet and kube-proxy. Let us start configuring them one by one.
First, we set up the common config for Kubernetes which is defined by `/etc/kubernetes/config`:
```
# logging to stderr means we get it in the systemd journal
KUBE_LOGTOSTDERR="--logtostderr=true"

# journal message level, 0 is debug
KUBE_LOG_LEVEL="--v=0"

# Should this cluster be allowed to run privileged docker containers
KUBE_ALLOW_PRIV="--allow_privileged=false"

# How the controller-manager, scheduler, and proxy find the apiserver
KUBE_MASTER="--master=http://master:8080"
```
I have used the default config but added the address of KUBE_MASTER. 
Let us setup kubelet now. Edit `/etc/kubernetes/kubelet` to look like this:
```
# The address for the info server to serve on (set to 0.0.0.0 or "" for all interfaces)
KUBELET_ADDRESS="--address=0.0.0.0"

# The port for the info server to serve on
# KUBELET_PORT="--port=10250"

# You may leave this blank to use the actual hostname
KUBELET_HOSTNAME="--hostname_override=node1"

# location of the api-server
KUBELET_API_SERVER="--api_servers=http://master:8080"

# Add your own!
KUBELET_ARGS=""
```
Other than telling it the master's address, I have also overridden the hostname for kubelet. This will make it easy for us to identify the node if we have many of them running in our environment. We do not need to do anything for kube-proxy because it will read the master's address from the common config that we defined.

Now let us change the Docker config. We are doing this to put the Docker bridge subnet in a routable network. If you have other means of doing so or if you are testing things out in a IaaS provider's environment like Google Cloud, then please skip to the next section. To make the bridge routable, we will give it an IP CIDR which we can route and we will take away the routing responsibilities from iptables. 
```bash
# Modify these options if you want to change the way the docker daemon runs
OPTIONS='--selinux-enabled --bip=10.10.10.1/24 --iptables=false --ip-masq=false'
```
We changed the bridge IP to 10.10.10.1/24 and we have set iptables and ip-masq to false.
Now we will start all the required services.
```
$ for SERVICES in kube-proxy kubelet docker; do
    sudo systemctl restart $SERVICES;
    sudo systemctl enable $SERVICES;
    sudo systemctl status $SERVICES; 
  done
```
With this, our node is up!
## Running 
Now that our Kubernetes setup up and running, let us verify our setup and build some entities of Kubernetes world. 
```
$ sudo kubectl get nodes
NAME        LABELS                             STATUS     AGE
node1       kubernetes.io/hostname=node1       Ready   1m
```
As long as we see our node here with a "Ready" status, we have done our setup in the right way. Let us define a pod, a replication controller and a service.
#### A basic Pod
Let us build a pod for our example application. 
```bash
apiVersion: v1
kind: Pod
metadata:
  name: nginx-flask-demo
  lables:
    demo: nginx-flask
spec:
  containers:
    - name: kubernetes-demo-nginx
      image: adimania/nginx_pp_5000
      command:
        - /opt/run.sh
      ports:
        - containerPort: 80
    - name: kubernetes-demo-flask
      image: adimania/demo-app
      ports:
        - containerPort: 5000
```
#### A simple Replication Controller
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-flask-controller
spec:
  replicas: 2
  selector:
    app: nginx-flask-pod
  template:
    metadata:
      labels:
        app: nginx-flask-pod
    spec:
      containers:
      - name: kubernetes-demo-nginx
        image: adimania/nginx_pp_5000
        command:
          - /opt/run.sh
        ports:
          - containerPort: 80
            targetPort: 80
      - name: kubernetes-demo-flask
        image: adimania/demo-app
        ports:
          - containerPort: 5000
```
#### Service for the Replication Controller
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-flask-service
spec:
  type: NodePort
  ports:
    -
      port: 80
      targetPort: 80
  selector:
    app: nginx-flask-pod
```
Now we have three components defined. Let us run them one by one
```
$ sudo kubectl create -f pod.yml
pod "nginx-flask-demo" created
$ sudo kubectl get pods
NAME               READY     STATUS    RESTARTS   AGE
nginx-flask-demo   2/2       Running   0          28s
``` 
```
$ sudo kubectl create -f rc.yml
replicationcontroller "nginx-flask-controller" created
$ sudo kubectl get rc
CONTROLLER               CONTAINER(S)            IMAGE(S)                 SELECTOR              REPLICAS   AGE
nginx-flask-controller   kubernetes-demo-nginx   adimania/nginx_pp_5000   app=nginx-flask-pod   2          31s
                         kubernetes-demo-flask   adimania/demo-app
``` 
```
$ sudo kubectl create -f svc.yml
service "nginx-flask-service" created
$ sudo kubectl get svc
NAME               LABELS                                    SELECTOR              IP(S)         PORT(S)   AGE
kubernetes         component=apiserver,provider=kubernetes   <none>                10.254.0.1    443/TCP   211d
nginx-flask-svce   <none>                                    app=nginx-flask-pod   10.10.1.121   80/TCP    34s
``` 
### Debugging Kubernetes Master or Nodes
Docker and Kubernetes, both are growing at a mind-boggling pace. There is a possibility that if you are reading this document at a later date, some things might not work as expected. Since we are using systemd and related services, journalctl command is the best tool we got to debug.
```
$ sudo journalctl -u kube-apiserver
``` 
Substitute the troubling component in place of "kube-apiserver" and check the logs. 