# elasticsearch

## build image

#### 01. build image mobon/elasticsearch.7:1.1(mobon/elasticsearch.7:latest) based mobon/centos.7.base:latest

`create elasticsearch config files : elasticsearch.yml, log4j2.properties`  
$ vi /apps/docker/images/elasticsearch.7/config/elasticsearch.yml
```
# ---------------------------------- Cluster -----------------------------------
cluster.name: ${CLUSTER_NAME}
cluster.remote.connect: false

# ------------------------------------ Node ------------------------------------
node.name: ${NODE_NAME:false}
node.master: ${NODE_MASTER:false}
node.data: ${NODE_DATA:false}
node.ingest: ${NODE_INGEST:false}

# coordinating/lb Node - node.master: false, node.data: false, node.ingest: false
# master-eligible Node - node.master: true, node.data: false, node.ingest: false
# data Node  - node.master: false, node.data: true, node.ingest: false
# ingest Node  - node.master: false, node.data: false, node.ingest: true

#node.attr.rack: rack_01

#processors: ${PROCESSORS:1}

# ----------------------------------- Memory -----------------------------------
# Kubernetes requires swap is turned off, so memory lock is redundant. set false
bootstrap.memory_lock: ${MEMORY_LOCK:true}
bootstrap.system_call_filter : false

# ---------------------------------- Network -----------------------------------
network.host: ${NETWORK_HOST}
#http.enabled: false (deprecated)
http.port: 6530
transport.tcp.port: 6540
#http.port: 9200(default)
#transport.tcp.port: 9300(default)

# --------------------------------- Discovery ----------------------------------
discovery.seed_hosts: ${DISCOVERY_SERVICE}
cluster.initial_master_nodes: ${MASTER_NODES}
#discovery.zen.minimum_master_nodes: 2(deprecated)

# --------------------------------- Paths ----------------------------------
path.data: ${PATH_DATA:/data/elasticsearch}
path.logs: ${PATH_LOGS:/logs/elasticsearch}

thread_pool.search.size: 50
thread_pool.search.queue_size: 2000

# CORS settings.
http.cors.enabled: ${HTTP_CORS_ENABLE:true}
http.cors.allow-origin: ${HTTP_CORS_ALLOW_ORIGIN:"*"}
http.cors.allow-methods : OPTIONS, HEAD, GET, POST
http.cors.allow-headers : X-Requested-With,X-Auth-Token,Content-Type, Content-Length
http.cors.allow-credentials: ${HTTP_CORS_ALLOW_CREDENTIALS:false}
```
$ vi /apps/docker/images/elasticsearch.7/config/log4j2.properties
```
status = error

appender.console.type = Console
appender.console.name = console
appender.console.layout.type = PatternLayout
appender.console.layout.pattern = [%d{ISO8601}][%-5p][%-25c{1.}] %marker%m%n

rootLogger.level = info
rootLogger.appenderRef.console.ref = console
```
$ chmod 640 /apps/docker/images/elasticsearch.7/config/*

`create elasticsearch service control sh`  
$ vi /apps/docker/images/elasticsearch.7/elasticsearch
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
ES_HOME=/apps/elasticsearch/7.5.0

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
$ chmod 755 /apps/docker/images/elasticsearch.7/elasticsearch  

`create dockerize file`  
$ vi /apps/docker/images/elasticsearch.7/Dockerfile
```
FROM docker-registry.mobon.net:5000/mobon/centos.7.base:latest
# FROM mobon/centos.7.base:latest

USER app

# install and setup application
WORKDIR /apps/install

RUN mkdir -p /apps/elasticsearch /data/elasticsearch /logs/elasticsearch

# install elasticsearch
RUN curl -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.5.0-linux-x86_64.tar.gz
RUN tar -zxvf /apps/install/elasticsearch-7.5.0-linux-x86_64.tar.gz
RUN mv /apps/install/elasticsearch-7.5.0 /apps/elasticsearch/7.5.0

# install plugin analysis-nori
RUN /apps/elasticsearch/7.5.0/bin/elasticsearch-plugin install analysis-nori
RUN touch /apps/elasticsearch/7.5.0/config/userdic_ko.txt

ADD config /apps/elasticsearch/7.5.0/config

ADD elasticsearch /apps/elasticsearch/elasticsearch

USER root

RUN chown -R app.app /apps/elasticsearch

# Set environment
#ENV DISCOVERY_SERVICE elasticsearch-discovery

# Kubernetes requires swap is turned off, so memory lock is redundant
#ENV MEMORY_LOCK false

USER app

# 컨테이너 실행시 실행될 명령
CMD /bin/bash
```

`build/create docker image`  
$ docker build -t mobon/elasticsearch.7:latest -f /apps/docker/images/elasticsearch.7/Dockerfile .  
>$ docker build --no-cache -t mobon/elasticsearch.7:latest -f /apps/docker/images/elasticsearch.7/Dockerfile .  
>$ docker build -t mobon/elasticsearch.7:1.1 -t mobon/elasticsearch.7:latest -f /apps/docker/images/elasticsearch.7/Dockerfile .  
>$ docker image tag mobon/elasticsearch.7:1.1 mobon/elasticsearch.7:latest

>`create docker service container`  
>$ docker run --net mobon.subnet --ip 192.168.104.51  --ulimit memlock=-1 --name mobon.elasticsearch.01 -d -p 6530:6530 -it mobon/elasticsearch.7:latest

>$ docker exec -it mobon.elasticsearch.01 /bin/bash

>`push image to docker private registry`  
>$ docker tag mobon/elasticsearch.7:latest docker-registry.mobon.net:5000/mobon/elasticsearch.7:latest  
>$ docker push docker-registry.mobon.net:5000/mobon/elasticsearch.7:latest


<details>
<summary>build image > create container</summary>
<div markdown="1">

#### create containers base mobon/centos.7.base:1.1
$ docker run --net mobon.subnet --ip 192.168.104.51  --ulimit memlock=-1 --name elasticsearch.7 -d -it docker-registry.mobon.net:5000/mobon/centos.7.base:latest

$ docker exec -it elasticsearch.7 /bin/bash

$$ mkdir -p /apps/elasticsearch  
$$ mkdir -p /data/elasticsearch  
$$ mkdir -p /logs/elasticsearch

$$ cd /apps/install

$$ curl -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.5.0-linux-x86_64.tar.gz  
$$ tar -zxvf /apps/install/elasticsearch-7.5.0-linux-x86_64.tar.gz  
$$ cp -R elasticsearch-7.5.0 /apps/elasticsearch/7.5.0

`Elasticsearch 기본 한글 형태소 분석기 노리 (nori) 설치`  
$$ /apps/elasticsearch/7.5.0/bin/elasticsearch-plugin install analysis-nori  
$$ touch /apps/elasticsearch/7.5.0/config/userdic_ko.txt

$$ cd /apps/elasticsearch/7.5.0/config  
$$ cp /apps/elasticsearch/7.5.0/config/elasticsearch.yml /apps/elasticsearch/7.5.0/config/elasticsearch.yml.default  
$$ vi /apps/elasticsearch/7.5.0/config/elasticsearch.yml
```
# ---------------------------------- Cluster -----------------------------------
cluster.name: ${CLUSTER_NAME}
cluster.remote.connect: false

# ------------------------------------ Node ------------------------------------
node.name: ${NODE_NAME:false}
node.master: ${NODE_MASTER:false}
node.data: ${NODE_DATA:false}
node.ingest: ${NODE_INGEST:false}

# coordinating/lb Node - node.master: false, node.data: false, node.ingest: false
# master-eligible Node - node.master: true, node.data: false, node.ingest: false
# data Node  - node.master: false, node.data: true, node.ingest: false
# ingest Node  - node.master: false, node.data: false, node.ingest: true

#node.attr.rack: rack_01

#processors: ${PROCESSORS:1}

# ----------------------------------- Memory -----------------------------------
# Kubernetes requires swap is turned off, so memory lock is redundant. set false
bootstrap.memory_lock: ${MEMORY_LOCK:true}
bootstrap.system_call_filter : false

# ---------------------------------- Network -----------------------------------
network.host: ${NETWORK_HOST}
#http.enabled: false (deprecated)
http.port: 6530
transport.tcp.port: 6540
#http.port: 9200(default)
#transport.tcp.port: 9300(default)

# --------------------------------- Discovery ----------------------------------
discovery.seed_hosts: ${DISCOVERY_SERVICE}
cluster.initial_master_nodes: ${MASTER_NODES}
#discovery.zen.minimum_master_nodes: 2(deprecated)

# --------------------------------- Paths ----------------------------------
path.data: ${PATH_DATA:/data/elasticsearch}
path.logs: ${PATH_LOGS:/logs/elasticsearch}

thread_pool.search.size: 50
thread_pool.search.queue_size: 2000

# CORS settings.
http.cors.enabled: ${HTTP_CORS_ENABLE:true}
http.cors.allow-origin: ${HTTP_CORS_ALLOW_ORIGIN:"*"}
http.cors.allow-methods : OPTIONS, HEAD, GET, POST
http.cors.allow-headers : X-Requested-With,X-Auth-Token,Content-Type, Content-Length
http.cors.allow-credentials: ${HTTP_CORS_ALLOW_CREDENTIALS:false}
```

$$ cp /apps/elasticsearch/7.5.0/config/jvm.options /apps/elasticsearch/7.5.0/config/jvm.options.default  
$$ vi /apps/elasticsearch/7.5.0/config/jvm.options
```
...
-Xms16g
-Xmx16g
...
```

$$ vi /apps/elasticsearch/elasticsearch
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
ES_HOME=/apps/elasticsearch/7.5.0

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
$$ chmod 755 /apps/elasticsearch/elasticsearch  

`develepment env`  
$$ export CLUSTER_NAME=es-cluster  
$$ export NODE_NAME=es-node  
$$ export NODE_MASTER=true  
$$ export NODE_DATA=true  
$$ export NODE_INGEST=true  
$$ export NETWORK_HOST=192.168.104.51  
$$ export DISCOVERY_SERVICE=["192.168.104.51:6540"]  
$$ export MASTER_NODES=["es-node"]  

$$ /apps/elasticsearch/elasticsearch start

#### create image from base.container - mobon/elasticsearch.7:latest
$ docker commit -a "sjlee@ruaniz.com" -m "create image from elasticsearch.7 container" elasticsearch.7 mobon/elasticsearch.7:latest

>`push image to docker private registry`  
>$ docker tag mobon/elasticsearch.7:latest docker-registry.mobon.net:5000/mobon/elasticsearch.7:latest  
>$ docker push docker-registry.mobon.net:5000/mobon/elasticsearch.7:latest

</div>
</details>

## kibana 

#### create containers base mobon/centos.7.base:1.1
$ docker run --net mobon.subnet --ip 192.168.104.53 --name elasticsearch.7.kibana -d -p 6636:6636 -it docker-registry.mobon.net:5000/mobon/centos.7.base:latest

$ docker exec -it elasticsearch.7.kibana /bin/bash

$$ mkdir -p /apps/kibana /data/kibana /logs/kibana

$$ cd /apps/install

$$ curl -O https://artifacts.elastic.co/downloads/kibana/kibana-7.5.0-linux-x86_64.tar.gz  
$$ tar -zxvf /apps/install/kibana-7.5.0-linux-x86_64.tar.gz  
$$ cp -R kibana-7.5.0-linux-x86_64 /apps/kibana/7.5.0

$$ cd /apps/kibana/7.5.0/config

$$ cp /apps/kibana/7.5.0/config/kibana.yml /apps/kibana/7.5.0/config/kibana.yml.default  
$$ vi /apps/kibana/7.5.0/config/kibana.yml
```
# Kibana is served by a back end server. This setting specifies the port to use.
server.port: 6636

# Specifies the address to which the Kibana server will bind. IP addresses and host names are both valid values.
# The default is 'localhost', which usually means remote machines will not be able to connect.
# To allow connections from remote users, set this parameter to a non-loopback address.
server.host: "192.168.104.53"

# The URLs of the Elasticsearch instances to use for all your queries.
elasticsearch.hosts: ["http://10.251.0.188:6530"]
```

$$ vi /apps/kibana/kibana  
```
#!/bin/sh
# kibana    kibana service shell
# chkconfig: 2345 90 90
# description: kibana
# processname: kibana
# config: $KIBANA_CONF
# pidfile:

UNAME=`id -u -n`

KIBANA_USER=app
KIBANA_HOME=/apps/kibana/7.5.0
KIBANA_EXEC=$KIBANA_HOME/bin/kibana

kibana_pid() {
	echo `ps aux | grep "$KIBANA_HOME" | grep -v grep | awk '{ print $2 }'`
}

_pid=$(kibana_pid)

case "$1" in
	start)
		if [ -n "$_pid" ]
		then
			echo "Kibana is already running (pid: $_pid)"
		else
			echo -en "Starting Kibana Server..."
	                if [ e$UNAME = "eroot" ]
	                then
	                        /bin/su -p -s /bin/sh $KIBANA_USER -c "$KIBANA_EXEC > /dev/null 2>&1&"
	                else
		                $KIBANA_EXEC > /dev/null 2>&1&
	                fi

	                if [ $? == 0 ]
        	        then
				echo "started."
	                else
        	                echo "failed."
			fi
		fi
		;;
	stop)
		if [ -n "$_pid" ]
		then
			echo -en  "Shutting Down Kibana Server..."
			kill $_pid

        	        if [ $? == 0 ]
	                then
	                        echo "stopped."
	                else
	                        echo "failed."
        	        fi
		else
			echo "Kibana is not running"
		fi
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
$$ chmod 755 /apps/kibana/kibana

$$ /apps/kibana/kibana start
