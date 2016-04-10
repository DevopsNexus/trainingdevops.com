---
title: 'Deploying Docker Registry with Let''s Encrypt SSL/TLS Certs'
summary:
    enabled: '1'
    format: short
content:
    items: '@self.children'
    limit: 5
    pagination: true
    url_taxonomy_filters: true
    order:
        dir: desc
        by: date
---

Docker Hub is a great option to store and distribute Docker images. But Docker Hub is SaaS. It is hosted by Docker Inc. In some cases, there might be a requirement to run Docker registry on-premise due to security or compliance reasons, or just to save money and bandwidth for operating private Docker images.
So how to host a Docker registry securely with appropriate certificates? In this tutorial, we'll see exactly that and try to build a basic registry with filesystem backend.

##Installing and Setting up Docker Registry
At the time of writing, registry:2 is the latest registry. So let us pull the image first
```bash
$ sudo docker pull registry:2
```

As and when we push images to the registry, registry will store the data. We want to ensure that our data is safe, even if the container running registry dies. The easiest way to achieve that is to mount a directory from the host to the container which will store the data. So let us create a directory.

```bash
$ sudo mkdir /opt/registry-data
```
## Running the Docker Registry - without TLS /SSL certs
We will try to run the registry without certificates first. This is to ensure that the registry and the volume mount actually works for us.
```bash
$ sudo docker run -d -p 5000:5000 -v /opt/data:/var/lib/registry registry:2
05c2552e6bcbd930f6ae885649b6ebd66d081a2e07f8c234e7e615a44241e421
```
Let us verify that the container is actually running
```
$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
05c2552e6bcb        registry:2          "/bin/registry /etc/d"   30 seconds ago      Up 29 seconds       0.0.0.0:5000->5000/tcp   thirsty_bohr
```
Great! We have our registry up and running. But keep in mind that this is not a very secure setup since it is running without TLS. 
Let us fix that.
## Getting TLS certificate from Let's Encrypt
_If you already have a SSL/TLS certificate from a trusted CA then please skip this section._

[Let's Encrypt](https://letsencrypt.org/) is a free, open and trusted CA. They provide ssl certs without any monetary cost. So let us get a cert for our setup. For this, we need to decide a domain and create an appropriate entry in our DNS. Let's Encrypt client will verify this.
Assuming that the public IP of our server is 1.1.1.1, we can create an A Record on our DNS server with name as registry.example.com and value as 1.1.1.1.
Note that this is needed just to create the cert. Once the creation is done, you can update the DNS record to any IP and use the cert on any machine.
Now, we need to clone the letsencrypt repository from Github to run the letsencrypt client. This has to be run from the server which points to the name registry.example.com.
```
$ git clone https://github.com/letsencrypt/letsencrypt.git
$ cd letsencrypt
$ sudo ./letsencrypt-auto
```
The above command will present an interface and will ask for your email and the domain for which the cert is to be generated, which is registry.example.com. In a few steps, we would be able to get the certificates for registry.example.com
## Running Docker Registry with SSL/TLS certificates
Running a registry with appropriate certificates is actually quite easy. We just have to ensure that the certs are available in the registry container. Mounting a directory with certs is a good idea to do so. Assuming that our cert and the private key is kept in /opt/certs, we should mount that
```bash
$ sudo docker run -d -p 5000:5000 --restart=always --name registry -v /opt/data:/var/lib/registry -v /opt/certs:/certs -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt 
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key registry:2
```
That is it! We got our registry up and running. 