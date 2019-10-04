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
>required JDK 1.8

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
>required JDK 1.8
>
>버전이 높을 경우 Compile 할때 아래와 같은 Error
>```
>ambiguous reference to overloaded definition,
>[error] both method putAll in class Properties of type (x$1: java.util.Map[_, _])Unit
>[error] and  method putAll in class Hashtable of type (x$1: java.util.Map[_ <: Object, _ <: Object])Unit
>[error] match argument types (java.util.Properties)
>[error]     props.putAll(overrides)
>[error]           ^
>```

#### make directory
$ mkdir -p /apps/kafka-manager
>$ mkdir -p /data/kafka-manager
$ mkdir -p /logs/kafka-manager

### C. download
>Kafka Manager(https://github.com/yahoo/kafka-manager)  
> : https://github.com/yahoo/kafka-manager/releases  
$ curl -O https://github.com/yahoo/kafka-manager/archive/2.0.0.2.tar.gz -P /apps/install  
~~$ wget https://github.com/yahoo/kafka-manager/archive/2.0.0.2.tar.gz -P /apps/install~~

### D. install

#### decompress tarball  
$ tar -zxvf /apps/install/2.0.0.2.tar.gz
>```
>tar: 이것은 tar 아카이브처럼 보이지 않습니다
>
>gzip: stdin: not in gzip format
>tar: Child returned status 1
>tar: Error is not recoverable: exiting now
>```
>$ file 2.0.0.2.tar.gz 
>```
>2.0.0.2.tar.gz: HTML document text
>```
>$ cat 2.0.0.2.tar.gz
>```
><html><body>You are being <a href="https://codeload.github.com/yahoo/kafka-manager/tar.gz/2.0.0.2">redirected</a>.</body></html>
>```
>$ curl -O https://codeload.github.com/yahoo/kafka-manager/tar.gz/2.0.0.2
>$ file 2.0.0.2
>```
>2.0.0.2: gzip compressed data, from Unix
>```
>$mv 2.0.0.2 2.0.0.2.tar.gz
>$ tar -zxvf /apps/install/2.0.0.2.tar.gz

#### compile/build
$ cd kafka-manager-2.0.0.2/
$ ./sbt clean dist
>compile  마지막에 error : required JDK 1.8
>```
>...
>[error] /apps/install/kafka-manager->2.0.0.2/app/kafka/manager/utils/zero90/LogConfig.scala:176:11: ambiguous reference to overloaded definition,
>[error] both method putAll in class Properties of type (x$1: java.util.Map[_, _])Unit
>[error] and  method putAll in class Hashtable of type (x$1: java.util.Map[_ <: Object, _ <: Object])Unit
>[error] match argument types (java.util.Map[_$3,_$4])
>[error]     props.putAll(defaults)
>[error]           ^
>[error] /apps/install/kafka-manager-2.0.0.2/app/kafka/manager/utils/zero90/LogConfig.scala:177:11: ambiguous reference to overloaded definition,
>[error] both method putAll in class Properties of type (x$1: java.util.Map[_, _])Unit
>[error] and  method putAll in class Hashtable of type (x$1: java.util.Map[_ <: Object, _ <: Object])Unit
>[error] match argument types (java.util.Properties)
>[error]     props.putAll(overrides)
>[error]           ^
>[error] 12 errors found
>[info] No documentation generated with unsuccessful compiler run
>[error] 12 errors found
>[error] (Compile / compileIncremental) Compilation failed
>[error] (Compile / doc) Scaladoc generation failed
>[error] Total time: 285 s, completed 2019. 6. 25. 오후 1:23:31
>```

#### extract zip
> $ yum install unzip

$ unzip -d /apps/kafka-manager /apps/install/kafka-manager-2.0.0.2/target/universal/kafka-manager-2.0.0.2.zip
$ mv /apps/kafka-manager/kafka-manager-2.0.0.2 /apps/kafka-manager/2.0.0.2 

### E. configure
$ cd /apps/kafka-manager/2.0.0.2/conf
$ cp application.conf application.conf.default
$ vi application.conf
```
...
kafka-manager.zkhosts="172.20.0.102:17214,172.20.0.102:27214,172.20.0.102:37214"
basicAuthentication.username="admin"  
basicAuthentication.password="password"
...
```

$ cp logback.xml logback.xml.default
$ vi logback.xml
```
...
    <logger name="kafka.manager" level="WARN" />
...
```

### F. run application

#### start process
$ nohup /apps/kafka-manager/2.0.0.2/bin/kafka-manager 1>/dev/null 2>&1 &
>/apps/kafka-manager/2.0.0.2/bin/kafka-manager &

#### check process and port
$ ps -ef | grep kafka-manager  

$ netstat -ntl
>$ netstat -na | grep LISTEN  
>...  
tcp        0      0 :::9000                     :::*                        LISTEN      
...

#### stop process
```
$ kill -9 `cat /apps/kafka-manager/2.0.0.2/RUNNING_PID`
```

## 9. Appendix

### reference site

#### Kafka Manager
https://github.com/yahoo/kafka-manager#deployment

#### Kafka Manager 설치 및 연동하기
https://blog.naver.com/PostView.nhn?blogId=occidere&logNo=221395731049&categoryNo=0&parentCategoryNo=0&viewDate=&currentPage=1&postListTopCurrentPage=1&from=postView
