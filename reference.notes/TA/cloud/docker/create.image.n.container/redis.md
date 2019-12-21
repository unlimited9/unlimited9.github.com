# redis

## build image

#### 01. build image mobon/redis.5.0.7(mobon/redis.5:latest) based mobon/centos.7.base:latest

`create redis config files : redis.conf, sentinel.conf`  
$ vi /apps/docker/images/redis.5/config/redis.conf
```
#bind 127.0.0.1을 주석 처리  
#bind 127.0.0.1
#원격 서버에서 접속 가능하게 설정하려면 no로 지정  
protected-mode no  
#포트 지정  
port 2816
tcp-backlog 511

timeout 0
tcp-keepalive 60

daemonize yes
supervised no
#위치 지정  
pidfile /tmp/redis.pid  
loglevel notice
logfile "/logs/redis/redis.log"  

#비밀번호 지정  
requirepass P@ssw0rd  
masterauth P@ssw0rd  

databases 16
always-show-logo no

#save 900 1
#save 300 10
#save 60 10000

stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir /data/redis/db

replica-serve-stale-data yes
replica-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5

repl-disable-tcp-nodelay no

replica-priority 100

lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
replica-lazy-flush no

#변경 시 해당 명령을 저장해 두었다가, 복구 시 저장된 명령을 순서대로 실행하는 옵션  
appendonly yes  

appendfilename "appendonly.aof"

appendfsync everysec

no-appendfsync-on-rewrite no

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
aof-use-rdb-preamble yes

lua-time-limit 5000

slowlog-log-slower-than 10000
slowlog-max-len 128

latency-monitor-threshold 0

notify-keyspace-events ""

hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
stream-node-max-bytes 4096
stream-node-max-entries 100
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit replica 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60

hz 10
dynamic-hz yes
aof-rewrite-incremental-fsync yes
rdb-save-incremental-fsync yes

#yes로 설정 시 클러스터 모드, no로 설정 시 standalone 모드로 시작  
cluster-enabled yes  

#클러스터의 상태를 기록하는 바이너리 파일 경로  
cluster-config-file nodes.conf  

#레디스 노드가 다운되었는지 판단하는 시간 단위는 millisecond  
cluster-node-timeout 3000  

#다운 시도 횟수  
cluster-replica-validity-factor 2  

#cluster-require-full-coverage no
#cluster-migration-barrier 1
```

$ vi /apps/docker/images/redis.5/config/sentinel.conf
```
port 2826
daemonize yes
pidfile /tmp/redis_sentinel.pid  
logfile /logs/redis/redis_sentinel.log  

dir /data/redis/sentinel

sentinel monitor esmaster 127.0.0.1 2816 2  
sentinel auth-pass esmaster dlsfkdlvmf1!  
sentinel down-after-milliseconds esmaster 5000  
sentinel parallel-syncs esmaster 1  
sentinel failover-timeout esmaster 60000

sentinel deny-scripts-reconfig yes
```

`create dockerize file`  
$ vi /apps/docker/images/redis.5/Dockerfile
```
FROM docker-registry.mobon.net:5000/mobon/centos.7.base:latest
# FROM mobon/centos.7.base:latest
#
USER root

RUN yum install -y gcc-c++ make

USER app

# install and setup application
WORKDIR /apps/install

RUN mkdir -p /apps/redis /data/redis/db /logs/redis

# install redis
RUN curl -O http://download.redis.io/releases/redis-5.0.7.tar.gz
RUN tar -zxvf /apps/install/redis-5.0.7.tar.gz
#RUN mv /apps/install/redis-5.0.7 /apps/redis/5.0.7

WORKDIR /apps/install/redis-5.0.7
RUN make
RUN make PREFIX=/apps/redis/5.0.7 install

ADD config /apps/redis/5.0.7/config

USER root

RUN chown -R app.app /apps/redis

USER app

WORKDIR /apps/redis

# 컨테이너 실행시 실행될 명령
CMD /bin/bash
```

`build/create docker image`  
$ docker build -t mobon/redis.5:latest -f /apps/docker/images/redis.5/Dockerfile .  
>$ docker build --no-cache -t mobon/redis.5:latest -f /apps/docker/images/redis.5/Dockerfile .  
>$ docker build -t mobon/redis.5.0.7 -t mobon/redis.5:latest -f /apps/docker/images/redis.5/Dockerfile .  
>$ docker image tag mobon/redis.5.0.7 mobon/redis.5:latest

>`create docker service container`  
>$ docker run --net mobon.subnet --ip 192.168.104.51  --ulimit memlock=-1 --name mobon.redis.01 -d -p 6530:6530 -it mobon/redis.5:latest

>$ docker exec -it mobon.redis.01 /bin/bash

>`push image to docker private registry`  
>$ docker tag mobon/redis.5:latest docker-registry.mobon.net:5000/mobon/redis.5:latest  
>$ docker push docker-registry.mobon.net:5000/mobon/redis.5:latest


<details>
<summary>build image > create container</summary>
<div markdown="1">

#### create containers base mobon/centos.7.base:1.1
$ docker run --net mobon.subnet --ip 192.168.104.51  --ulimit memlock=-1 --name redis.5 -d -it docker-registry.mobon.net:5000/mobon/centos.7.base:latest

$ docker exec -it redis.5 /bin/bash

$$ mkdir -p /apps/redis /data/redis /logs/redis

$$ cd /apps/install

$$ curl -O http://download.redis.io/releases/redis-5.0.7.tar.gz  
$$ tar -xvf redis-5.0.7.tar.gz

$$ cd redis-5.0.7

> $$ sudo yum install -y gcc-c++ make  
$$ make  
$$ make test  
$$ make PREFIX=/apps/redis/5.0.7 install  

$$ mkdir -p /apps/redis/instances/esdb01/config /data/redis/ /logs/redis
$$ cp /apps/install/redis-5.0.7/redis.conf /apps/redis/instances/esdb01/config

$$ sed -i -e 's/^always-show-logo yes$/always-show-logo no/' /apps/redis/instances/esdb01/config/redis.conf
$$ sed -i -e 's/^supervised no$/supervised systemd/' /apps/redis/instances/esdb01/config/redis.conf
$$ sed -i -e 's/^dir \.\/$/dir \/data\/redis\/esdb01/' /apps/redis/instances/esdb01/config/redis.conf

$$ sed -i -e 's/^save 900 1$/# save 900 1/' /apps/redis/instances/esdb01/config/redis.conf  
$$ sed -i -e 's/^save 300 10$/# save 300 10/' /apps/redis/instances/esdb01/config/redis.conf  
$$ sed -i -e 's/^save 60 10000$/# save 60 10000/' /apps/redis/instances/esdb01/config/redis.conf

$$ vi /apps/redis/instances/esdb01/config/redis.conf
```
...  
#원격 서버에서 접속 가능하게 설정하려면 no로 지정  
protected-mode no  

tcp-keepalive 60  
loglevel notice  
logfile "/logs/redis/redis_esdb01.log"  
daemonize yes  

#포트 지정  
port 2816  

#bind 127.0.0.1을 주석 처리  
# bind 127.0.0.1  

#비밀번호 지정  
requirepass <password>  
masterauth <password>  

#yes로 설정 시 클러스터 모드, no로 설정 시 standalone 모드로 시작  
cluster-enabled yes  

#클러스터의 상태를 기록하는 바이너리 파일 경로  
cluster-config-file nodes_esdb01.conf  

#레디스 노드가 다운되었는지 판단하는 시간 단위는 millisecond  
cluster-node-timeout 3000  

#다운 시도 횟수  
cluster-replica-validity-factor 2  

#위치 지정  
pidfile /tmp/redis_esdb01.pid  
dbfilename dump-esdb01.rdb

#변경 시 해당 명령을 저장해 두었다가, 복구 시 저장된 명령을 순서대로 실행하는 옵션  
appendonly yes  
...
```

$$ cp /apps/install/redis-5.0.7/sentinel.conf /apps/redis/instances/essentinel01.config
```
...  
port 2826  
dir /data/redis/essentinel01

sentinel monitor esmaster 127.0.0.1 2816 2  
sentinel auth-pass esmaster dlsfkdlvmf1!  
sentinel down-after-milliseconds esmaster 5000  
sentinel parallel-syncs esmaster 1  
sentinel failover-timeout esmaster 60000

daemonize yes  
pidfile /tmp/redis_essentinel01.pid  
logfile /logs/redis/redis_essentinel01.log  
...
```

$$ vi /apps/redis/redis
```
#!/bin/sh  
# redis redis service shell  
# chkconfig: 2345 90 90  
# description: redis  
# processname: redis-server  
# config: $REDIS_CONF  
# pidfile:

REDIS_HOME='/apps/redis/5.0.7'  
REDIS_BASE="/apps/redis/instances"

REDIS_EXEC=$REDIS_HOME/bin/redis-server  
REDIS_CLIEXEC=$REDIS_HOME/bin/redis-cli

INSTANCE_NAMES=(${2-"esdb01" "esdb02" "esdb03"})  
declare -A INSTANCE_PORTS=(["esdb01"]="2816" ["esdb02"]="2817" ["esdb03"]="2818")

serviceCtrl() {  
    # find $MONGO_BASE -mindepth 1 -maxdepth 1 -type d | sort -t '\0' -n | while read instance  
    for instance in "${INSTANCE_NAMES[@]}"  
    do
        PIDFILE=/tmp/redis_${instance}.pid  
        case "$1" in
            start)
                if [ -f $PIDFILE ]  
                then
                    echo "$PIDFILE exists, process is already running or crashed"
                else
                    echo -en "Starting Redis $instance instance...\n"  
                    $REDIS_EXEC $REDIS_BASE/$instance/config/redis.conf
                fi  
                ;;
            stop)
                if [ ! -f $PIDFILE ]  
                then
                    echo "$PIDFILE does not exist, process is not running"
                else
                    PID=$(cat $PIDFILE)  
                    echo -en "Stopping Redis $instance instance...\n"  
                    $REDIS_CLIEXEC -p ${INSTANCE_PORTS[$instance]} -a <password> shutdown  
                    while [ -x /proc/${PID} ]  
                    do
                        echo "Waiting for Redis to shutdown ..."  
                        sleep 1
                    done  
                    echo "Redis $instance stopped"
                fi  
                ;;
        esac
    done
}  
case "$1" in  
    start)
        echo -en "Starting Redis Server...\n"  
        serviceCtrl start  
        echo -e "\n"  
        ;;
        
    stop)
        echo -en "Shutting Down Redis Server...\n"  
        serviceCtrl stop  
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
$$ chmod 755 /apps/redis/redis  

$$ vi /apps/redis/sentinel
```
#!/bin/sh  
# sentinel sentinel service shell  
# chkconfig: 2345 90 90  
# description: sentinel  
# processname: redis-sentinel  
# config: $SENTINEL_CONF  
# pidfile:

REDIS_HOME='/apps/redis/5.0.5'  
REDIS_BASE="/apps/redis/instances"

REDIS_EXEC=$REDIS_HOME/bin/redis-sentinel  
REDIS_CLIEXEC=$REDIS_HOME/bin/redis-cli

INSTANCE_NAMES=(${2-"essentinel01" "essentinel02" "essentinel03"})  
declare -A INSTANCE_PORTS=(["essentinel01"]="2826" ["essentinel02"]="2827" ["essentinel03"]="2828")

serviceCtrl() {  
    # find $MONGO_BASE -mindepth 1 -maxdepth 1 -type d | sort -t '\0' -n | while read instance  
    for instance in "${INSTANCE_NAMES[@]}"  
    do
        PIDFILE=/tmp/redis_${instance}.pid
        case "$1" in
            start)
                if [ -f $PIDFILE ]
                then
                    echo "$PIDFILE exists, process is already running or crashed"
                else
                    echo -en "Starting Redis Sentinel $instance instance...\n"
                    $REDIS_EXEC $REDIS_BASE/$instance/config/sentinel.conf
                fi
                ;;
            stop)
                if [ ! -f $PIDFILE ]
                then
                    echo "$PIDFILE does not exist, process is not running"
                else
                    PID=$(cat $PIDFILE)
                    echo -en "Stopping Redis Sentinel $instance instance...\n"
                    $REDIS_CLIEXEC -p ${INSTANCE_PORTS[$instance]} -a <password> shutdown
                    while [ -x /proc/${PID} ]
                    do
                        echo "Waiting for Redis Sentinel to shutdown ..."
                        sleep 1
                    done
                    echo "Redis Sentinel $instance stopped"
                fi
                ;;
        esac
    done
}  
case "$1" in  
    start)
        echo -en "Starting Redis Sentinel Server...\n"
        serviceCtrl start
        echo -e "\n"
        ;;
        
    stop)
        echo -en "Shutting Down Redis Sentinel Server...\n"
        serviceCtrl stop
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
$$ chmod 755 /apps/redis/sentinel  

$$ /apps/redis/redis start

#### create image from base.container - mobon/redis.5:latest
$ docker commit -a "sjlee@ruaniz.com" -m "create image from redis.5 container" redis.5 mobon/redis.5:latest

>`push image to docker private registry`  
>$ docker tag mobon/redis.5:latest docker-registry.mobon.net:5000/mobon/redis.5:latest  
>$ docker push docker-registry.mobon.net:5000/mobon/redis.5:latest

</div>
</details>
