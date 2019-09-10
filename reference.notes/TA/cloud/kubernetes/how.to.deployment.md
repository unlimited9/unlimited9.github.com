# How to Kubernetes Deployment

## 

#### packaging : get/create docker image (dockerizing)
ref. [create docker image and container](/reference.notes/TA/cloud/docker/create.image.n.container.md)

`create docker service container`  
$ docker run --name mobon.service.01 -d -p 8080:8080 -it mobon/centos.7.base:latest
>$ docker run --net mobon.subnet --ip 192.168.104.91 --name mobon.service.01 -d -p 8080:8080 -p 18080:18080 -it mobon/centos.7.base:latest

$ docker exec -it mobon.service.01 /bin/bash

ref. [install and setup apache tomcat](TA/apache.tomcat/install.n.setup.md) ([install script](TA/apache.tomcat/install.n.setup.script.md)) : Tomcat Servlet Container - Multi Instances


#### 
