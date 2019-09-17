# How to Kubernetes Deployment

## service packaging deploy

#### packaging : get/create docker image (dockerizing)
ref. [create docker image and container](../docker/create.image.n.container.md)

`create dockerize file`  
$ vi /apps/docker/images/Dockerfile.mobon.platform.gateway.aggregator
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

# 컨테이너 실행시 실행될 명령
CMD /bin/bash
```

`build/create docker image`  
docker build -t mobon/platform/gateway.aggregator:1.0 -t mobon/platform/gateway.aggregator:latest -f /apps/docker/images/Dockerfile.mobon.platform.gateway.aggregator .
>docker image tag mobon/platform/gateway/aggregator:1.0 mobon/platform/gateway/aggregator:latest

>`create docker service container`  
>$ docker run --name mobon.platform.gateway.aggregator.01 -d -p 8080:8080 -it mobon/platform/gateway.aggregator:latest 
>>$ docker run --net mobon.subnet --ip 192.168.104.91 --name mobon.platform.gateway.aggregator.01 -d -p 8080:8080 -p 18080:18080 -itmobon/platform/gateway.aggregator:latest
>
>$ docker exec -it mobon.platform.gateway.aggregator.01 /bin/bash

#### ReplicationController
`create kubernetes object file : ReplicationController`  
$ vi /apps/kubernetes/object/mobon.gateway.rc.yaml 
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: mobon.platform.gateway.aggregator.rc
spec:
  replicas: 3
  selector:
    app: mobon.platform.gateway.aggregator
  template:
    metadata:
      name: mobon.platform.gateway.aggregator.pod
      labels:
        app: mobon.platform.gateway.aggregator
    spec:
      containers:
      - name: mobon.platform.gateway.aggregator
        image: mobon/platform/gateway.aggregator:latest
        imagePullPolicy: Always
        volumeMounts:
          - name: app.source
            mountPath: /pgms/mobon.platform.gateway/source/git
            readOnly: true
        ports:
          - containerPort: 8080
        command: ["/bin/bash", "-c"]
        args:
          - ls -al /pgms/mobon.platform.gateway/source/git
    volumes:
     - name: app.source
       gitRepo:
            repository: http://172.20.0.7:9000/enliple/mobon/platform/gateway.git
            revision: master
            directory: .
```

#### Service
`create kubernetes object file : Service`  
$ vi /apps/kubernetes/object/mobon.gateway.svc.yaml 
```
apiVersion: v1
kind: Service
metadata:
  name: mobon.gateway.svc
spec:
  selector:
    app: mobon.gateway
  ports:
    - port: 80
      protocol: TCP
      targetPort: 8080
  type: LoadBalancer
```
