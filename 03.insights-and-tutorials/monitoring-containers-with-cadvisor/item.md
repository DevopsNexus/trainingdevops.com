---
title: 'Monitoring Containers with cAdvisor'
summary:
    enabled: '1'
    format: short
---

Monitoring the resource consumption of a Docker container and the host on which it is running is very important. Such monitoring often presents with insights about the infrastructure and is very helpful in debugging unexpected behaviors.
[cAdvisor](https://github.com/google/cadvisor/) is a monitoring tool, open sourced by Google which can present the monitoring and performance data in various forms. cAdvisor run a daemon and collects the data per container and for overall host. This data can be consumed in the following ways:

- **cAdvisor web interface**
  cAdvisor daemon exposes a web interface with graphs and performance data. Downside to this is that the historical data is not stored. The data presented is real time which makes it great for debugging.
- **[Influxdb](https://docs.influxdata.com/influxdb/v0.10/)**
  Influxdb is a time series database. cAdvisor can push the monitoring data to Influxdb where we can either query for the data by their client or manually build dashboards using Grafana or Chronograf. 
- **Rest API**
  cAdvisor also exposes a rest API which can be used to consume the monitoring data by adhoc applications and build logic over it.
- **[Elasticsearch](https://www.elastic.co/products/elasticsearch) (pre-release)**
In pre release version, we can use Elasticsearch to store the monitoring data.

cAdvisor can run on a host in two ways, as a standalone binary or as a Docker container. We will see both the ways in this post.

## cAdvisor as a Standalone Binary
cAdvisor publishes standalone binaries on their [release page](https://github.com/google/cadvisor/releases). These binaries can be downloaded and run directly.
```bash
$ wget https://github.com/google/cadvisor/releases/download/v0.20.5/cadvisor
$ chmod 755 cadvisor
$ sudo ./cadvisor 
```
By default, cAdvisor runs the web interface on port 8080. So we will go on our browser and checkout the web interface at [http://localhost:8080](http://localhost:8080). Note that cAdvisor is aware of cgroups and namespaces. So, in addition to the containers, processes running on the hosts are also visible and can be monitored. 

## cAdvisor as a Docker Container
For environments where running a binary is not possible, cAdvisor provides a docker container which can be used to run the cAdvisor daemon. 
```bash
$ sudo docker run --volume=/:/rootfs:ro --volume=/var/run:/var/run:rw --volume=/sys:/sys:ro --volume=/var/lib/docker/:/var/lib/docker:ro --publish=8080:8080 --detach=true --name=cadvisor google/cadvisor:latest
```
Note that on Red Hat, CentOS and Fedora, we may need to pass `--privileged=true` because of security measures enforced on the containers by SELinux (it is a good thing). 

Now we can just browse the monitoring graphs at [http://localhost:8080](http://localhost:8080). 

![enter image description here](https://www.dropbox.com/s/auhmz61mbq1gabj/cadvisor.png?dl=1)