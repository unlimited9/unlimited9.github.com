# kafka

## build image

#### 01. build image mobon/kafka.2.4.0(mobon/kafka.2:latest) based mobon/java.app.env:latest
`create dockerize file`  
$ vi /apps/docker/images/kafka.2/Dockerfile
```
FROM docker-registry.mobon.net:5000/mobon/java.app.env:latest
# FROM mobon/java.app.ext:latest
#

USER app

# install and setup application
WORKDIR /apps/install

RUN mkdir -p /apps/kafka /data/kafka /logs/kafka

# install kafka
RUN curl -O http://apache.mirror.cdnetworks.com/kafka/2.4.0/kafka_2.13-2.4.0.tgz
RUN tar -zxvf /apps/install/kafka_2.13-2.4.0.tgz
RUN mv /apps/install/kafka_2.13-2.4.0 /apps/kafka/2.13-2.4.0

RUN sed -i -e 's/^log.dirs=\/tmp\/kafka-logs$/log.dirs=\/logs\/kafka/' /apps/kafka/2.13-2.4.0/config/server.properties

RUN echo '' >> /apps/kafka/2.13-2.4.0/config/server.properties
RUN echo '# Switch to enable topic deletion or not, default value is false' >> /apps/kafka/2.13-2.4.0/config/server.properties
RUN echo 'delete.topic.enable=true' >> /apps/kafka/2.13-2.4.0/config/server.properties
RUN echo '' >> /apps/kafka/2.13-2.4.0/config/server.properties

WORKDIR /apps/kafka

# 컨테이너 실행시 실행될 명령
CMD /bin/bash
```

`build/create docker image`  
$ docker build -t mobon/kafka.2:latest -f /apps/docker/images/kafka.2/Dockerfile .  
>$ docker build --no-cache -t mobon/kafka.2:latest -f /apps/docker/images/kafka.2/Dockerfile .  
>$ docker build -t mobon/kafka.2.4.0 -t mobon/kafka.2:latest -f /apps/docker/images/kafka.2/Dockerfile .  
>$ docker image tag mobon/kafka.2.4.0 mobon/kafka.2:latest

>`create docker service container`  
>$ docker run --net mobon.subnet --ip 192.168.104.51  --ulimit memlock=-1 --name mobon.kafka.01 -d -p 7642:7642 -it mobon/kafka.2:latest

>$ docker exec -it mobon.kafka.01 /bin/bash

>`push image to docker private registry`  
>$ docker tag mobon/kafka.2:latest docker-registry.mobon.net:5000/mobon/kafka.2:latest  
>$ docker push docker-registry.mobon.net:5000/mobon/kafka.2:latest


<details>
<summary>build image > create container</summary>
<div markdown="1">

#### create containers base mobon/centos.7.base:1.1
$ docker run --net mobon.subnet --ip 192.168.104.51  --ulimit memlock=-1 --name kafka.2 -d -p 7642:7642 -it docker-registry.mobon.net:5000/mobon/java.app.env:latest

$ docker exec -it kafka.2 /bin/bash

$$ mkdir -p /apps/kafka /data/kafka /logs/kafka

$$ cd /apps/install

$$ curl -O http://apache.mirror.cdnetworks.com/kafka/2.4.0/kafka_2.13-2.4.0.tgz
$$ tar -zxvf /apps/install/kafka_2.13-2.4.0.tgz
$$ mv /apps/install/kafka_2.13-2.4.0 /apps/kafka/2.13-2.4.0

$$ vi /apps/kafka/2.13-2.4.0/config/server.properties
```
...  
# 클러스터 구성 시 각 노드에 id값을 주어, 각 노드를 식별하는 역할 (다른 노드와 중복되면 안됨)  
broker.id=0  
...  
# 현재 호스트 머신의 IP를 적어준다. 앞에 PLAINTEXT 부분은 kafka에서 지원하는 전송 프로토콜로, 자세한건 공식홈을 참조하는 것이 정확하다.  
listeners=PLAINTEXT://:7642  
# 현재 호스트 머신의 IP를 적어준다. (Producer, Consumer가 참조하게 되는 IP)  
advertised.listeners=PLAINTEXT://10.10.10.25:7642  
...  
log.dirs=/logs/kafka  
...  
zookeeper.connect=10.10.10.25:7214,10.10.10.26:7214,10.10.10.27:7214  
...  
# Switch to enable topic deletion or not, default value is false  
delete.topic.enable=true
```

$$ vi /apps/kafka/kafka
```
#!/bin/sh  
# kafka kafka service shell  
# chkconfig: 2345 90 90  
# description: kafka  
# processname: kafka-server-start.sh  
# config: $KAFKA_CONF  
# pidfile:

KAFKA_HOME='/apps/kafka/2.12-2.2.0'

export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G"

case "$1" in  
  start)
    echo -en "Starting Kafka Server...\n"  
    $KAFKA_HOME/bin/kafka-server-start.sh -daemon $KAFKA_HOME/config/server.properties  
    echo -e "\n"  
    ;;
    
  stop)
    echo -en "Shutting Down Kafka Server...\n"  
    $KAFKA_HOME/bin/kafka-server-stop.sh  
    echo -e "\n"  
    ;;
  
  restart)
    $0 stop  
    sleep 5  
    $0 start  
    ;;
  
  *)
    echo "Usage: $0 {start|stop|restart}"  
    exit 1

esac

exit 0

```
$$ chmod 755 /apps/kafka/kafka  

$$ /apps/kafka/kafka start

#### create image from base.container - mobon/kafka.5:latest
$ docker commit -a "sjlee@ruaniz.com" -m "create image from kafka.2 container" kafka.2 mobon/kafka.2:latest

>`push image to docker private registry`  
>$ docker tag mobon/kafka.2:latest docker-registry.mobon.net:5000/mobon/kafka.2:latest  
>$ docker push docker-registry.mobon.net:5000/mobon/kafka.2:latest

</div>
</details>
