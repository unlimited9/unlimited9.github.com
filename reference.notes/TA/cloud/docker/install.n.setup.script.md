groupadd -g 3000 app  
useradd -d /apps -g 3000 -m -u 3000 -s /bin/bash app  
passwd app

mkdir -p /apps/install  
mkdir -p /pgms  
mkdir -p /data  
mkdir -p /logs

chown -R app:app /apps  
chown -R app:app /pgms  
chown -R app:app /data  
chown -R app:app /logs  

su - app

mkdir -p /data/docker

cd /apps/install  
curl -O http://mirror.centos.org/centos/7/extras/x86_64/Packages/container-selinux-2.99-1.el7_6.noarch.rpm  
sudo rpm -Uvh container-selinux-2.99-1.el7_6.noarch.rpm 

sudo yum install -y policycoreutils-python

curl -fsSL https://get.docker.com/ | sudo sh  
sudo usermod -aG docker app

sudo systemctl enable docker  
sudo systemctl start docker

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
 

