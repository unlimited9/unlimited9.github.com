# centos.7.base:1.1

## build image > create container

#### 01. build image mobon/centos.7.base:1.1(mobon/centos.7.base:latest) based docker.io/centos:7
`make directory`  
$ mkdir -p /apps/docker/images

`create dockerize file`  
$ vi /apps/docker/images/Dockerfile.centos.7.base 
```
FROM docker.io/centos:7

RUN yum clean all\
 && yum repolist \
 && yum -y update \
 && sed -i "s/en_US/all/" /etc/yum.conf \
 && yum -y reinstall glibc-common

RUN yum install -y net-tools iproute wget \
# && yum -y install tar unzip vi vim telnet curl openssl \
# && yum -y install apr apr-util apr-devel apr-util-devel \
# && yum -y install elinks locate python-setuptools \
# && yum -y bind-utils \
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
$ docker build -t mobon/centos.7.base:latest -f /apps/docker/images/Dockerfile.centos.7.base .  
>$ docker build -t mobon/centos.7.base:1.1 -t mobon/centos.7.base:latest -f /apps/docker/images/Dockerfile.centos.7.base .  
>$ docker image tag mobon/centos.7.base:1.1 mobon/centos.7.base:latest

>`push image to docker private registry`  
>$ docker tag mobon/centos.7.base:latest docker-registry.mobon.net:5000/mobon/centos.7.base:latest  
>$ docker push docker-registry.mobon.net:5000/mobon/centos.7.base:latest

#### 02. create private network - mobon.subnet
$ docker network create --driver=bridge --subnet=192.168.104.0/24 --gateway=192.168.104.10 mobon.subnet

#### 03. create containers base mobon/centos.7.base:1.1
$ docker run --net mobon.subnet --ip 192.168.104.51  --ulimit memlock=-1 --name mobon.elasticsearch.01 -d -it mobon/centos.7.base:1.1
 
$ docker run --net mobon.subnet --ip 192.168.104.52  --ulimit memlock=-1 --name mobon.elasticsearch.02 -d -it mobon/centos.7.base:1.1
 
$ docker run --net mobon.subnet --ip 192.168.104.41 --name mobon.mongodb.01 -d -it mobon/centos.7.base:1.1
 
$ docker run --net mobon.subnet --ip 192.168.104.42 --name mobon.mongodb.02 -d -it mobon/centos.7.base:1.1
 
$ docker run --net mobon.subnet --ip 192.168.104.43 --name mobon.mongodb.03 -d -it mobon/centos.7.base:1.1

## create container > create image > create container

#### 01. create base.container

`create container base centos:7 - centos.7.app`  
$ docker run -d -it --name centos.7.app centos:7 /bin/bash  
$ docker exec -it  centos.7.app  /bin/bash

>$ docker run --privileged --name centos.7.app -d centos:7 init  
>>--privileged
>: give extended privileges to this container
(container에서 host의 linux kernel 기능을 모두 사용)
>
>$ docker exec -it  centos.7.app  /bin/bash
>#systemctl --version

`install utility package`  
$ yum install -y net-tools  
$ yum install -y iproute

`create group/user : app`  
$ groupadd -g 3000 app  
$ useradd -d /apps -g 3000 -m -u 3000 -s /bin/bash app  
$ passwd app

`create directory`  
$ su - app

$ mkdir -p /apps/install  
$ mkdir -p /pgms  
$ mkdir -p /data  
$ mkdir -p /logs

#### 02. create image from base.container - mobon/apps.server:1.1
$ docker commit -a "sjlee@ruaniz.com" -m "create image from centos.7.app container" centos.7.app mobon/apps.server:1.1

#### 03. create private network - mobon.subnet
$ docker network create --driver=bridge --subnet=192.168.104.0/24 --gateway=192.168.104.10 mobon.subnet

#### 04. create containers base mobon/apps.server:1.1
$ docker run --net mobon.subnet --ip 192.168.104.51  --ulimit memlock=-1 --name mobon.elasticsearch.01 -d -it mobon/apps.server:1.1
 
$ docker run --net mobon.subnet --ip 192.168.104.52  --ulimit memlock=-1 --name mobon.elasticsearch.02 -d -it mobon/apps.server:1.1
 
$ docker run --net mobon.subnet --ip 192.168.104.41 --name mobon.mongodb.01 -d -it mobon/apps.server:1.1
 
$ docker run --net mobon.subnet --ip 192.168.104.42 --name mobon.mongodb.02 -d -it mobon/apps.server:1.1
 
$ docker run --net mobon.subnet --ip 192.168.104.43 --name mobon.mongodb.03 -d -it mobon/apps.server:1.1
 
# java.app.env:1.1

## build image

#### 01. build image mobon/java.app.env:1.1(mobon/java.app.env:latest) based mobon/centos.7.base:latest

`create dockerize file`  
$ vi /apps/docker/images/Dockerfile.java.app.env
```
FROM mobon/centos.7.base:latest
# FROM docker-registry.mobon.net:5000/mobon/centos.7.base:latest

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
RUN unzip gradle-5.6.2-bin.zip

RUN mkdir -p /apps/gradle
RUN mv /apps/install/gradle-5.6.2 /apps/gradle/5.6.2

USER root

# set env java development kit
RUN echo 'export JAVA_HOME=/apps/jdk/12.0.2' >> /etc/profile.d/java.sh
RUN echo 'export PATH=$PATH:$JAVA_HOME/bin' >> /etc/profile.d/java.sh
RUN echo 'export CLASSPATH=.:$JAVA_HOME/jre/lib:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar' >> /etc/profile.d/java.sh

# set env gradle
RUN echo 'export GRADLE_HOME=/apps/gradle/5.6.2' >> /etc/profile.d/gradle.sh
RUN echo 'export PATH=$PATH:$GRADLE_HOME/bin' >> /etc/profile.d/gradle.sh

RUN source /etc/profile

USER app

# 컨테이너 실행시 실행될 명령
CMD /bin/bash
```

`build/create docker image`  
$ docker build -t mobon/java.app.env:latest -f /apps/docker/images/Dockerfile.java.app.env .
>$ docker build -t mobon/java.app.env:1.1 -t mobon/java.app.env:latest -f /apps/docker/images/Dockerfile.java.app.env .
>$ docker image tag mobon/java.app.env:1.1 mobon/java.app.env:latest

>`create docker service container`  
>$ docker run --name mobon.service.01 -d -p 8080:8080 -it mobon/java.app.env:latest 
>>$ docker run --net mobon.subnet --ip 192.168.104.91 --name mobon.service.01 -d -p 8080:8080 -p 18080:18080 -it mobon/java.app.env:latest
>
>$ docker exec -it mobon.service.01 /bin/bash

>`push image to docker private registry`  
>$ docker tag mobon/java.app.env:latest docker-registry.mobon.net:5000/mobon/java.app.env:latest
>$ docker push docker-registry.mobon.net:5000/mobon/java.app.env:latest

# java.app.ext:1.1

## build image

#### 01. build image mobon/java.app.ext:1.1(mobon/java.app.ext:latest) based mobon/java.app.env:latest

`create dockerize file`  
$ vi /apps/docker/images/Dockerfile.java.app.ext
```
FROM mobon/java.app.env:latest
# FROM docker-registry.mobon.net:5000/mobon/java.app.env:latest 

# install and setup application
WORKDIR /apps/install

# install scouter agent
RUN wget https://github.com/scouter-project/scouter/releases/download/v2.7.0/scouter-min-2.7.0.tar.gz
RUN tar xvf scouter-min-2.7.0.tar.gz

RUN mkdir /apps/scouter
RUN mv /apps/install/scouter /apps/scouter/2.7.0

```
>```
># set host agent
>RUN mv /apps/scouter/2.7.0/agent.host/conf/scouter.conf /apps/scouter/2.7.0/agent.host/conf/scouter.conf.org
>RUN echo '### scouter host configruation sample' >> /apps/scouter/2.7.0/agent.host/conf/scouter.conf
>RUN echo 'net_collector_ip=192.168.100.8' >> /apps/scouter/2.7.0/agent.host/conf/scouter.conf
>RUN echo '#net_collector_udp_port=6100' >> /apps/scouter/2.7.0/agent.host/conf/scouter.conf
>RUN echo '#net_collector_tcp_port=6100' >> /apps/scouter/2.7.0/agent.host/conf/scouter.conf
>RUN echo '#cpu_warning_pct=80' >> /apps/scouter/2.7.0/agent.host/conf/scouter.conf
>RUN echo '#cpu_fatal_pct=85' >> /apps/scouter/2.7.0/agent.host/conf/scouter.conf
>RUN echo '#cpu_check_period_ms=60000' >> /apps/scouter/2.7.0/agent.host/conf/scouter.conf
>RUN echo '#cpu_fatal_history=3' >> /apps/scouter/2.7.0/agent.host/conf/scouter.conf
>RUN echo '#cpu_alert_interval_ms=300000' >> /apps/scouter/2.7.0/agent.host/conf/scouter.conf
>RUN echo '#disk_warning_pct=88' >> /apps/scouter/2.7.0/agent.host/conf/scouter.conf
>RUN echo '#disk_fatal_pct=92' >> /apps/scouter/2.7.0/agent.host/conf/scouter.conf
>
>RUN cd /apps/scouter/2.7.0/agent.host; ./host.sh;
>
># set java agent
># RUN cp /apps/scouter/2.7.0/agent.java/conf/scouter.conf /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
>RUN echo '### scouter java agent configuration sample' >> /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
>RUN echo '#obj_name=mdev-was-01-product' >> /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
>RUN echo '# Scouter Server IP Address (Default : 127.0.0.1)' >> /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
>RUN echo 'net_collector_ip=172.20.108' >> /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
>RUN echo '# Scouter Server Port (Default : 6100)' >> /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
>RUN echo '#net_collector_udp_port=6100' >> /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
>RUN echo '#net_collector_tcp_port=6100' >> /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
>RUN echo '#hook_method_patterns=sample.mybiz.*Biz.*,sample.service.*Service.*' >> /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
>RUN echo 'trace_http_client_ip_header_key=X-Forwarded-For' >> /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
>RUN echo '#profile_spring_controller_method_parameter_enabled=false' >> /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
>RUN echo '#hook_exception_class_patterns=my.exception.TypedException' >> /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
>RUN echo '#profile_fullstack_hooked_exception_enabled=true' >> /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
>RUN echo '#hook_exception_handler_method_patterns=my.AbstractAPIController.fallbackHandler,my.ApiExceptionLoggingFilter.handleNotFoundErrorResponse' >> /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
>RUN echo '#hook_exception_hanlder_exclude_class_patterns=exception.BizException' >> /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
>RUN echo '' >> /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
>RUN echo '### HTTP header/parameter profile' >> /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
>RUN echo 'profile_http_parameter_enabled=true' >> /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
>RUN echo 'profile_http_header_enabled=true' >> /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
>RUN echo '' >> /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
>RUN echo '### ignore low response time(ms)' >> /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
>RUN echo 'xlog_lower_bound_time_ms=300' >> /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
>RUN echo '' >> /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
>RUN echo '### method profile' >> /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
>RUN echo '#hook_method_patterns=com.xxx.controller*.*,com.xxx.service*.*,com.xxx.dao*.*' >> /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
>RUN echo '#hook_method_ignore_classes=com.xxx.dto*.*' >> /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
>
>#scouter.agent.java
>RUN export SCOUTER_DIR=/apps/scouter/2.7.0  
>RUN export JAVA_OPTS="$JAVA_OPTS -javaagent:$SCOUTER_DIR/agent.java/scouter.agent.jar"
>RUN export JAVA_OPTS="$JAVA_OPTS -Dscouter.config=$SCOUTER_DIR/agent.java/conf/scouter-product.conf"
>RUN export JAVA_OPTS="$JAVA_OPTS -Dobj_name=product-01"
>
>RUN java $JAVA_OPTSD -jar ./mobon.gateway.aggregation.service-1.0.0.jar
>```

`build/create docker image`  
$ docker build -t mobon/java.app.ext:latest -f /apps/docker/images/Dockerfile.java.app.ext .
>$ docker build -t mobon/java.app.ext:1.1 -t mobon/java.app.ext:latest -f /apps/docker/images/Dockerfile.java.app.ext .
>$ docker image tag mobon/java.app.ext:1.1 mobon/java.app.ext:latest

>`create docker service container`  
>$ docker run --name mobon.service.01 -d -p 8080:8080 -it mobon/java.app.ext:latest 
>>$ docker run --net mobon.subnet --ip 192.168.104.91 --name mobon.service.01 -d -p 8080:8080 -p 18080:18080 -it mobon/java.app.ext:latest
>
>$ docker exec -it mobon.service.01 /bin/bash

>`push image to docker private registry`  
>$ docker tag mobon/java.app.ext:latest docker-registry.mobon.net:5000/mobon/java.app.ext:latest
>$ docker push docker-registry.mobon.net:5000/mobon/java.app.ext:latest

# node.app.env:1.1

## build image

#### 01. build image mobon/node.app.env:1.1(mobon/node.app.env:latest) based docker.io/node
`make directory`  
$ mkdir -p /apps/docker/images

`create dockerize file`  
$ vi /apps/docker/images/Dockerfile.node.app.env 
```
FROM docker.io/node

## create group/user : app/app
#RUN groupadd -g 3000 app
#RUN useradd -d /apps -g 3000 -m -u 3000 -s /bin/bash app

RUN mkdir -p /pgms /data /logs
#RUN chown -R app.app /pgms /data /logs

#USER app

WORKDIR /apps

# create directory 
RUN mkdir -p /apps/install

# 컨테이너 실행시 실행될 명령
CMD /bin/bash
```

`build/create docker image`  
$ docker build -t mobon/node.app.env:latest -f /apps/docker/images/Dockerfile.node.app.env .  
>$ docker build -t mobon/node.app.env:1.1 -t mobon/node.app.env:latest -f /apps/docker/images/Dockerfile.node.app.env .  
>$ docker image tag mobon/node.app.env:1.1 mobon/node.app.env:latest

>`push image to docker private registry`  
>$ docker tag mobon/node.app.env:latest docker-registry.mobon.net:5000/mobon/node.app.env:latest  
>$ docker push docker-registry.mobon.net:5000/mobon/node.app.env:latest

# node.app.ext:1.1

## build image

#### 01. build image mobon/node.app.ext:1.1(mobon/node.app.ext:latest) based mobon/node.app.env:latest

`create dockerize file`  
$ vi /apps/docker/images/Dockerfile.node.app.ext
```
FROM mobon/node.app.env:latest
# FROM docker-registry.mobon.net:5000/mobon/node.app.env:latest 

# install and setup application
WORKDIR /apps/install

#USER root

ENV DEBIAN_FRONTEND teletype

# Before adding Influx repository, run this so that apt will be able to read the repository.
RUN apt-get update && apt-get install -y --no-install-recommends apt-utils
RUN apt-get install -y sudo apt-transport-https

# Add the InfluxData key
RUN curl -sL https://repos.influxdata.com/influxdb.key | apt-key add -
#RUN wget -qO- https://repos.influxdata.com/influxdb.key | sudo apt-key add -

RUN echo "deb https://repos.influxdata.com/debian $(. /etc/os-release; echo $VERSION_CODENAME) stable" | tee /etc/apt/sources.list.d/influxdb.list
#RUN cat /etc/apt/sources.list.d/influxdb.list

# install and start the Telegraf service
RUN apt-get update && apt-get install -y telegraf
#RUN service telegraf start
#RUN systemctl start telegraf

#USER app
```

`build/create docker image`  
$ docker build -t mobon/node.app.ext:latest -f /apps/docker/images/Dockerfile.node.app.ext .  
>$ docker build --no-cache -t mobon/node.app.ext:latest -f /apps/docker/images/Dockerfile.node.app.ext .  
>$ docker build -t mobon/node.app.ext:1.1 -t mobon/node.app.ext:latest -f /apps/docker/images/Dockerfile.node.app.ext .  
>$ docker image tag mobon/node.app.ext:1.1 mobon/node.app.ext:latest

>`create docker service container`  
>$ docker run --name mobon.service.01 -d -p 8080:8080 -it mobon/node.app.ext:latest 
>>$ docker run --net mobon.subnet --ip 192.168.104.91 --name mobon.service.01 -d -p 8080:8080 -p 18080:18080 -it mobon/node.app.ext:latest
>
>$ docker exec -it mobon.service.01 /bin/bash

>`push image to docker private registry`  
>$ docker tag mobon/node.app.ext:latest docker-registry.mobon.net:5000/mobon/node.app.ext:latest  
>$ docker push docker-registry.mobon.net:5000/mobon/node.app.ext:latest


