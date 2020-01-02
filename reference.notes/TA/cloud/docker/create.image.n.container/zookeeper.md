# zookeeper

## build image

#### 01. build image mobon/zookeeper.3.5.6(mobon/zookeeper.3:latest) based mobon/java.app.env:latest

`create zookeeper config files : zoo.cfg`  
$ vi /apps/docker/images/zookeeper.3/config/zoo.cfg
```
# The number of milliseconds of each tick  
tickTime=2000  
# The number of ticks that the initial synchronization phase can take  
initLimit=10  
# The number of ticks that can pass between sending a request and getting an acknowledgement  
syncLimit=5  
# the directory where the snapshot is stored. do not use /tmp for storage, /tmp here is just example sakes.  
dataDir=/data/zookeeper
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
server.1=mobon-data-zookeeper-0:7224:7234  
server.2=mobon-data-zookeeper-1:7224:7234  
server.3=mobon-data-zookeeper-2:7224:7234
```

`create dockerize file`  
$ vi /apps/docker/images/zookeeper.3/Dockerfile
```
FROM docker-registry.mobon.net:5000/mobon/java.app.env:latest
# FROM mobon/java.app.ext:latest
#
USER root

RUN yum install -y gcc-c++ make

USER app

# install and setup application
WORKDIR /apps/install

RUN mkdir -p /apps/zookeeper /data/zookeeper/db /logs/zookeeper

# install zookeeper
RUN curl -O http://apache.tt.co.kr/zookeeper/zookeeper-3.5.6/apache-zookeeper-3.5.6.tar.gz  
RUN tar -zxvf /apps/install/apache-zookeeper-3.5.6.tar.gz
RUN mv /apps/install/apache-zookeeper-3.5.6 /apps/zookeeper/3.5.6

ADD config /apps/zookeeper/3.5.6/config

RUN sed -i -e 's/^zookeeper.log.dir=.$/zookeeper.log.dir=\/logs\/zookeeper/' /apps/zookeeper/3.5.6/conf/log4j.properties

WORKDIR /apps/zookeeper

# 컨테이너 실행시 실행될 명령
CMD /bin/bash
```

`build/create docker image`  
$ docker build -t mobon/zookeeper.3:latest -f /apps/docker/images/zookeeper.3/Dockerfile .  
>$ docker build --no-cache -t mobon/zookeeper.3:latest -f /apps/docker/images/zookeeper.3/Dockerfile .  
>$ docker build -t mobon/zookeeper.3.5.6 -t mobon/zookeeper.3:latest -f /apps/docker/images/zookeeper.3/Dockerfile .  
>$ docker image tag mobon/zookeeper.3.5.6 mobon/zookeeper.3:latest

>`create docker service container`  
>$ docker run --net mobon.subnet --ip 192.168.104.51  --ulimit memlock=-1 --name mobon.zookeeper.01 -d -p 7214:7214 -it mobon/zookeeper.3:latest

>$ docker exec -it mobon.zookeeper.01 /bin/bash

>`push image to docker private registry`  
>$ docker tag mobon/zookeeper.3:latest docker-registry.mobon.net:5000/mobon/zookeeper.3:latest  
>$ docker push docker-registry.mobon.net:5000/mobon/zookeeper.3:latest


<details>
<summary>build image > create container</summary>
<div markdown="1">

#### create containers base mobon/centos.7.base:1.1
$ docker run --net mobon.subnet --ip 192.168.104.51  --ulimit memlock=-1 --name zookeeper.3 -d -it docker-registry.mobon.net:5000/mobon/java.app.env:latest

$ docker exec -it zookeeper.5 /bin/bash

$$ mkdir -p /apps/zookeeper /data/zookeeper /logs/zookeeper

$$ cd /apps/install

$$ curl -O http://apache.tt.co.kr/zookeeper/zookeeper-3.5.6/apache-zookeeper-3.5.6.tar.gz  
$$ tar -zxvf /apps/install/apache-zookeeper-3.5.6.tar.gz
$$ mv /apps/install/apache-zookeeper-3.5.6 /apps/zookeeper/3.5.6

$$ vi /apps/zookeeper/3.5.6/conf/zoo.cfg
```
# The number of milliseconds of each tick  
tickTime=2000  
# The number of ticks that the initial synchronization phase can take  
initLimit=10  
# The number of ticks that can pass between sending a request and getting an acknowledgement  
syncLimit=5  
# the directory where the snapshot is stored. do not use /tmp for storage, /tmp here is just example sakes.  
dataDir=/data/zookeeper
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

$$ sed -i -e 's/^zookeeper.log.dir=.$/zookeeper.log.dir=\/logs\/zookeeper/' /apps/zookeeper/3.5.6/conf/log4j.properties

$$ echo 1 > /data/zookeeper/myid  
$$ echo 2 > /data/zookeeper/myid  
$$ echo 3 > /data/zookeeper/myid

$$ vi /apps/zookeeper/zookeeper
```
#!/bin/sh  
# zookeeper zookeeper service shell  
# chkconfig: 2345 90 90  
# description: zookeeper  
# processname: zkServer  
# config: $ZOOKEEPER_CONF  
# pidfile:

ZOOKEEPER_HOME='/apps/zookeeper/3.5.6'

ZOOKEEPER_EXEC=$ZOOKEEPER_HOME/bin/zkServer.sh

export JVMFLAGS="-Xms1024m -Xmx1024m"  
export ZOO_LOG_DIR=/logs/zookeeper  
export ZOO_LOG4J_PROP="INFO,CONSOLE"

case "$1" in  
  start)
    echo -en "Starting Zookeeper Server...\n"  
    $ZOOKEEPER_EXEC start  
    echo -e "\n"  
    ;;
  
  stop)
    echo -en "Shutting Down Zookeeper Server...\n"  
    $ZOOKEEPER_EXEC stop  
    echo -e "\n"  
    ;;
  
  status)
    echo -en "Shutting Down Zookeeper Server...\n"  
    $ZOOKEEPER_EXEC status  
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
$$ chmod 755 /apps/zookeeper/zookeeper  

$$ /apps/zookeeper/zookeeper start

#### create image from base.container - mobon/zookeeper.5:latest
$ docker commit -a "sjlee@ruaniz.com" -m "create image from zookeeper.3 container" zookeeper.3 mobon/zookeeper.3:latest

>`push image to docker private registry`  
>$ docker tag mobon/zookeeper.3:latest docker-registry.mobon.net:5000/mobon/zookeeper.35:latest  
>$ docker push docker-registry.mobon.net:5000/mobon/zookeeper.3:latest

</div>
</details>
