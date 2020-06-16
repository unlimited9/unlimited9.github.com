# 10.install.n setup : elasticsearch

>Created 목요일 30 11월 2017
Server.setting - application

# Installation/Setup/Configuration

## 1. Pre-installation setup

### A. creating required operating system group and user

#### create the os application group(typically, app)
$ groupadd -g 3000 app

> 어플리케이션 단위 관리 주체가 다르지 않고 권한을 나눌 필요가 없어 통합 계정을 생성하여 관리apps
> ~~어플리케이션 단위 관리 주체가 다르다면 어플리케이션별 계정을 생성하여 생성하여 관리~~

#### create the software unified account (typically, app)
$ useradd -d /apps -g 3000 -m -u 3000 -s /bin/bash rerisb
$ passwd app

#### create the elasticsearch software owner (typically, elasticsearch) 
~~$ useradd -d /apps/r -g 3000 -m -u 3030 -s /bin/bash elasticsearch~~
> ~~$ useradd -d /apps/elasticsearch -g 3000 -m -u 20 -s /bin/bash elasticsearch  -s /sbin/nologin~~
> 
~~$ passwd elasticsearch~~
>#### usermod (append group) 
>$ usermod --append --groups app elasticsearch
>#### service status 
>$ sudo /sbin/service --status-all

### B. install dependency packages

### C. system kernel setting

#### elasticsearch accout limits setting  
$ ulimit -a

>$ vi /etc/security/limits.conf
>```
>...
>#elasticsearch
>app             -       nofile          65536
>app             -       nproc           4096
>app             -       memlock         unlimited
>...
>```

$ vi /etc/security/limits.d/20-nproc.conf 
```
# Default limit for number of user's processes to prevent
# accidental fork bombs.
# See rhbz #432903 for reasoning.

app        soft    nproc     4096
root       soft    nproc     unlimited
```

$ vi /etc/security/limits.d/30-nofile.conf
```
app        -    nofile     65536
```

$ /etc/security/limits.d/40-memlock.conf
```
app        -    memlock     unlimited
```

#### system setting  
>docker container system kernel 설정은 host 설정이 같이 적용된다.

$ cat /proc/sys/vm/max_map_count  

$ sysctl -w vm.max_map_count=262144  
$ vi /etc/sysctl.d/99-sysctl.conf
>$ vi /etc/sysctl.conf  
```
...
vm.overcommit_memory = 1
...
```

>#### > detail
>#### elasticsearch message about resource 
>`> max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]`
$ ulimit -n  
1024  
$ ulimit -Hn  
4096  
$ ulimit -Sn  
1024
$ vi /etc/security/limits.conf  
>```
>...
>elasticsearch - nofile 65535
>...
>```
>
>`> max number of threads [1024] for user [elasticsearch] is too low, increase to at least [4096]`
>$ ulimit -u  
1024  
~~$ ulimit -u 4096~~  
$ vi /etc/security/limits.conf  
elasticsearch - nproc 4096  
>
>`> memory locking requested for elasticsearch process but memory is not locked`  
$ ulimit -l  
64  
~~$ ulimit -l unlimited~~  
$ vi /etc/security/limits.conf  
>```
>...
>elasticsearch - memlock unlimited
>...
>```
>
>`> max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]`
$ cat /proc/sys/vm/max_map_count  
65530  
$ sysctl -w vm.max_map_count=262144  
$ vi /etc/sysctl.conf  
>```
>...
>vm.overcommit_memory = 262144
>...
>```

### D. firewall configuration

$ vi /etc/sysconfig/iptables
```
...  
## elasticsearch
# http  
-A INPUT -m state --state NEW -m tcp -p tcp --dport 6530 -j ACCEPT  
# https  
-A INPUT -m state --state NEW -m tcp -p tcp --dport 6540 -j ACCEPT
...
```

#### restart iptalbes service 
$ service iptables restart  

$ iptables -nL  

## 2. installation setup : app

### A. change account

$ su - app

### B. creating application directory

#### make directory
$ mkdir -p /apps/elasticsearch /data/elasticsearch /logs/elasticsearch  

### C. download

#### Elastic(https://www.elastic.co/downloads)
$ cd /apps/install  
$ curl -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.1.1-linux-x86_64.tar.gz  
~~$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.1.1-linux-x86_64.tar.gz -P /apps/install~~  

### D. install

#### decompress tarball
$ tar -zxvf /apps/install/elasticsearch-7.1.1-linux-x86_64.tar.gz  

#### copy application directory
$ cp -R /apps/install/elasticsearch-7.1.1 /apps/elasticsearch/7.1.1  

### E. configure
$ cd /apps/elasticsearch/7.1.1/config  

$ cp elasticsearch.yml elasticsearch.yml.default  

> 운영서버는 4기로 구성(예정)  
> essrch01 : 192.168.104.51(master-eligible/ingest Node)  
> essrch02 : 192.168.104.52(master-eligible/ingest Node)  
> essrch03 : 192.168.104.53(data Node)  
> essrch04 : 192.168.104.54(data Node)  
> jvm.options : -Xms16g -Xmx16g  

$ vi /apps/elasticsearch/7.1.1/config/elasticsearch.yml  
```
# ---------------------------------- Cluster -----------------------------------
cluster.name: essrch

# ------------------------------------ Node ------------------------------------
#essrch01, essrch02, essrch03, essrch04
node.name: essrch01

# master-eligible : essrch01, essrch02
# ingest : essrch01, essrch02
# data : essrch03, essrch04
# coordinating/lb(master-eligible/ingest/data false) : -

# coordinating/lb Node
node.master: false
node.data: false
node.ingest: false
cluster.remote.connect: false

# master-eligible Node  
node.master: true
node.data: false
node.ingest: false
cluster.remote.connect: false

# data Node  
node.master: false
node.data: true
node.ingest: false
cluster.remote.connect: false

# ingest Node  
node.master: false
node.data: false
node.ingest: true
cluster.remote.connect: false

# ----------------------------------- Memory -----------------------------------
bootstrap.memory_lock: true
bootstrap.system_call_filter : false

# -----------------------------apps----- Network -----------------------------------
#192.168.104.51, 192.168.104.52, 192.168.104.53, 192.168.104.54
network.host: 192.168.104.51
http.port: 6530
#http.enabled: false (deprecated)
transport.tcp.port: 6540

# --------------------------------- Discovery ----------------------------------
discovery.seed_hosts: ["192.168.104.51:6540", "192.168.104.52:6540", "192.168.104.53:6540", "192.168.104.54:6540"]
cluster.initial_master_nodes: ["essrch01", "essrch02"]
#discovery.zen.ping.unicast.hosts: ["192.168.104.51:6540", "192.168.104.52:6540", "192.168.104.53:6540", "192.168.104.54:6540"](deprecated)
#discovery.zen.minimum_master_nodes: 2(deprecated)

# --------------------------------- Paths ----------------------------------
path.data: /data/elasticsearch
path.logs: /logs/elasticsearch

thread_pool.search.size: 50
thread_pool.search.queue_size: 2000

# CORS settings.
http.cors.enabled: true
http.cors.allow-origin: "*"
#http.cors.allow-origin: "http://www.dreamsearch.or.kr"
http.cors.allow-methods : OPTIONS, HEAD, GET, POST
http.cors.allow-headers : X-Requested-With,X-Auth-Token,Content-Type, Content-Length
http.cors.allow-credentials: true
```

$ cp jvm.options jvm.options.default  

$ vi /apps/elasticsearch/7.1.1/config/jvm.options  
```
...
-Xms16g
-Xmx16g
...
```

> 개발서버는 2기로 구성  
> essrch01 : 192.168.104.51(master-eligible/ingest Node)  
> essrch02 : 192.168.104.52(data Node)  
> jvm.options는 default로 설정(-Xms1g -Xmx1g)  

### F. excute

#### start process
$ /apps/elasticsearch/7.1.1/bin/elasticsearch -d  

### check process and port
$ ps -ef | grep elasticsearch  
...  
3350 4714 1 7 17:50 pts/3 00:00:18 /apps/jdk/1.8.0_152/bin/java -Xms1g -Xmx1g -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -XX:+AlwaysPreTouch -server -Xss1m -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djna.nosys=true -XX:-OmitStackTraceInFastThrow -Dio.netty.noUnsafe=true -Dio.netty.noKeySetOptimization=true -Dio.netty.recycler.maxCapacityPerThread=0 -Dlog4j.shutdownHookEnabled=false -Dlog4j2.disable.jmx=true -XX:+HeapDumpOnOutOfMemoryError -Des.path.home=/apps/elasticsearch/6.1.1 -Des.path.conf=/apps/elasticsearch/6.1.1/config -cp /apps/elasticsearch/6.1.1/lib/* org.elasticsearch.bootstrap.Elasticsearch -d  
...  

$ netstat -na | grep LISTEN  
...  
tcp 0 0 :::6530 :::* LISTEN  
tcp 0 0 :::6540 :::* LISTEN  
...  

#### stop process
$ ps -ef | grep elasticsearch  
$ kill -9 <pid>  

## 3. post-installation setup

#### A. create shell script

#### elasticsearch
$ vi /apps/elasticsearch/elasticsearch  
```
#!/bin/sh
# elasticsearch service control
# chkconfig: 2345 90 90
# description: elasticsearch
# processname: elasticsearch
# config: $ES_CONFIG_FILE
# pidfile:

UNAME=`id -u -n`

# the user you used to run the elastic search
ES_USER=app
ES_HOME=/apps/elasticsearch/7.1.1

ES_NAME=elasticsearch
ES_PID_FILE=$ES_HOME/$ES_NAME.pid

ES_EXEC=$ES_HOME/bin/elasticsearch
ES_EXEC_OPTS="-d -p $ES_PID_FILE"

case "$1" in
        start)
                echo -en "Starting Elastic Search Server..."
                if [ e$UNAME = "eroot" ]
                then
                        /bin/su -p -s /bin/sh $ES_USER -c "$ES_EXEC $ES_EXEC_OPTS"
                else
                        $ES_EXEC $ES_EXEC_OPTS
                fi

                if [ $? == 0 ]
                then
                        echo "started."
                else
                        echo "failed."
                fi
                echo -e "\n"
                ;;
        stop)
                echo -en "Shutting Down Elastic Search Server..."
                kill `cat $ES_PID_FILE`
                #rm $ES_PID_FILE

                if [ $? == 0 ]
                then
                        echo "stopped."
                else
                        echo "failed."
                fi
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

#### change mode  
$ chmod 755 /apps/elasticsearch/elasticsearch

#### B. add service
$ cp /apps/elasticsearch/elasticsearch /etc/rc.d/init.d/elasticsearch  
$ chkconfig --add elasticsearch

#### C. configure SELinux(optional)
$ vi /etc/selinux/config  
SELINUX=disabled

### 4. execution(starting and stopping daemon services)

#### A. start service
$ service elasticsearch start

#### B. restart service
$ service elasticsearch restart

#### C. stop service
$ service elasticsearch stop
