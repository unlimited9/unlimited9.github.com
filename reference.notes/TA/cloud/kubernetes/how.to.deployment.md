# How to Kubernetes Deployment

## service packaging deploy

#### packaging : get/create docker image (dockerizing)
ref. [create docker image and container](../docker/create.image.n.container.md)

`build/create docker image`  
docker build -t mobon/centos.7.base:1.1 -t mobon/centos.7.base:latest -f /apps/docker/images/Dockerfile.centos.7.base .
>docker image tag mobon/centos.7.base:1.1 mobon/centos.7.base:latest

`create docker service container`  
$ docker run --name mobon.service.01 -d -p 8080:8080 -it mobon/centos.7.base:latest
>$ docker run --net mobon.subnet --ip 192.168.104.91 --name mobon.service.01 -d -p 8080:8080 -p 18080:18080 -it mobon/centos.7.base:latest

$ docker exec -it mobon.service.01 /bin/bash

`install and setup application`  
ref. [install and setup apache tomcat](../../apache.tomcat/install.n.setup) ([install script](../../apache.tomcat/install.n.setup.script.md)) : Tomcat Servlet Container - Multi Instances


#### 

`create dockerize file`  
vi /apps/docker/images/Dockerfile.mobon.service 
```
FROM mobon/centos.7.base:latest

USER app
WORKDIR /apps



# 컨테이너 실행시 실행될 명령
CMD /bin/bash
```
