# install.n setup : sonarqube

>Created 월요일 20 11월 2017

정적코드분석 도구

# Installation/Setup/Configuration

## 1. Pre-installation setup

#### A. creating required operating system group and user
[Create operating system group and user](../system/management.account.n.group.md)

### B. install dependency packages
>#### case of redhat
>$ yum install -y unzip

[sonarqube requires Java to run](../../AA/JDK/install.n.setup.md)

### C. system kernel setting

### D. creating base directory

#### 디렉토리 생성
[Create operating system drectory](../system/management.directory.md)

### E. firewall configuration

## 2. installation setup

### A. change account

$ su - app

### B. creating application directory

#### make directory
$ mkdir -p /apps/sonarqube /data/sonarqube /logs/sonarqube

>$ docker run --net mobon.subnet --ip 192.168.104.91 --name mobon.qa.01 -d -p 19000:9000 -p 19092:9092 -it docker-registry.mobon.net:5000/mobon/java.app.env:latest
>$ docker exec -it mobon.qa.01 /bin/bash

### C. download

$ cd /apps/install

#### https://www.sonarqube.org/downloads/
$ curl -O https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.1.0.31237.zip

### D. install

#### decompress tarball  
$ unzip sonarqube-8.1.0.31237.zip

#### copy application directory
$ cp -R sonarqube-8.1.0.31237 /apps/sonarqube/8.1.0

### E. setup


### F. configure

>기본적으로 H2로 기동되지만 변경이 필요하면 수정
>
>#### backup the original configuration file by renaming it.  
>$ cp /apps/sonarqube/8.1.0/conf/sonar.properties /apps/sonarqube/8.1.0/conf/sonar.properties.bak
$ vi /apps/sonarqube/8.1.0/conf/sonar.properties

## 3. post-installation setup

#### A. create shell script

#### sonarqube
$ vi /apps/sonarqube/sonarqube
```
#!/bin/bash
# SonarQube
#chkconfig: 2345 80 05
#description: SonarQube
 
RUN_AS_USER=app
APPLICATION_HOME=/apps/sonarqube/8.1.0
 
start() {
        echo "Starting SonarQube: "
        if [ "x$USER" != "x$RUN_AS_USER" ]; then
          su - $RUN_AS_USER -c "$APPLICATION_HOME/bin/linux-x86-64/sonar.sh start"
        else
          $APPLICATION_HOME/bin/linux-x86-64/sonar.sh start
        fi
        echo "done."
}
stop() {
        echo "Shutting down SonarQube: "
        if [ "x$USER" != "x$RUN_AS_USER" ]; then
          su - $RUN_AS_USER -c "$APPLICATION_HOME/bin/linux-x86-64/sonar.sh stop"
        else
          $APPLICATION_HOME/bin/linux-x86-64/sonar.sh stop
        fi
        echo "done."
}
 
case "$1" in
  start)
        start
        ;;
  stop)
        stop
        ;;
  *)
        echo "Usage: $0 {start|stop}"
esac

exit 0

```

#### change mode  
$ chmod 755 /apps/sonarqube/sonarqube

#### B. add service
$ cp /apps/sonarqube/sonarqube /etc/rc.d/init.d/sonarqube  
$ chkconfig --add sonarqube

#### C. configure SELinux(optional)
$ vi /etc/selinux/config  
SELINUX=disabled

## 4. execution(starting and stopping daemon services)

### A. start service
$ service sonarqube start

> $ /apps/sonarqube/8.1.0/bin/linux-x86-64/sonar.sh start

### B. stop service
$ service sonarqube stop

## 9. Appendix

### reference site

#### SonarSource Product
https://confluence.curvc.com/display/ASD/SonarSource+Product

#### Linux SonarQube 설치
https://confluence.curvc.com/pages/viewpage.action?pageId=6160585

#### SonarScanner for Gradle
https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-gradle/

#### Gradle SonarQube
http://kwonnam.pe.kr/wiki/gradle/sonarqube
