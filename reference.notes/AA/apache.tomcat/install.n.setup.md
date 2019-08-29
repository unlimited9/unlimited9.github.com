
# 01.install.n setup : tomcat

>Created 목요일 30 11월 2017
Server.setting - application

# Installation/Setup/Configuration

## 1. Pre-installation setup

### A. creating required operating system group and user

[Create operating system group and user](/reference.notes/TA/system/create.account.n.group.md)

### B. install dependency packages

>[tomcat requires Java to run](/reference.notes/AA/JDK/install.n.setup.md)

### C. creating base directory

[Create operating system drectory](/reference.notes/TA/system/create.directory.md)

### D. firewall configuration

$ vi /etc/sysconfig/iptables
```
...  
## tomcat : 3030  
# http  
-A INPUT -m state --state NEW -m tcp -p tcp --dport 3309 -j ACCEPT  
# https  
-A INPUT -m state --state NEW -m tcp -p tcp --dport 3319 -j ACCEPT
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
$ mkdir -p /apps/tomcat  
$ mkdir -p /data/tomcat  
$ mkdir -p /logs/tomcat

$ mkdir -p /logs/tomcat  
$ mkdir -p /pgms/tomcat/webapps  
$ mkdir -p /pgms/tomcat/wars  
$ mkdir -p /pgms/tomcat/backup

### C. download

#### Tomcat(http://tomcat.apache.org/)  

$ cd /apps/install
$ curl -O http://mirror.apache-kr.org/tomcat/tomcat-9/v9.0.20/bin/apache-tomcat-9.0.20.tar.gz  
~~$ wget http://mirror.apache-kr.org/tomcat/tomcat-9/v9.0.20/bin/apache-tomcat-9.0.20.tar.gz -P /apps/install~~

### D. install

#### decompress tarball
$ tar -zxvf /apps/install/apache-tomcat-9.0.20.tar.gz

#### copy application directory
$ cp -R apache-tomcat-9.0.20 /apps/tomcat/9.0.20	

### E. instance configure

#### make directory
$ mkdir -p \  
/apps/tomcat/instances/es.01/bin  \  
/apps/tomcat/instances/es.01/conf \  
/apps/tomcat/instances/es.01/logs \  
/apps/tomcat/instances/es.01/temp \  
/apps/tomcat/instances/es.01/webapps/ROOT

$ cp -R /apps/tomcat/9.0.20/conf/* /apps/tomcat/instances/es.01/conf

#### port - system property  
$ sed -i -e 's/8005/\${tomcat\.port\.shutdown}/g' /apps/tomcat/instances/es.01/conf/server.xml  
$ sed -i -e 's/8080/\${tomcat\.port\.http}/g' /apps/tomcat/instances/es.01/conf/server.xml  
$ sed -i -e 's/8009/\${tomcat\.port\.ajp}/g' /apps/tomcat/instances/es.01/conf/server.xml  
$ sed -i -e 's/8443/\${tomcat\.port\.https}/g' /apps/tomcat/instances/es.01/conf/server.xml  

#### application context 
$ vi /apps/tomcat/instances/es.01/conf/server.xml
```
    ...
    <Host name="localhost"  appBase="/pgms/tomcat/webapps"
          unpackWARs="true" autoDeploy="true">
    
        <Context path="/default" docBase="es" reloadable="false" />
        
        <!-- SingleSignOn valve, share authentication between web applications
             Documentation at: /docs/config/valve.html -->
        <!--
        <Valve className="org.apache.catalina.authenticator.SingleSignOn" />
        -->
        
        <!-- Access log processes all example.
             Documentation at: /docs/config/valve.html
             Note: The pattern used is equivalent to using pattern="common" -->
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
        
    </Host>
    ...
```

### F. create instance shell script

$ vi /apps/tomcat/instances/es.01/bin/tomcat.sh  
```
#!/bin/sh
# tomcat    tomcat service shell
# chkconfig: 2345 90 90
# description: tomcat.es.01
# processname: tomcat
# config: $TOMCAT_CONF
# pidfile:

SERVER_NAME=es.01
TOMCAT_USER=app
SHUTDOWN_WAIT=10
UNAME=`id -u -n`

export JAVA_OPTS="-server"
export JAVA_OPTS="$JAVA_OPTS -Xms2048m"
export JAVA_OPTS="$JAVA_OPTS -Xmx2048m"
export JAVA_OPTS="$JAVA_OPTS -XX:NewSize=128m"
export JAVA_OPTS="$JAVA_OPTS -XX:MaxNewSize=128m"
export JAVA_OPTS="$JAVA_OPTS -XX:PermSize=128m"
export JAVA_OPTS="$JAVA_OPTS -XX:MaxPermSize=128m"
export JAVA_OPTS="$JAVA_OPTS -XX:+DisableExplicitGC"

export JAVA_OPTS="$JAVA_OPTS -Dtomcat.port.shutdown=8005"
export JAVA_OPTS="$JAVA_OPTS -Dtomcat.port.http=8080"
export JAVA_OPTS="$JAVA_OPTS -Dtomcat.port.https=8443"
export JAVA_OPTS="$JAVA_OPTS -Dtomcat.port.ajp=8009"

export JAVA_OPTS="$JAVA_OPTS -Dspring.profiles.active=dev"

export CATALINA_HOME=/apps/tomcat/9.0.20
export CATALINA_BASE=/apps/tomcat/instances/$SERVER_NAME
export CATALINA_OPTS="$CATALINA_OPTS -Djava.security.egd=file:/dev/./urandom"

export CATALINA_OUT="/dev/null"

tomcat_pid() {
	echo `ps aux | grep "java" | grep "$CATALINA_BASE" | grep -v grep | awk '{ print $2 }'`
}

start() {
	pid=$(tomcat_pid)
	if [ -n "$pid" ]
	then
		echo "Tomcat is already running (pid: $pid)"
	else
		echo "Starting tomcat (pid: $pid)"

		if [ e$UNAME = "eroot" ]
		then
			ulimit -n 100000
			umask 007
			/bin/su -p -s /bin/sh $TOMCAT_USER $CATALINA_HOME/bin/startup.sh
		else
			$CATALINA_HOME/bin/startup.sh
		fi
	fi

	return 0
}

stop() {
	pid=$(tomcat_pid)

	if [ -n "$pid" ]
	then
		echo "Stoping Tomcat"

		if [ e$UNAME = "eroot" ]
		then
			/bin/su -p -s /bin/sh $TOMCAT_USER $CATALINA_HOME/bin/shutdown.sh
		else
			$CATALINA_HOME/bin/shutdown.sh -force
		fi

		let kwait=$SHUTDOWN_WAIT
		count=0;
		until [ `ps -p $pid | grep -c $pid` = '0' ] || [ $count -gt $kwait ]
		do
			echo -n -e "\nwaiting for processes to exit (pid: $pid)\n";
			sleep 1
			let count=$count+1;
		done

		if [ $count -gt $kwait ]; then
			echo -n -e "\nkilling processes which didn't stop after $SHUTDOWN_WAIT seconds (pid: $pid)"
			kill -9 $pid
		fi
	else
		echo "Tomcat is not running"
	fi

	return 0
}

case $1 in
	start)
		start
		;;
	stop)
		stop
		;;
	restart)
		stop
		start
		;;
	status)
		pid=$(tomcat_pid)
		if [ -n "$pid" ]
		then
			echo "Tomcat is running with pid: $pid"
		else
			echo "Tomcat is not running"
		fi
		;;
	*)
		echo $"Usage : $0 {start|stop|restart|status}"
		exit 1
esac
exit 0
```

$ chmod 755 /apps/tomcat/instances/es.01/bin/tomcat.sh

>$ vi  /apps/tomcat/instances/es.01/bin/setenv.sh
>```
>export JAVA_HOME=/apps/java
>#export LD_LIBRARY_PATH=/usr/local/apr/lib
>#export CATALINA_OPTS="-Duser.timezone=Asia/Seoul -Djava.library.path=$LD_LIBRARY_PATH"
>#CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote"
>#CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote.port=18012"
>#CATALINA_OPTS="$CATALINA_OPTS -Djava.rmi.server.hostname=172.20.0.103"
>#CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote.ssl=false"
>#CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote.authenticate=false"
>```
>
>$ chmod 755 /apps/tomcat/instances/es.01/bin/setenv.sh

#### set instance port
$ sed -i -e 's/8080/18080/g' /apps/tomcat/instances/es.01/bin/tomcat.sh  
$ sed -i -e 's/8443/18443/g' /apps/tomcat/instances/es.01/bin/tomcat.sh  
$ sed -i -e 's/8005/18005/g' /apps/tomcat/instances/es.01/bin/tomcat.sh  
$ sed -i -e 's/8009/18009/g' /apps/tomcat/instances/es.01/bin/tomcat.sh  

>####  set more instances
>`copy sample instance`  
$ cp -R /apps/tomcat/instances/es.01 /apps/tomcat/instances/new.instance.01
>
>`change instance ports`  
$ sed -i -e 's/es.01/new.instance.01/g' /apps/tomcat/instances/ngrinder/bin/tomcat.sh  
$ sed -i -e 's/18080/28080/g' /apps/tomcat/instances/new.instance.01/bin/tomcat.sh  
$ sed -i -e 's/18443/28443/g' /apps/tomcat/instances/new.instance.01/bin/tomcat.sh  
$ sed -i -e 's/18005/28005/g' /apps/tomcat/instances/new.instance.01/bin/tomcat.sh  
$ sed -i -e 's/18009/28009/g' /apps/tomcat/instances/new.instance.01/bin/tomcat.sh

### G. run application

#### start service
$ /apps/tomcat/instances/es.01/bin/tomcat start

#### check status
$ /apps/tomcat/instances/es.01/bin/tomcat status

#### check process and port
$ ps -ef | grep tomcat
$ netstat -na | grep LISTEN  

#### stop service
$ /apps/tomcat/instances/es.01/bin/tomcat stop

## 3. post-installation setup

### A. create shell script

#### 등록된 인스턴스들을 일괄적으로 start/stop
$ vi /apps/tomcat/tomcat
```
#!/bin/sh
# tomcat tomcat service shell
# chkconfig: 2345 90 90
# description: tomcat
# processname: tomcat
# config: $TOMCAT_CONF
# pidfile:

CATALINA_HOME="/apps/tomcat/9.0.20"
CATALINA_BASE="/apps/tomcat/instances"

INSTANCE_NAMES=(${2-"es.01" "es02"})

serviceCtrl() {
	# find $CATALINA_BASE -mindepth 1 -maxdepth 1 -type d | sort -t '\0' -n | while read instance

	for instance in "${INSTANCE_NAMES[@]}"
	do
		instance=$CATALINA_BASE/$instance
		case "$1" in
			start)
				echo -en "$instance Instance...\n"
				$instance/bin/tomcat.sh start
				;;
			stop)
				echo -en "$instance Instance...\n"
				$instance/bin/tomcat.sh stop
				;;
		esac
	done
}

case "$1" in
	start)
		echo -en "Starting Tomcat...\n"
		serviceCtrl start
		echo -e "\n"
		;;
	stop)
		echo -en "Shutting down Tomcat...\n"
		serviceCtrl stop
		echo -e "\n"
		;;
	restart)
		serviceCtrl stop
		sleep 5
		serviceCtrl start
		;;
	*)
		echo "Usage: $0 {start|stop|restart}"
		exit 1
esac
exit 0
```

### B. add service

$ cp /apps/tomcat/tomcat /etc/rc.d/init.d/tomcat

#### case of redhat
$ chkconfig --add tomcat  
$ chkconfig --level 35 tomcat on

$ chkconfig --list | grep tomcat  
```
nginx 0:off 1:off 2:off 3:on 4:off 5:on 6:off
```

>#### case of debian
>$ sudo update-rc.d -f tomcat defaults

### C. configure SELinux(optional)
$ vi /etc/selinux/config  
```
...
SELINUX=disabled
...
```

## 4. execution(starting and stopping daemon services)

#### A. start service
$ service tomcat start

#### B. restart service
$ service tomcat restart

#### C. stop service
$ service tomcat stop


## 8. trouble-shooting

## 9. Appendix

### reference site

* DBCP 소개/설정 및 validationQuery 이슈  
http://linuxism.tistory.com/579

* The Tomcat JDBC Connection Pool  
https://tomcat.apache.org/tomcat-7.0-doc/jdbc-pool.html
