# 10.install.n setup : kibana

>Created 목요일 30 11월 2017
데이터의 시각화. 전체 스택 탐색. : Kibana

# Installation/Setup/Configuration

## 1. installation setup : app

### A. change account

$ su - app

### B. creating application directory

#### make directory
$ mkdir -p /apps/kibana
$ mkdir -p /data/kibana
$ mkdir -p /logs/kibana

### C. download

#### Elastic(https://www.elastic.co/downloads)
$ cd /apps/install
$ curl -O https://artifacts.elastic.co/downloads/kibana/kibana-7.1.1-linux-x86_64.tar.gz
~~$ wget https://artifacts.elastic.co/downloads/kibana/kibana-7.1.1-linux-x86_64.tar.gz -P /apps/install~~

### D. install

#### decompress tarball
$ tar -zxvf /apps/install/kibana-7.1.1-linux-x86_64.tar.gz

#### copy application directory
$ cp -R /apps/install/kibana-7.1.1-linux-x86_64 /apps/kibana/7.1.1	

### E. configure
$ cd /apps/kibana/7.1.1/config

$ cp kibana.yml kibana.yml.default

$ vi /apps/kibana/7.1.1/config/kibana.yml
```
# Kibana is served by a back end server. This setting specifies the port to use.
server.port: 6636

# Specifies the address to which the Kibana server will bind. IP addresses and host names are both valid values.
# The default is 'localhost', which usually means remote machines will not be able to connect.
# To allow connections from remote users, set this parameter to a non-loopback address.
server.host: "192.168.104.51"

# The URLs of the Elasticsearch instances to use for all your queries.
elasticsearch.hosts: ["http://192.168.104.51:6530"]
```

### E. excute

#### start process
$ /apps/kibana/7.1.1/bin/kibana

### check process and port
$ ps -ef | grep kibana  
```
app       2657  1623 97 02:27 pts/2    00:00:12 /apps/kibana/7.1.1/bin/../node/bin/node --no-warnings --max-http-header-size=65536 /apps/kibana/7.1.1/bin/../src/cli
```

$ netstat -na | grep LISTEN  
```
tcp        0      0 192.168.104.51:6636     0.0.0.0:*               LISTEN     
```

#### stop process
$ ps -ef | grep kibana  
$ kill -9 <pid>

## 3. post-installation setup

#### A. create shell script

#### elasticsearch
$ vi /apps/kibana/kibana  
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

#### change mode  
$ chmod 755 /apps/kibana/kibana

#### B. add service
$ cp /apps/kibana/kibana /etc/rc.d/init.d/kibana  
$ chkconfig --add kibana

#### C. configure SELinux(optional)
$ vi /etc/selinux/config  
SELINUX=disabled

### 4. execution(starting and stopping daemon services)

#### A. start service
$ service kibana start

#### B. restart service
$ service kibana restart

#### C. stop service
$ service kibana stop

## 8. trouble-shooting

#### Kibana server is not ready yet
```
...
FATAL Error: Index .kibana belongs to a version of Kibana that cannot be automatically migrated. Reset it or use the X-Pack upgrade assistant
...
```
$ curl -X DELETE http://elasticsearch_client_server:9200/.kibana

## 9. Appendix

#### reference site

+ Highly Available and Scalable Elasticsearch on Kubernetes  
https://medium.com/faun/https-medium-com-thakur-vaibhav23-ha-es-k8s-7e655c1b7b61
