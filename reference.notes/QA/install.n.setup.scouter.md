## Scouter : server

### Requirement

#### scouter requires Java 1.8 over to run
>ref. [Install and setup JAVA](../AA/JDK/install.n.setup.md)

### Create directory
$ mkdir /apps/scouter

### Download
>https://github.com/scouter-project/scouter/releases

$ cd /apps/install  
$ wget https://github.com/scouter-project/scouter/releases/download/v2.7.0/scouter-all-2.7.0.tar.gz
>$ curl -O https://github.com/scouter-project/scouter/releases/download/v2.7.0/scouter-all-2.7.0.tar.gz

$ tar xvf scouter-all-2.7.0.tar.gz  
$ cp -R /apps/install/scouter /apps/scouter/2.7.0

### Configuration
$ vi /apps/scouter/2.7.0/server/conf/scouter.conf
```
# Agent Control and Service Port(Default : TCP 6100)
#net_tcp_listen_port=6100
# UDP Receive Port(Default : 6100)
#net_udp_listen_port=6100
# DB directory(Default : ./database)
#db_dir=./database
# Log directory(Default : ./logs)
#log_dir=./logs
```

### Run
$ cd /apps/scouter/2.7.0/server  
$ ./startup.sh

## Scouter : client

### Download
>https://github.com/scouter-project/scouter/releases

$ cd /apps/install  
$ wget https://github.com/scouter-project/scouter/releases/download/v2.7.0/scouter.client.product-linux.gtk.x86_64.tar.gz

$ tar xvf scouter.client.product-linux.gtk.x86_64.tar.gz

### Configuration
$ vi scouter.client/scouter.ini
```
-startup
plugins/org.eclipse.equinox.launcher_1.5.100.v20180827-1352.jar
--launcher.library
plugins/org.eclipse.equinox.launcher.gtk.linux.x86_64_1.1.800.v20180827-1352
-data
@user.home/.scouter
-vmargs
-Xms2048m
-Xmx2048m
-XX:+UseG1GC
-Dosgi.requiredJavaVersion=1.8
```

### Execute
$ scouter.client/scouter

### Login
>Server Address : 192.168.100.8:6100  
ID : admin  
Password : admin

## Scouter : agent

### create directory
$ mkdir /apps/scouter

### Download
>https://github.com/scouter-project/scouter/releases

$ cd /apps/install  
$ wget https://github.com/scouter-project/scouter/releases/download/v2.7.0/scouter-all-2.7.0.tar.gz
>$ curl -O https://github.com/scouter-project/scouter/releases/download/v2.7.0/scouter-all-2.7.0.tar.gz

$ tar xvf scouter-all-2.7.0.tar.gz
$ cp -R /apps/install/scouter /apps/scouter/2.7.0

> 기본 설정으로 포트 등이 설정되어있어 서버에 기본 설정을 변경하지 않았다면 설정에서 생략해도 된다.

### set host agent
- Configuration  
$ vi /apps/scouter/2.7.0/agent.host/conf/scouter.conf
	```
	### scouter host configruation sample
	net_collector_ip=192.168.100.8
	#net_collector_udp_port=6100
	#net_collector_tcp_port=6100
	#cpu_warning_pct=80
	#cpu_fatal_pct=85
	#cpu_check_period_ms=60000
	#cpu_fatal_history=3
	#cpu_alert_interval_ms=300000
	#disk_warning_pct=88
	#disk_fatal_pct=92
	```
- Run  
$ cd /apps/scouter/2.7.0/agent.host  
$ ./host.sh

### set java agent
>JVM Option보다 설정파일에 설정이 우선해서 적용되는 듯  
>여러 어플리케이션에서 같은 파일에 설정을 공유하고 obj_name 같은 설정만 JVM Option으로 다르게 적용

- Configuration  
  $ cp /apps/scouter/2.7.0/agent.java/conf/scouter.conf /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf  
  $ vi /apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
  ```
  ### scouter java agent configuration sample
  #obj_name=mdev-was-01-product
  # Scouter Server IP Address (Default : 127.0.0.1)
  net_collector_ip=192.168.100.8
  # Scouter Server Port (Default : 6100)
  #net_collector_udp_port=6100
  #net_collector_tcp_port=6100
  #hook_method_patterns=sample.mybiz.*Biz.*,sample.service.*Service.*
  trace_http_client_ip_header_key=X-Forwarded-For
  #profile_spring_controller_method_parameter_enabled=false
  #hook_exception_class_patterns=my.exception.TypedException
  #profile_fullstack_hooked_exception_enabled=true
  #hook_exception_handler_method_patterns=my.AbstractAPIController.fallbackHandler,my.ApiExceptionLoggingFilter.handleNotFoundErrorResponse
  #hook_exception_hanlder_exclude_class_patterns=exception.BizException

  ### HTTP header/parameter profile
  profile_http_parameter_enabled=true
  profile_http_header_enabled=true

  ### ignore low response time(ms)
  xlog_lower_bound_time_ms=300

  ### method profile
  #hook_method_patterns=com.xxx.controller*.*,com.xxx.service*.*,com.xxx.dao*.*
  #hook_method_ignore_classes=com.xxx.dto*.*
  ```

- Set JVM Options  
  모니터링 대상 Java Application 실행 시 JVM 옵션 지정  
  1. -javaagent:/apps/scouter/2.7.0/agent.java/scouter.agent.jar
  2. -Dscouter.config=/apps/scouter/2.7.0/agent.java/conf/scouter-product.conf
    (default : [directory of scouter.agent.jar]/conf/scouter.conf)
  3. -Dobj_name=mdev-was-01-product
    (default : java1 or tomcat1)

  $ vi /apps/tomcat/instances/mobon.product.01/bin/tomcat.sh
  ```
  ...
  #scouter.agent.java
  export SCOUTER_DIR=/apps/scouter/2.7.0  
  export JAVA_OPTS="$JAVA_OPTS -javaagent:$SCOUTER_DIR/agent.java/scouter.agent.jar"
  export JAVA_OPTS="$JAVA_OPTS -Dscouter.config=$SCOUTER_DIR/agent.java/conf/scouter-product.conf"
  export JAVA_OPTS="$JAVA_OPTS -Dobj_name=product-01"
  ...
  ```
  $ vi /apps/tomcat/instances/mobon.product.02/bin/tomcat.sh
  ```
  ...
  #scouter.agent.java
  export SCOUTER_DIR=/apps/scouter/2.7.0  
  export JAVA_OPTS="$JAVA_OPTS -javaagent:$SCOUTER_DIR/agent.java/scouter.agent.jar"
  export JAVA_OPTS="$JAVA_OPTS -Dscouter.config=$SCOUTER_DIR/agent.java/conf/scouter-product.conf"
  export JAVA_OPTS="$JAVA_OPTS -Dobj_name=product-02"
  ...
  ```

## 9. Appendix

### reference site

* scouter/scouter.document/main/Setup.md  
https://github.com/scouter-project/scouter/blob/master/scouter.document/main/Setup.md

+ [Scouter] 오픈소스 APM Scouter 설치 및 연동 가이드  
https://waspro.tistory.com/409

+ Scouter APM 소소한 시리즈 #1 - 설치하기  
http://gunsdevlog.blogspot.com/2017/07/scouter-apm-1.html

+ 오픈소스 성능 모니터링 도구 Scouter 설정하기  
https://www.popit.kr/scouter-open-source-apm-config/


- Scouter Plugin Guide  
https://github.com/scouter-project/scouter/blob/master/scouter.document/main/Plugin-Guide_kr.md
