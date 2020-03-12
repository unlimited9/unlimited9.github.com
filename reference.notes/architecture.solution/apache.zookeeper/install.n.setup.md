# 01.install.n setup : Zookeeper

>Created 목요일 30 11월 2017
Distributed Coordination Service

# Installation/Setup/Configuration

## 1. Pre-installation setup

#### A. creating required operating system group and user
[Create operating system group and user](../system/management.account.n.group.md)

### B. install dependency packages
>zooKeeper requires Java to run

### C. creating base directory

#### 디렉토리 생성
[Create operating system drectory](../system/management.directory.md)

### D. firewall configuration

$ vi /etc/sysconfig/iptables
```
...  
# zookeeper  
-A INPUT -m state --state NEW -m tcp -p tcp --dport 7214 -j ACCEPT  
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
$ mkdir -p /apps/zookeeper /data/zookeeper /logs/zookeeper

### C. download

#### http://zookeeper.apache.org
$ curl -O http://apache.tt.co.kr/zookeeper/zookeeper-3.5.6/apache-zookeeper-3.5.6.tar.gz -P /apps/install
~~$ wget http://apache.tt.co.kr/zookeeper/zookeeper-3.5.6/apache-zookeeper-3.5.6.tar.gz -P /apps/install~~

### D. install

#### decompress tarball
$ tar -zxvf /apps/install/apache-zookeeper-3.5.6.tar.gz

#### copy application directory  
$ cp -R /apps/install/apache-zookeeper-3.5.6 /apps/zookeeper/3.5.6

### E. configure

#### create a new zoo.cnf file by copying an existing template. 
$ cp -rfp /apps/zookeeper/3.5.6/conf/zoo_sample.cfg /apps/zookeeper/3.5.6/conf/zoo.cfg

$ vi /apps/zookeeper/3.5.6/conf/zoo.cfg  
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
server.1=10.10.10.25:7224:7234  
server.2=10.10.10.26:7224:7234  
server.3=10.10.10.27:7224:7234
```

>-   `tickTime`: Heartbeat 간 사용되는 timeout
>-   `dataDir`: Data디렉토리로 사용되며  **Producton**환경에서는 분리필요!! (성능가 발생할 정도라면 반드시 분리)
>-   `clientPort`: Service port
>-   `initLimit`: 책임자 연결에 허용된 시간, 과반수 후보자가 초과시 다른 책임자 선출
>-   `syncLimit`: 후보자에게 허용된 data sync 시간, 초과시 클라이언트는 다른 서버로 연결

>-   `autopurge.snapRetainCount`: 여기서는 3개를 남기고 모두 지우도록 입력했습니다.
>-   `autopurge.purgeInterval`: 1 시간 마다 스냅샷을 찍어냅니다. (기본값은 0이며, 0이면 동작하지 않음)
    
>-   `server.1=10.10.10.25:7224:7234`:  `1`은 myid,  `10.10.10.25`은 myid1의 IP,  `7224`은 follow가 leader와의 통신간 사용하는 port,  `7234`은 리더선출을 위한 port


#### logging  
$ vi /apps/zookeeper/3.5.6/conf/log4j.properties  
```
# Define some default values that can be overridden by system properties  
zookeeper.root.logger=INFO, CONSOLE  
zookeeper.console.threshold=INFO  
zookeeper.log.dir=/logs/zookeeper  
zookeeper.log.file=zookeeper.log  
zookeeper.log.threshold=DEBUG  
zookeeper.tracelog.dir=/logs/zookeeper  
zookeeper.tracelog.file=zookeeper_trace.log  
...
```

#### create myid : 10.10.10.25, 10.10.10.26, 10.10.10.27
$ echo 1 > /data/zookeeper/myid  
$ echo 2 > /data/zookeeper/myid  
$ echo 3 > /data/zookeeper/myid

### F. run application

#### start process

$ /apps/zookeeper/3.5.6/bin/zkServer.sh start

>하나의 node에 여러 인스턴스를 띄워야 할 경우 아래와 같이 설정파일을 분리해서 기동한다.
>/apps/zookeeper/3.5.6/bin/zkServer.sh start /apps/zookeeper/instances/01/conf/zoo.cfg

#### check status

$ /apps/zookeeper/3.5.6/bin/zkServer.sh status

>기본 경로 밖의 설정파일을 지정한 경우 상태 확인도 설정파일을 지정해 확인한다.
>/apps/zookeeper/3.5.6/bin/zkServer.sh status /apps/zookeeper/instances/01/conf/zoo.cfg

#### check process and port

$ ps -ef | grep zkServer  
> ...  
/apps/zookeeper/3.5.6/bin/zkServer.sh start  
...

$ netstat -na | grep LISTEN  
> ...  
tcp 0 0 0.0.0.0:7214 0.0.0.0:* LISTEN  
...

#### stop process

$ /apps/zookeeper/3.5.6/bin/zkServer.sh stop

>기본 경로 밖의 설정파일을 지정한 경우 설정파일을 지정해 중지한다.
>/apps/zookeeper/3.5.6/bin/zkServer.sh stop /apps/zookeeper/instances/01/conf/zoo.cfg

## 3. post-installation setup

### A. create shell script

#### zookeeper

$ vi /apps/zookeeper/zookeeper  
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

### B. add service

$ cp /apps/zookeeper/zookeeper /etc/rc.d/init.d/zookeeper  
$ chkconfig --add zookeeper

### C. configure SELinux(optional)

$ vi /etc/selinux/config  
```
SELINUX=disabled
```

## 4. execution(starting and stopping daemon services)

### A. start service

$ service zookeeper start

### B. restart service

$ service zookeeper restart

### C. stop service

$ service zookeeper stop

## 8. trouble-shooting

### A. ipv4(xxx.xxx.xxx.xxx)로 통신을 해야하는데 ipv6로 기동될 경우

>  jvm 상에 기동되는 hadoop ecosystem 등의 많은 솔루션에서 발생하여 b. disable IPv6 on Server를 하는 방법으로 일괄 처리
  
#### a. JAVA ipv4,ipv6 설정 방법
> Ipv4/Ipv6 system properties stack 우선 순위를 변경한다.

$ vi /apps/zookeeper/3.5.6/bin/zkServer.sh
```
...
    nohup "$JAVA" "-Djava.net.preferIPv4Stack=true" "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" "-Dzookeeper.root.logger=${ZOO_LOG4J_PROP}" \
    -cp "$CLASSPATH" $JVMFLAGS $ZOOMAIN "$ZOOCFG" > "$_ZOO_DAEMON_OUT" 2>&1 < /dev/null &
...
```

>-Djava.net.preferIPv4Stack=true
~~-Djava.net.preferIPv6Addresses=false~~

#### b. disable IPv6 on Server
ref. 30.architecture/30.technical/10.system/z.how.to#How to disable IPv6 on Linux

## 9. Appendix

### reference site

#### apache zookeeper  
http://zookeeper.apache.org

#### apache zookeeper document  
http://zookeeper.apache.org/doc/current/
http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance

#### Apache ZooKeeper (아파치 주키퍼) 분산 코디네이터 소개 및 설치하기  
http://kingname.tistory.com/45

#### ZooKeeper Clustering 설치  
http://mslee89.tistory.com/188

#### [Zookeeper] 설치 DB&NoSql/Zookeeper
http://ilovehsk.tistory.com/181

#### 주키퍼 설치 및 설정  
http://wonwoo.ml/index.php/post/142

#### [Zookeeper] 설치 동일 장비에 멀티 인스턴스
https://uncle-bae.blogspot.com/2016/02/zookeeper_18.html

#### Zookeeper 설치
https://dec9th.github.io/2018-03-14-zookeeper-installation

#### ZooKeeper ▸ Installation - ZooKeeper Standalone Server
http://www.mtitek.com/tutorials/zookeeper/installation_standalone.php
