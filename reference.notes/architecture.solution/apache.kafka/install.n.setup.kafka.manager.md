# Install and setup : kafka manager

>Created 월요일 04 12월 2017  
A tool for managing Apache Kafka.

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

### B. Requirements and install dependency packages
~~>required JDK 1.8~~  
~~>required JDK 12~~  
>required JDK 11  
Try building with Java 11 since CMAK 3.x.

### C. creating base directory

#### 디렉토리 생성
~~$ mkdir -p /apps~~
$ mkdir -p /apps/install
$ mkdir -p /pgms
$ mkdir -p /data
$ mkdir -p /logs

#### 소유권 변경
$ chown -R app:app /apps
$ chown -R app:app /pgms
$ chown -R app:app /data
$ chown -R app:app /logs

### D. firewall configuration

$ vi /etc/sysconfig/iptables  
```
...  
# kafka manager 
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
>required JDK 11  
>$ cd /app/install  
>$ wget https://download.java.net/openjdk/jdk11/ri/openjdk-11+28_linux-x64_bin.tar.gz  
>$ tar xvf openjdk-11+28_linux-x64_bin.tar.gz.tar.gz  
>$ mv jdk-11 /apps/jdk/11+28  
>set JAVA_HOME, PATH  

#### make directory
$ mkdir -p /apps/kafka-manager
>$ mkdir -p /data/kafka-manager
$ mkdir -p /logs/kafka-manager

### C. download
>Kafka Manager(https://github.com/yahoo/CMAK/releases)  

$ cd /apps/install  
$ curl -o https://github.com/yahoo/CMAK/releases/download/3.0.0.4/cmak-3.0.0.4.zip

### D. install

#### unzip archive  
$ unzip cmak-3.0.0.4.zip  
$ mv cmak-3.0.0.4 /apps/kafka-manager/3.0.0.4

>### C. download
>>Kafka Manager(https://github.com/yahoo/kafka-manager)  
>> : https://github.com/yahoo/CMAK.git  
>$ curl -O https://github.com/yahoo/CMAK/archive/master.zip  
>~~$ wget https://github.com/yahoo/CMAK/archive/master.zip~~  
>
>### D. install
>
>#### unzip archive  
>$ unzip master.zip  
>
>#### compile/build
>$ cd CMAK-master  
>
>>error: SBT Assembly about MergeStrategy
>>$ vi build.sbt
>>```
>>...
>>assemblyMergeStrategy in assembly := {
>>  case PathList("javax", "servlet", xs @ _*)         => MergeStrategy.first
>>  case PathList(ps @ _*) if ps.last endsWith ".html" => MergeStrategy.first
>>  case "application.conf"                            => MergeStrategy.concat
>>  case "unwanted.txt"                                => MergeStrategy.discard
>>  case x =>
>>    val oldStrategy = (assemblyMergeStrategy in assembly).value
>>    oldStrategy(x)
>>}
>>...
>>```
>
>$ ./sbt clean dist  
>
>$ cd /apps/install/CMAK-master/target/universal  
>$ unzip cmak-3.0.0.4.zip  
>$ mv cmak-3.0.0.4 /apps/kafka-manager/3.0.0.4

### E. configure
>$ cd /apps/kafka-manager/3.0.0.4/conf  
>$ cp application.conf application.conf.default  
>$ vi application.conf  
>```
>...
>kafka-manager.zkhosts="172.20.0.102:17214,172.20.0.102:27214,172.20.0.102:37214"
>...
>basicAuthentication.username="admin"  
>basicAuthentication.password="password"
>...
>```
>
>$ cp logback.xml logback.xml.default  
>$ vi logback.xml  
>```
>...
>    <logger name="kafka.manager" level="WARN" />
>...
>```

### F. run application

#### start process
$ nohup /apps/kafka-manager/3.0.0.4/bin/cmak -DZK_HOSTS=”172.20.0.102:17214,172.20.0.102:27214,172.20.0.102:37214″ 1>/dev/null 2>&1 &  
>$ /apps/kafka-manager/3.0.0.4/bin/cmak -DZK_HOSTS=”172.20.0.102:17214,172.20.0.102:27214,172.20.0.102:37214″ &  

>설정파일을 수정했다면...  
>$ nohup /apps/kafka-manager/3.0.0.4/bin/cmak 1>/dev/null 2>&1 &  
>>$ /apps/kafka-manager/3.0.0.4/bin/cmak &  

#### check process and port
$ ps -ef | grep kafka-manager  

$ netstat -ntl
>$ netstat -na | grep LISTEN  
>...  
tcp        0      0 :::9000                     :::*                        LISTEN      
...

#### stop process
```
$ kill -9 `cat /apps/kafka-manager/3.0.0.4/RUNNING_PID`
```
## Kafka Monitoring

#### Clusters > cluster01 > Topics
Metrics / Consumers consuming from this topic jmx 설정

> kafka 기동 시 JMX_PORT를 설정해 줘야 한다.  
$ env JMX_PORT=19999 /apps/kafka/2.12-2.2.0/bin/kafka-server-start.sh -daemon /apps/kafka/instances/01/config/server.properties  
$ env JMX_PORT=29999 /apps/kafka/2.12-2.2.0/bin/kafka-server-start.sh -daemon /apps/kafka/instances/02/config/server.properties

## trouble-shooting

#### Will not attempt to authenticate using SASL (unknown error)
Kubernetes Service로 zookeeper cluster client를 설정해서 연계구성했다. 위와 같은 개발환경에서는 문제가 없는데  
다음과 같이 CMAK를 기동 시  
```
$ nohup /apps/kafka-manager/3.0.0.4/bin/cmak -DZK_HOSTS="mobon-data-zookeeper-client-svc:7214" 1>/dev/null 2>&1 &
```
'Will not attempt to authenticate using SASL (unknown error)' 메세지와 함께 클러스터가 빠져서 CMAK 화면에 클러스터가 보이지 않는다.  
화면에 `Add Cluster`로 추가하면 잘 나오기는 하는데... 하니까 다음에 보자. ㅋ  

## 9. Appendix

#### reference site

+ Kafka Manager
https://github.com/yahoo/kafka-manager#deployment

+ Kafka Manager 설치 및 연동하기
https://blog.naver.com/PostView.nhn?blogId=occidere&logNo=221395731049&categoryNo=0&parentCategoryNo=0&viewDate=&currentPage=1&postListTopCurrentPage=1&from=postView

+ KAFKA/EXACTLYONCE  
https://www.joinc.co.kr/w/man/12/Kafka/exactlyonce  

+ How to install Kafka Manager for Managing kafka cluster  
http://manastri.blogspot.com/2020/03/how-to-install-kafka-manager-for.html



