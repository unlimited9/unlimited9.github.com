# centos.7.base:1.1

## build image > create container

#### 01. build image mobon/centos.7.base:1.1(mobon/centos.7.base:lastest) based docker.io/centos:7
`make directory`  
mkdir -p /apps/docker/images

`create dockerize file`  
vi /apps/docker/images/Dockerfile.centos.7.base 
```
FROM docker.io/centos:7

RUN yum clean all\
 && yum repolist \
 && yum -y update \
 && sed -i "s/en_US/all/" /etc/yum.conf \
 && yum -y reinstall glibc-common

RUN yum install -y net-tools iproute \
# && yum -y install tar unzip vi vim telnet curl openssl \
# && yum -y install apr apr-util apr-devel apr-util-devel \
# && yum -y install elinks locate python-setuptools \
 && yum clean all

ENV LANG=ko_KR.utf8 TZ=Asia/Seoul

# create group/user : app/app
RUN groupadd -g 3000 app
RUN useradd -d /apps -g 3000 -m -u 3000 -s /bin/bash app

# create basic directory 
RUN mkdir -p /pgms /data /logs
RUN chown -R app.app /pgms /data /logs

USER app
WORKDIR /apps

# create install directory 
RUN mkdir -p /apps/install

# 컨테이너 실행시 실행될 명령
CMD /bin/bash
```

`build/create docker image`  
docker build -t mobon/centos.7.base:1.1 -t mobon/centos.7.base:latest -f /apps/docker/images/Dockerfile.centos.7.base .
>docker image tag mobon/centos.7.base:1.1 mobon/centos.7.base:latest

#### 02. create private network - mobon.subnet
docker network create --driver=bridge --subnet=192.168.104.0/24 --gateway=192.168.104.10 mobon.subnet

#### 03. create containers base mobon/centos.7.base:1.1
docker run --net mobon.subnet --ip 192.168.104.51  --ulimit memlock=-1 --name mobon.elasticsearch.01 -d -it mobon/centos.7.base:1.1
 
docker run --net mobon.subnet --ip 192.168.104.52  --ulimit memlock=-1 --name mobon.elasticsearch.02 -d -it mobon/centos.7.base:1.1
 
docker run --net mobon.subnet --ip 192.168.104.41 --name mobon.mongodb.01 -d -it mobon/centos.7.base:1.1
 
docker run --net mobon.subnet --ip 192.168.104.42 --name mobon.mongodb.02 -d -it mobon/centos.7.base:1.1
 
docker run --net mobon.subnet --ip 192.168.104.43 --name mobon.mongodb.03 -d -it mobon/centos.7.base:1.1
 
## create container > create image > create container

#### 01. create base.container

`create container base centos:7 - centos.7.app`  
docker run -d -it --name centos.7.app centos:7 /bin/bash  
docker exec -it  centos.7.app  /bin/bash

> docker run --privileged --name centos.7.app -d centos:7 init  
>>--privileged
>: give extended privileges to this container
(container에서 host의 linux kernel 기능을 모두 사용)
>
>docker exec -it  centos.7.app  /bin/bash
>#systemctl --version

`install utility package`  
yum install -y net-tools  
yum install -y iproute

`create group/user : app`  
groupadd -g 3000 app  
useradd -d /apps -g 3000 -m -u 3000 -s /bin/bash app  
passwd app

`create directory`  
su - app

mkdir -p /apps/install  
mkdir -p /pgms  
mkdir -p /data  
mkdir -p /logs

#### 02. create image from base.container - mobon/apps.server:1.1
docker commit -a "sjlee@ruaniz.com" -m "create image from centos.7.app container" centos.7.app mobon/apps.server:1.1

#### 03. create private network - mobon.subnet
docker network create --driver=bridge --subnet=192.168.104.0/24 --gateway=192.168.104.10 mobon.subnet

#### 04. create containers base mobon/apps.server:1.1
docker run --net mobon.subnet --ip 192.168.104.51  --ulimit memlock=-1 --name mobon.elasticsearch.01 -d -it mobon/apps.server:1.1
 
docker run --net mobon.subnet --ip 192.168.104.52  --ulimit memlock=-1 --name mobon.elasticsearch.02 -d -it mobon/apps.server:1.1
 
docker run --net mobon.subnet --ip 192.168.104.41 --name mobon.mongodb.01 -d -it mobon/apps.server:1.1
 
docker run --net mobon.subnet --ip 192.168.104.42 --name mobon.mongodb.02 -d -it mobon/apps.server:1.1
 
docker run --net mobon.subnet --ip 192.168.104.43 --name mobon.mongodb.03 -d -it mobon/apps.server:1.1
 

# java.app.env:1.1

## build image

#### 01. build image mobon/java.app.env:1.1(mobon/java.app.env:lastest) based mobon/centos.7.base:latest

`create dockerize file`  
$ vi /apps/docker/images/Dockerfile.java.app.env
```
FROM mobon/centos.7.base:latest

USER root

RUN yum -y install unzip

USER app

# install and setup application
WORKDIR /apps/install

# install java development kit
RUN curl -O https://download.java.net/java/GA/jdk12.0.2/e482c34c86bd4bf8b56c0b35558996b9/10/GPL/openjdk-12.0.2_linux-x64_bin.tar.gz
RUN tar xvf openjdk-12.0.2_linux-x64_bin.tar.gz

RUN mkdir /apps/jdk
RUN mv /apps/install/jdk-12.0.2 /apps/jdk/12.0.2

# install gradle
RUN curl -O https://downloads.gradle-dn.com/distributions/gradle-5.6.2-bin.zip

RUN mkdir -p /apps/gradle/5.6.2
RUN unzip -d /apps/gradle/5.6.2 gradle-5.6.2-bin.zip

USER root

# set env java development kit
RUN echo "export JAVA_HOME=$/apps/jdk/12.0.2\n" \
  "export PATH=$PATH:$JAVA_HOME/bin\n" \
  "export CLASSPATH=.:$JAVA_HOME/jre/lib:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar" \
  >> /etc/profile.d/jdk.sh

# set env gradle
RUN echo "export GRADLE_HOME=/apps/gradle/5.6.2\n" \
  "export PATH=${GRADLE_HOME}/bin:${PATH}\n" \
  >> /etc/profile.d/gradle.sh

RUN source /etc/profile

USER app

# 컨테이너 실행시 실행될 명령
CMD /bin/bash
```

`build/create docker image`  
$ docker build -t mobon/java.app.env:1.1 -t mobon/java.app.env:latest -f /apps/docker/images/Dockerfile.java.app.env .
>$ docker image tag mobon/java.app.env:1.1 mobon/java.app.env:latest

>`create docker service container`  
>$ docker run --name mobon.service.01 -d -p 8080:8080 -it mobon/java.app.env:latest 
>>$ docker run --net mobon.subnet --ip 192.168.104.91 --name mobon.service.01 -d -p 8080:8080 -p 18080:18080 -it mobon/java.app.env:latest
>
>$ docker exec -it mobon.service.01 /bin/bash
