## 1. nGrinder : controller

### Download
>https://github.com/naver/ngrinder/releases

$ cd /apps/install  
$ wget https://github.com/naver/ngrinder/releases/download/ngrinder-3.4.3-20190709/ngrinder-controller-3.4.3.war

### Requirement

#### nGrinder requires Java 1.6 over to run
ref. [Install and setup JAVA](../AA/install.n.setup.java.md)

nGrinder is distributed as a self executable web archive file(WAR) file just like Jenkins, you can put this archive file into your familiar web application server (like Tomcat) or run the package in the command line.

>2019.08.06 현재 openjdk version "12.0.2"에서 embedded jetty로 기동시 에러 
>```
>## HTTP ERROR: 503
>```
>
>#### Make directory and copy library
>$ mkdir -p /apps/ngrinder/controller
>
>$ cp /apps/install/ngrinder-controller-3.4.3.war /apps/ngrinder/controller
>
>#### Make shell script
>$ vi /apps/ngrinder/controller/startup.sh
>```
>#!/usr/bin/env bash
>
>nohup java -Xms2048m -Xmx2048m -XX:MaxPermSize=200m -jar /apps/ngrinder/controller/ngrinder-controller-3.4.3.war --port 8080 > nohup.out &
>sleep 1
>tail -100 nohup.out
>```
>
>$ chmod 755 /apps/ngrinder/controller/startup.sh

#### deploy to application server : tomcat 6.x over
>deploying a war file from ngrinder to tomcat  
>ref. 30.architecture/30.technical/30.solution/g.tomcat/01.install.n.setup

`copy sample instance`
$ cp -R /apps/tomcat/instances/es.01 /apps/tomcat/instances/ngrinder

`change instance ports`  
$ sed -i -e 's/es.01/ngrinder/g' /apps/tomcat/instances/ngrinder/bin/tomcat.sh  
$ sed -i -e 's/8080/18080/g' /apps/tomcat/instances/ngrinder/bin/tomcat.sh  
$ sed -i -e 's/8443/18443/g' /apps/tomcat/instances/ngrinder/bin/tomcat.sh  
$ sed -i -e 's/8005/18005/g' /apps/tomcat/instances/ngrinder/bin/tomcat.sh  
$ sed -i -e 's/8009/18009/g' /apps/tomcat/instances/ngrinder/bin/tomcat.sh

$ cp /apps/install/ngrinder-controller-3.4.3.war /pgms/tomcat/service/ngrinder/webapps/ROOT.war

`Run`  
$ /apps/tomcat/instances/ngrinder/bin/tomcat.sh start

`Access`  
http://192.168.100.9:18080  
admin / P@ssw0rd (default  : admin /admin)

## 2. nGrinder : agent

### Download
> nGrinder - controller admin menu / Download Agent

$ cd /apps/install  
$ wget http://192.168.100.9:18080/ngrinder-controller/agent/download/ngrinder-agent-3.4.3-192.168.100.9.tar  
$ tar xvf ngrinder-agent-3.4.3-192.168.100.9.tar  
$ mv /apps/install/ngrinder-agent /apps

#### Configuration
$ vi ~/.ngrinder_agent/agent.conf
```
common.start_mode=agent
agent.controller_host=192.168.100.9
agent.controller_port=16001
agent.region=NONE
#agent.host_id=
#agent.server_mode=true

# provide more agent java execution option if necessary.
#agent.java_opt=
# set following false if you want to use more than 1G Xmx memory per a agent process.
#agent.limit_xmx=true
# please uncomment the following option if you want to send all logs to the controller.
#agent.all_logs=true
# some jvm is not compatible with DNSJava. If so, set this false.
#agent.enable_local_dns=false
```

#### Run
$ ./apps/ngrinder-agent/run_agent.sh
> `error`  
0 [main] DEBUG Sigar  - no libsigar-amd64-linux.so in java.library.path: [/usr/java/packages/lib, /usr/lib64, /lib64, /lib, /usr/lib]
>
>#### Hyperic SIGAR : https://sourceforge.net/projects/sigar/
>$ cd /apps/install  
$ wget https://jaist.dl.sourceforge.net/project/sigar/sigar/1.6/hyperic-sigar-1.6.4.zip  
$ unzip hyperic-sigar-1.6.4.zip  
$ cp /apps/install/hyperic-sigar-1.6.4/sigar-bin/lib/libsigar-amd64-linux.so /apps/ngrinder-agent/lib

#### Check
> nGrinder - controller admin menu / Agent Management

## 9. Appendix

### reference site

* Installation Guide  
https://github.com/naver/ngrinder/wiki/Installation-Guide

+ nGrinder 시작하기  
https://nesoy.github.io/articles/2018-10/nGrinder-Start


