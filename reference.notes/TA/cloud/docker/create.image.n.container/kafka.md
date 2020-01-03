# kafka

## build image

#### 01. build image mobon/kafka.3.5.6(mobon/kafka.2:latest) based mobon/java.app.env:latest

`create kafka config files : zoo.cfg`  
$ vi /apps/docker/images/kafka.2/config/zoo.cfg
```
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial synchronization phase can take
initLimit=10
# The number of ticks that can pass between sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored. do not use /tmp for storage, /tmp here is just example sakes.
dataDir=/data/kafka
# the port at which the clients will connect
clientPort=7214
# the port address at which the clients will connect
clientPortAddress=0.0.0.0
# the maximum number of client connections. increase this if you need to handle more clients
#maxClientCnxns=60

# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours. Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1

# Set ensemble(cluster) as
# server.{server id[1-255]} = {hostname}:{port to connect followers to the leader}:{port for leader election}
server.1=mobon-data-kafka-0.mobon-data-kafka-svc:7224:7234
server.2=mobon-data-kafka-1.mobon-data-kafka-svc:7224:7234
server.3=mobon-data-kafka-2.mobon-data-kafka-svc:7224:7234

```

`create dockerize file`  
$ vi /apps/docker/images/kafka.3/Dockerfile
```
FROM docker-registry.mobon.net:5000/mobon/java.app.env:latest
# FROM mobon/java.app.ext:latest
#

USER app

# install and setup application
WORKDIR /apps/install

RUN mkdir -p /apps/kafka /data/kafka/db /logs/kafka

# install kafka
RUN curl -O http://apache.tt.co.kr/kafka/kafka-3.5.6/apache-kafka-3.5.6-bin.tar.gz  
RUN tar -zxvf /apps/install/apache-kafka-3.5.6-bin.tar.gz
RUN mv /apps/install/apache-kafka-3.5.6-bin /apps/kafka/3.5.6

ADD config /apps/kafka/3.5.6/config

RUN sed -i -e 's/^kafka.log.dir=.$/kafka.log.dir=\/logs\/kafka/' /apps/kafka/3.5.6/conf/log4j.properties

USER root

RUN chown -R app.app /apps/kafka

USER app

WORKDIR /apps/kafka

# 컨테이너 실행시 실행될 명령
CMD /bin/bash
```

`build/create docker image`  
$ docker build -t mobon/kafka.3:latest -f /apps/docker/images/kafka.3/Dockerfile .  
>$ docker build --no-cache -t mobon/kafka.3:latest -f /apps/docker/images/kafka.3/Dockerfile .  
>$ docker build -t mobon/kafka.3.5.6 -t mobon/kafka.3:latest -f /apps/docker/images/kafka.3/Dockerfile .  
>$ docker image tag mobon/kafka.3.5.6 mobon/kafka.3:latest

>`create docker service container`  
>$ docker run --net mobon.subnet --ip 192.168.104.51  --ulimit memlock=-1 --name mobon.kafka.01 -d -p 7214:7214 -it mobon/kafka.3:latest

>$ docker exec -it mobon.kafka.01 /bin/bash

>`push image to docker private registry`  
>$ docker tag mobon/kafka.3:latest docker-registry.mobon.net:5000/mobon/kafka.3:latest  
>$ docker push docker-registry.mobon.net:5000/mobon/kafka.3:latest


<details>
<summary>build image > create container</summary>
<div markdown="1">

#### create containers base mobon/centos.7.base:1.1
$ docker run --net mobon.subnet --ip 192.168.104.51  --ulimit memlock=-1 --name kafka.3 -d -it docker-registry.mobon.net:5000/mobon/java.app.env:latest

$ docker exec -it kafka.5 /bin/bash

$$ mkdir -p /apps/kafka /data/kafka /logs/kafka

$$ cd /apps/install

$$ curl -O http://apache.tt.co.kr/kafka/kafka-3.5.6/apache-kafka-3.5.6.tar.gz  
$$ tar -zxvf /apps/install/apache-kafka-3.5.6.tar.gz
$$ mv /apps/install/apache-kafka-3.5.6 /apps/kafka/3.5.6

$$ vi /apps/kafka/3.5.6/conf/zoo.cfg
```
# The number of milliseconds of each tick  
tickTime=2000  
# The number of ticks that the initial synchronization phase can take  
initLimit=10  
# The number of ticks that can pass between sending a request and getting an acknowledgement  
syncLimit=5  
# the directory where the snapshot is stored. do not use /tmp for storage, /tmp here is just example sakes.  
dataDir=/data/kafka
# the port at which the clients will connect  
clientPort=7214  
# the port address at which the clients will connect
clientPortAddress=0.0.0.0
# the maximum number of client connections. increase this if you need to handle more clients  
#maxClientCnxns=60

# The number of snapshots to retain in dataDir  
#autopurge.snapRetainCount=3  
# Purge task interval in hours. Set to "0" to disable auto purge feature  
#autopurge.purgeInterval=1

# Set ensemble(cluster) as  
# server.{server id[1-255]} = {hostname}:{port to connect followers to the leader}:{port for leader election}  
server.1=192.168.104.51:7224:7234  
server.2=192.168.104.52:7224:7234  
server.3=192.168.104.53:7224:7234
```

$$ sed -i -e 's/^kafka.log.dir=.$/kafka.log.dir=\/logs\/kafka/' /apps/kafka/3.5.6/conf/log4j.properties

$$ echo 1 > /data/kafka/myid  
$$ echo 2 > /data/kafka/myid  
$$ echo 3 > /data/kafka/myid

$$ vi /apps/kafka/kafka
```
#!/bin/sh  
# kafka kafka service shell  
# chkconfig: 2345 90 90  
# description: kafka  
# processname: zkServer  
# config: $kafka_CONF  
# pidfile:

kafka_HOME='/apps/kafka/3.5.6'

kafka_EXEC=$kafka_HOME/bin/zkServer.sh

export JVMFLAGS="-Xms1024m -Xmx1024m"  
export ZOO_LOG_DIR=/logs/kafka  
export ZOO_LOG4J_PROP="INFO,CONSOLE"

case "$1" in  
  start)
    echo -en "Starting kafka Server...\n"  
    $kafka_EXEC start  
    echo -e "\n"  
    ;;
  
  stop)
    echo -en "Shutting Down kafka Server...\n"  
    $kafka_EXEC stop  
    echo -e "\n"  
    ;;
  
  status)
    echo -en "Shutting Down kafka Server...\n"  
    $kafka_EXEC status  
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
$ docker commit -a "sjlee@ruaniz.com" -m "create image from kafka.3 container" kafka.3 mobon/kafka.3:latest

>`push image to docker private registry`  
>$ docker tag mobon/kafka.3:latest docker-registry.mobon.net:5000/mobon/kafka.35:latest  
>$ docker push docker-registry.mobon.net:5000/mobon/kafka.3:latest

</div>
</details>
