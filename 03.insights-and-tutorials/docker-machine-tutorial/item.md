---
title: 'Docker Machine Tutorial'
summary:
    enabled: '1'
    format: short
---

Docker Project has come up with a tool to manage multiple Docker Engines, both remote and local. This tool is called Docker Machine. It comes with pre-built drivers to interact with virtualization providers like VMware and VirtualBox as well as cloud providers like Amazon Web Services, Microsoft Azure, Google Cloud and Digital Ocean.

##Setting up a cloud hosting provider
A full list of supported Docker Machine drivers can be found on their [documentation](https://docs.docker.com/machine/drivers/). Let us try out the Digital Ocean driver. Each driver has a way to authenticate. For Digital Ocean, we need an [API access token](https://cloud.digitalocean.com/settings/api/tokens). To generate one, we just need to click on "Generate New Token" button. Once that is done, we are good to try out Docker Machine.

##Installing Docker Machine
Docker Machine is distributed as a binary. So we just need to download it from [Docker Machine release page](https://github.com/docker/machine/releases/) and set executable permission.
```bash
$ wget -O docker-machine https://github.com/docker/machine/releases/download/v0.6.0/docker-machine-Linux-x86_64
$ chmod +x docker-machine
```

##Creating a Docker Machine managed VM
Let us create a new machine and set it up with Docker daemon. Doing this manually would involve quite a few steps. Using Docker Machine, it is as simple as firing a command locally
```bash
$ ./docker-machine create --driver digitalocean --digitalocean-access-token=<token> test-vm
Running pre-create checks...
Creating machine...
(test-vm) Creating SSH key...
(test-vm) Creating Digital Ocean droplet...
(test-vm) Waiting for IP address to be assigned to the Droplet...
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with ubuntu(systemd)...
Installing Docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: ./docker-machine env test-vm
```
As the last line indicates, let us obtain the information required to connect our Docker client to the Docker daemon of the test-vm
```bash
$ ./docker-machine env test-vm
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://159.203.87.233:2376"
export DOCKER_CERT_PATH="/root/.docker/machine/machines/test-vm"
export DOCKER_MACHINE_NAME="test-vm"
# Run this command to configure your shell: 
# eval $(./docker-machine env test-vm)
```
So we need to eval a bash statement. This statement will set the environment variable the are displayed above.
```bash
$ eval $(./docker-machine env test-vm)
```
Once we are done with the eval, whatever commands we fire using Docker client (like `docker ps` or `docker run`) will be executed on the test-vm. To connect to another Docker daemon, we just need to do eval again to set the right environment variables.

This way, we can manage remote Docker Engines or daemons from a local machine.