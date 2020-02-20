# setup
~~docker run -d -it --ulimit memlock=-1 --name mobon.elasticsearch.01 centos:7~~

> _Kibana를 같이 설치할 경우 외부에서 접근 가능하도록 host와 port mapping 설정을 한다._
> docker run --net mobon.subnet --ip 192.168.104.51  --ulimit memlock=-1 --name mobon.elasticsearch.01 -d -p 6530:6530 -p 6636:6636 -it mobon/apps.server:1.1

docker run --net mobon.subnet --ip 192.168.104.51  --ulimit memlock=-1 --name mobon.elasticsearch.01 -d -p 6530:6530 -it mobon/apps.server:1.1
> docker run --net mobon.subnet --ip 192.168.104.52  --ulimit memlock=-1 --name mobon.elasticsearch.02 -d -it mobon/apps.server:1.1

docker exec -it mobon.elasticsearch.01 /bin/bash

>mobon/apps.server:1.1에 이미 생성 및 처리 되어 있음. 
>ref. 30.architecture/30.technical/30.solution/z1.docker/10.install.n.setup_script
>
>> /bin/bash
>>docker exec -it mobon.elasticsearch.01 /bin/bash
>>
>>groupadd -g 3000 app
>>useradd -d /apps -g 3000 -m -u 3000 -s /bin/bash app
>>
>>mkdir -p /apps/install
>>mkdir -p /pgms
>>mkdir -p /data
>>mkdir -p /logs
>>
>>chown -R app:app /apps
>>chown -R app:app /pgms
>>chown -R app:app /data
>>chown -R app:app /logs

# install : elasticsearch
su - app

mkdir -p /apps/elasticsearch
mkdir -p /data/elasticsearch
mkdir -p /logs/elasticsearch

cd /apps/install
curl -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.1.1-linux-x86_64.tar.gz

tar -zxvf /apps/install/elasticsearch-7.1.1-linux-x86_64.tar.gz

cp -R elasticsearch-7.1.1 /apps/elasticsearch/7.1.1

cd /apps/elasticsearch/7.1.1/config
cp elasticsearch.yml elasticsearch.yml.default
vi /apps/elasticsearch/7.1.1/config/elasticsearch.yml
>dmoespd01
```
# ---------------------------------- Cluster -----------------------------------
cluster.name: dmoespd

# ------------------------------------ Node ------------------------------------
node.name: dmoespd01

# coordinating/lb Node
node.master: false
node.data: false
node.ingest: false
cluster.remote.connect: false

# ----------------------------------- Memory -----------------------------------
bootstrap.memory_lock: true
bootstrap.system_call_filter : false

# -----------------------------apps----- Network -----------------------------------
network.host: 192.168.104.51
http.port: 6530
transport.tcp.port: 6540

# --------------------------------- Discovery ----------------------------------
discovery.seed_hosts: ["192.168.104.51:6540", "192.168.104.52:6540"]
cluster.initial_master_nodes: ["dmoespd02"]

# --------------------------------- Paths ----------------------------------
path.data: /data/elasticsearch
path.logs: /logs/elasticsearch

thread_pool.search.size: 50
thread_pool.search.queue_size: 2000

# CORS settings.
http.cors.enabled: true
http.cors.allow-origin: "*"
http.cors.allow-methods : OPTIONS, HEAD, GET, POST
http.cors.allow-headers : X-Requested-With,X-Auth-Token,Content-Type, >Content-Length
http.cors.allow-credentials: true
```

>dmoespd02
```
# ---------------------------------- Cluster -----------------------------------
cluster.name: dmoespd

# ------------------------------------ Node ------------------------------------
node.name: dmoespd02

# master-eligible/data Node
node.master: true
node.data: true
node.ingest: false
cluster.remote.connect: false
discovery.seed_hosts: ${DISCOVERY_SERVICE}
cluster.initial_master_nodes: ${MASTER_NODES}
#discovery.zen.minimum_master_nodes: 2(deprecated)

# ----------------------------------- Memory -----------------------------------
bootstrap.memory_lock: true
bootstrap.system_call_filter : false

# -----------------------------apps----- Network -----------------------------------
network.host: 192.168.104.52
http.port: 6530
transport.tcp.port: 6540

# --------------------------------- Discovery ----------------------------------
discovery.seed_hosts: ["192.168.104.51:6540", "192.168.104.52:6540"]
cluster.initial_master_nodes: ["dmoespd02"]

# --------------------------------- Paths ----------------------------------
path.data: /data/elasticsearch
path.logs: /logs/elasticsearch

thread_pool.search.size: 50
thread_pool.search.queue_size: 2000

# CORS settings.
http.cors.enabled: true
http.cors.allow-origin: "*"
http.cors.allow-methods : OPTIONS, HEAD, GET, POST
http.cors.allow-headers : X-Requested-With,X-Auth-Token,Content-Type, >Content-Length
http.cors.allow-credentials: true
```

vi /apps/elasticsearch/elasticsearch
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

chmod 755 /apps/elasticsearch/elasticsearch

/apps/elasticsearch/elasticsearch start

# install : kibana

su - app

mkdir -p /apps/kibana
mkdir -p /data/kibana
mkdir -p /logs/kibana

cd /apps/install
curl -O https://artifacts.elastic.co/downloads/kibana/kibana-7.1.1-linux-x86_64.tar.gz

tar -zxvf /apps/install/kibana-7.1.1-linux-x86_64.tar.gz

cp -R /apps/install/kibana-7.1.1-linux-x86_64 /apps/kibana/7.1.1	

cd /apps/kibana/7.1.1/config
cp kibana.yml kibana.yml.default

vi /apps/kibana/7.1.1/config/kibana.yml
```
server.port: 6636
server.host: "192.168.104.51"
elasticsearch.hosts: ["http://192.168.104.51:6530"]
```

vi /apps/kibana/kibana
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
KIBANA_HOME=/apps/kibana/7.1.1
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

chmod 755 /apps/kibana/kibana

/apps/kibana/kibana start

>외부에서 docker container에 접속하기 위해 port mapping이 필요하다.

http://172.20.0.104:6636

# add port mapping : docker container

docker stop mobon.elasticsearch.01
docker stop mobon.elasticsearch.02

sudo systemctl stop docker

vi /data/docker/containers/bef735334fb8ee57d1a7b97b449deed5b9e96059f22be8257a0be385cd94dfee/config.v2.json
```
..."AttachStderr":false,"ExposedPorts":{"6530/tcp":{},"6636/tcp":{}},"Tty":...
..."Service":null,"Ports":{"6530/tcp":[{"HostIp":"","HostPort":"6530"}],"6636/tcp":[{"HostIp":"","HostPort":"6636"}]},"SandboxKey":...
```

vi /data/docker/containers/bef735334fb8ee57d1a7b97b449deed5b9e96059f22be8257a0be385cd94dfee/hostconfig.json
```
...,"NetworkMode":"mobon.subnet","PortBindings":{"6530/tcp":[{"HostIp":"","HostPort":"6530"}], "6636/tcp":[{"HostIp":"","HostPort":"6636"}]},"RestartPolicy":...
```

sudo systemctl start docker

docker start mobon.elasticsearch.01
docker start mobon.elasticsearch.02



vi /apps/elasticsearch/elasticsearch
chmod 755 /apps/elasticsearch/elasticsearch
