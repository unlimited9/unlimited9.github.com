# How to Kubernetes Deployment

## service packaging deploy

#### packaging : get/create docker image (dockerizing)
ref. [create docker image and container](../docker/create.image.n.container.md)

`create dockerize file`  
$ vi /apps/docker/images/Dockerfile.mobon.gateway 
```
FROM mobon/centos.7.base:latest

USER app

# install and setup application
WORKDIR /apps/install

# set Java Development Kit
RUN curl -O https://download.java.net/java/GA/jdk12.0.2/e482c34c86bd4bf8b56c0b35558996b9/10/GPL/openjdk-12.0.2_linux-x64_bin.tar.gz
RUN tar xvf openjdk-12.0.2_linux-x64_bin.tar.gz

RUN mkdir /apps/jdk
RUN mv /apps/install/jdk-12.0.2 /apps/jdk/12.0.2

RUN cat > /etc/profile.d/jdk.sh <<EOF
export JAVA_HOME=$/apps/jdk/12.0.2
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/jre/lib:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
EOF
RUN source /etc/profile


# run spring-boot application
RUN java -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8000,suspend=n -jar /pgms/tomcat/wars/.jar




# set tomcat servlet container
#RUN curl -O http://mirror.navercorp.com/apache/tomcat/tomcat-9/v9.0.24/bin/apache-tomcat-9.0.24.tar.gz
#RUN tar xvf apache-tomcat-9.0.24.tar.gz
#
# create application directory 
#RUN mkdir /apps/tomcat
#RUN mv /apps/install/apache-tomcat-9.0.24 /apps/tomcat/9.0.24
#
#RUN mkdir -p /data/tomcat /logs/tomcat /pgms/tomcat/webapps /pgms/tomcat/wars /pgms/tomcat/backup
#
#RUN mkdir -p \
#/apps/tomcat/instances/mobon.gateway.01/bin \
#/apps/tomcat/instances/mobon.gateway.01/conf \
#/apps/tomcat/instances/mobon.gateway.01/logs \
#/apps/tomcat/instances/mobon.gateway.01/temp \
#/apps/tomcat/instances/mobon.gateway.01/webapps/ROOT
#
#RUN cp -R /apps/tomcat/9.0.24/conf/* /apps/tomcat/instances/mobon.gateway.01/conf
#

# 컨테이너 실행시 실행될 명령
CMD /bin/bash
```

`build/create docker image`  
docker build -t mobon/centos.7.base:1.1 -t mobon/centos.7.base:latest -f /apps/docker/images/Dockerfile.centos.7.base .
>docker image tag mobon/centos.7.base:1.1 mobon/centos.7.base:latest

>`create docker service container`  
>$ docker run --name mobon.gateway.01 -d -p 8080:8080 -it mobon/centos.7.base:latest
>>$ docker run --net mobon.subnet --ip 192.168.104.91 --name mobon.gateway.01 -d -p 8080:8080 -p 18080:18080 -it mobon/centos.7.base:latest
>
>$ docker exec -it mobon.gateway.01 /bin/bash
>
>`install and setup application`  
>ref. [install and setup apache tomcat](../../apache.tomcat/install.n.setup.md) ([install script](../../apache.tomcat/install.n.setup.script.md)) : Tomcat Servlet Container - Multi Instances

#### register ReplicationController

`create kubernetes object file : ReplicationController`  

$ vi /apps/kubernetes/object/mobon.gateway.rc.yaml 

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: mobon.gateway.rc
spec:
  replicas: 3
  selector:
    app: mobon.gateway
  template:
    metadata:
      name: mobon.gateway.pod
      labels:
        app: mobon.gateway
    spec:
      containers:
      - name: mobon.gateway
        image: mobon/centos.7.base:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
```
