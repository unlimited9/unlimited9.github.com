# install.n setup : kafka

>Created 월요일 04 12월 2017
distributed streaming platform : Server.setting - Failover and high availability

# Installation/Setup/Configuration

## 1. Pre-installation setup

### A. creating required operating system group and user

#### create the os application group(typically, app)  
$ groupadd -g 3000 app

> 어플리케이션 단위 관리 주체가 다르지 않고 권한을 나눌 필요가 없어 통합 계정을 생성하여 관리
> ~~어플리케이션 단위 관리 주체가 다르다면 어플리케이션별 계정을 생성하여 생성하여 관리~~

#### create the software unified account (typically, app)
$ useradd -d /apps -g 3000 -m -u 3000 -s /bin/bash app
$ passwd app

#### create the kafka software owner (typically, kafka)  
~~$ useradd -d /apps/kafka -g 3000 -m -u 3460 -s /bin/bash kafka  
$ passwd kafka~~

### B. install dependency packages
>kafka(brokers, producers and consumers) use zookeeper to manage and share state

### C. creating base directory

#### 디렉토리 생성
$ mkdir -p /apps/install /pgms /data /logs

#### 소유권 변경
$ chown -R app:app /apps /pgms /data /logs

### D. firewall configuration

$ vi /etc/sysconfig/iptables  
```
...  
# kafka  
-A INPUT -m state --state NEW -m tcp -p tcp --dport 7642 -j ACCEPT  
...
```

#### restart iptalbes service  
$ service iptables restart

$ iptables -nL

## 2. installation setup

### A. change account

$ su - app

### B. creating application directory

#### make directory
$ mkdir -p /apps/kafka /data/kafka /logs/kafka

### C. download

#### http://kafka.apache.org
$ curl -O http://apache.mirror.cdnetworks.com/kafka/2.2.0/kafka_2.12-2.2.0.tgz -P /apps/install
~~$ wget http://apache.mirror.cdnetworks.com/kafka/2.2.0/kafka_2.12-2.2.0.tgz -P /apps/install~~

### D. install

#### decompress tarball  
$ tar -zxvf /apps/install/kafka_2.12-2.2.0.tgz

#### copy application directory  
$ cp -R /apps/install/kafka_2.12-2.2.0 /apps/kafka/2.12-2.2.0

### E. configure

$ vi /apps/kafka/2.12-2.2.0/config/server.properties
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

> 개발서버 1기에 설치 시
>zookeeper.connect=172.20.0.102:17214,172.20.0.102:27214,172.20.0.102:37214


### F. run application

#### start process

$ /apps/kafka/2.12-2.2.0/bin/kafka-server-start.sh -daemon /apps/kafka/2.12-2.2.0/config/server.properties

>하나의 node에 여러 인스턴스를 띄워야 할 경우 아래와 같이 설정파일을 분리해서 기동한다.  
>$ /apps/kafka/2.12-2.2.0/bin/kafka-server-start.sh -daemon /apps/kafka/instances/01/config/server.properties  
>$ /apps/kafka/2.12-2.2.0/bin/kafka-server-start.sh -daemon /apps/kafka/instances/01/config/server.properties


>kafka-manager 등을 통해 JMX로 모니터링을 위해 jmx_port를 지정해 기동한다.  
>$ env JMX_PORT=19999 /apps/kafka/2.12-2.2.0/bin/kafka-server-start.sh -daemon /apps/kafka/instances/01/config/server.properties  
>$ env JMX_PORT=29999 /apps/kafka/2.12-2.2.0/bin/kafka-server-start.sh -daemon /apps/kafka/instances/02/config/server.properties


#### check process and port

$ ps -ef | grep kafka  
>...  
/apps/jdk/1.8.0_152/bin/java -Xmx1G -Xms1G -server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+ExplicitGCInvokesConcurrent -Djava.awt.headless=true -Xloggc:/apps/kafka/2.12-2.2.0/bin/../logs/kafkaServer-gc.log -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=100M -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Dkafka.logs.dir=/apps/kafka/2.12-2.2.0/bin/../logs -Dlog4j.configuration=file:/apps/kafka/2.12-2.2.0/bin/../config/log4j.properties -cp :/apps/kafka/2.12-2.2.0/bin/../libs/aopalliance-repackaged-2.5.0-b32.jar:/apps/kafka/2.12-2.2.0/bin/../libs/argparse4j-0.7.0.jar:/apps/kafka/2.12-2.2.0/bin/../libs/commons-lang3-3.5.jar:/apps/kafka/2.12-2.2.0/bin/../libs/connect-api-1.0.0.jar:/apps/kafka/2.12-2.2.0/bin/../libs/connect-file-1.0.0.jar:/apps/kafka/2.12-2.2.0/bin/../libs/connect-json-1.0.0.jar:/apps/kafka/2.12-2.2.0/bin/../libs/connect-runtime-1.0.0.jar:/apps/kafka/2.12-2.2.0/bin/../libs/connect-transforms-1.0.0.jar:/apps/kafka/2.12-2.2.0/bin/../libs/guava-20.0.jar:/apps/kafka/2.12-2.2.0/bin/../libs/hk2-api-2.5.0-b32.jar:/apps/kafka/2.12-2.2.0/bin/../libs/hk2-locator-2.5.0-b32.jar:/apps/kafka/2.12-2.2.0/bin/../libs/hk2-utils-2.5.0-b32.jar:/apps/kafka/2.12-2.2.0/bin/../libs/jackson-annotations-2.9.1.jar:/apps/kafka/2.12-2.2.0/bin/../libs/jackson-core-2.9.1.jar:/apps/kafka/2.12-2.2.0/bin/../libs/jackson-databind-2.9.1.jar:/apps/kafka/2.12-2.2.0/bin/../libs/jackson-jaxrs-base-2.9.1.jar:/apps/kafka/2.12-2.2.0/bin/../libs/jackson-jaxrs-json-provider-2.9.1.jar:/apps/kafka/2.12-2.2.0/bin/../libs/jackson-module-jaxb-annotations-2.9.1.jar:/apps/kafka/2.12-2.2.0/bin/../libs/javassist-3.20.0-GA.jar:/apps/kafka/2.12-2.2.0/bin/../libs/javassist-3.21.0-GA.jar:/apps/kafka/2.12-2.2.0/bin/../libs/javax.annotation-api-1.2.jar:/apps/kafka/2.12-2.2.0/bin/../libs/javax.inject-1.jar:/apps/kafka/2.12-2.2.0/bin/../libs/javax.inject-2.5.0-b32.jar:/apps/kafka/2.12-2.2.0/bin/../libs/javax.servlet-api-3.1.0.jar:/apps/kafka/2.12-2.2.0/bin/../libs/javax.ws.rs-api-2.0.1.jar:/apps/kafka/2.12-2.2.0/bin/../libs/jersey-client-2.25.1.jar:/apps/kafka/2.12-2.2.0/bin/../libs/jersey-common-2.25.1.jar:/apps/kafka/2.12-2.2.0/bin/../libs/jersey-container-servlet-2.25.1.jar:/apps/kafka/2.12-2.2.0/bin/../libs/jersey-container-servlet-core-2.25.1.jar:/apps/kafka/2.12-2.2.0/bin/../libs/jersey-guava-2.25.1.jar:/apps/kafka/2.12-2.2.0/bin/../libs/jersey-media-jaxb-2.25.1.jar:/apps/kafka/2.12-2.2.0/bin/../libs/jersey-server-2.25.1.jar:/apps/kafka/2.12-2.2.0/bin/../libs/jetty-continuation-9.2.22.v20170606.jar:/apps/kafka/2.12-2.2.0/bin/../libs/jetty-http-9.2.22.v20170606.jar:/apps/kafka/2.12-2.2.0/bin/../libs/jetty-io-9.2.22.v20170606.jar:/apps/kafka/2.12-2.2.0/bin/../libs/jetty-security-9.2.22.v20170606.jar:/apps/kafka/2.12-2.2.0/bin/../libs/jetty-server-9.2.22.v20170606.jar:/apps/kafka/2.12-2.2.0/bin/../libs/jetty-servlet-9.2.22.v20170606.jar:/apps/kafka/2.12-2.2.0/bin/../libs/jetty-servlets-9.2.22.v20170606.jar:/apps/kafka/2.12-2.2.0/bin/../libs/jetty-util-9.2.22.v20170606.jar:/apps/kafka/2.12-2.2.0/bin/../libs/jopt-simple-5.0.4.jar:/apps/kafka/2.12-2.2.0/bin/../libs/kafka_2.12-2.2.0.jar:/apps/kafka/2.12-2.2.0/bin/../libs/kafka_2.12-2.2.0-sources.jar:/apps/kafka/2.12-2.2.0/bin/../libs/kafka_2.12-2.2.0-test-sources.jar:/apps/kafka/2.12-2.2.0/bin/../libs/kafka-clients-1.0.0.jar:/apps/kafka/2.12-2.2.0/bin/../libs/kafka-log4j-appender-1.0.0.jar:/apps/kafka/2.12-2.2.0/bin/../libs/kafka-streams-1.0.0.jar:/apps/kafka/2.12-2.2.0/bin/../libs/kafka-streams-examples-1.0.0.jar:/apps/kafka/2.12-2.2.0/bin/../libs/kafka-tools-1.0.0.jar:/apps/kafka/2.12-2.2.0/bin/../libs/log4j-1.2.17.jar:/apps/kafka/2.12-2.2.0/bin/../libs/lz4-java-1.4.jar:/apps/kafka/2.12-2.2.0/bin/../libs/maven-artifact-3.5.0.jar:/apps/kafka/2.12-2.2.0/bin/../libs/metrics-core-2.2.0.jar:/apps/kafka/2.12-2.2.0/bin/../libs/osgi-resource-locator-1.0.1.jar:/apps/kafka/2.12-2.2.0/bin/../libs/plexus-utils-3.0.24.jar:/apps/kafka/2.12-2.2.0/bin/../libs/reflections-0.9.11.jar:/apps/kafka/2.12-2.2.0/bin/../libs/rocksdbjni-5.7.3.jar:/apps/kafka/2.12-2.2.0/bin/../libs/scala-library-2.11.11.jar:/apps/kafka/2.12-2.2.0/bin/../libs/slf4j-api-1.7.25.jar:/apps/kafka/2.12-2.2.0/bin/../libs/slf4j-log4j12-1.7.25.jar:/apps/kafka/2.12-2.2.0/bin/../libs/snappy-java-1.1.4.jar:/apps/kafka/2.12-2.2.0/bin/../libs/validation-api-1.1.0.Final.jar:/apps/kafka/2.12-2.2.0/bin/../libs/zkclient-0.10.jar:/apps/kafka/2.12-2.2.0/bin/../libs/zookeeper-3.4.10.jar kafka.Kafka /apps/kafka/2.12-2.2.0/config/server.properties  
...

$ netstat -na | grep LISTEN  
>...  
tcp 0 0 0.0.0.0:7642 0.0.0.0:* LISTEN  
...

#### stop process

$ /apps/kafka/2.12-2.2.0/bin/kafka-server-stop.sh

>기본 경로 밖의 설정파일을 지정한 경우 설정파일을 지정해 중지한다.  
>$ /apps/kafka/2.12-2.2.0/bin/kafka-server-stop.sh /apps/kafka/instances/01/config/server.properties


## 3. post-installation setup

### A. create shell script

#### kafka

$ vi /apps/kafka/kafka  
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

$ chmod 755 /apps/kafka/kafka

### B. add service

$ cp /apps/kafka/kafka /etc/rc.d/init.d/kafka  
$ chkconfig --add kafka

### C. configure SELinux(optional)

$ vi /etc/selinux/config  
```
SELINUX=disabled
```

## 4. execution(starting and stopping daemon services)

### A. start service

$ service kafka start

### B. restart service

$ service kafka restart

### C. stop service

$ service kafka stop

## 8. trouble-shooting

### A. ipv4(xxx.xxx.xxx.xxx)로 통신을 해야하는데 ipv6로 기동될 경우

>  jvm 상에 기동되는 hadoop ecosystem 등의 많은 솔루션에서 발생하여 b. disable IPv6 on Server를 하는 방법으로 일괄 처리
  
#### a. JAVA ipv4,ipv6 설정 방법
> Ipv4/Ipv6 system properties stack 우선 순위를 변경한다.

$ vi /apps/kafka/2.12-2.2.0/bin/kafka-server-start.sh
```
...
export KAFKA_OPTS="-Djava.net.preferIPv4Stack=true"
exec $base_dir/kafka-run-class.sh $EXTRA_ARGS kafka.Kafka "$@"
```

>export KAFKA_OPTS="-Djava.net.preferIPv4Stack=true"

>-Djava.net.preferIPv4Stack=true
~~-Djava.net.preferIPv6Addresses=false~~


#### b. disable IPv6 on Server
ref. 30.architecture/30.technical/10.system/z.how.to#How to disable IPv6 on Linux

## 9. Appendix

### reference site

#### Apache Kafka Documentation  
http://kafka.apache.org/documentation/

#### Kafka 운영자가 말하는 처음 접하는 Kafka  
http://www.popit.kr/kafka-%EC%9A%B4%EC%98%81%EC%9E%90%EA%B0%80-%EB%A7%90%ED%95%98%EB%8A%94-%EC%B2%98%EC%9D%8C-%EC%A0%91%ED%95%98%EB%8A%94-kafka/

#### Kafka 운영자가 말하는 Topic Replication  
http://www.popit.kr/kafka-%EC%9A%B4%EC%98%81%EC%9E%90%EA%B0%80-%EB%A7%90%ED%95%98%EB%8A%94-topic-replication/

#### Apache Kafka 클러스터링 구축 및 테스트  
http://programist.tistory.com/entry/Apache-Kafka-%ED%81%B4%EB%9F%AC%EC%8A%A4%ED%84%B0%EB%A7%81-%EA%B5%AC%EC%B6%95-%EB%B0%8F-%ED%85%8C%EC%8A%A4%ED%8A%B8

#### [Apache Kafka] 2. 클러스터 구축하기  
http://epicdevs.com/20

#### kafka centos 확인사항  
http://ujfish-tools.tistory.com/entry/kafka-centos-%ED%99%95%EC%9D%B8%EC%82%AC%ED%95%AD
