# Tomcat Management

>Created 월요일 20 11월 2017
operation and maintenance

## Extended function

#### deployment shell script

`deploy.sh : latest`  
$ vi /pgms/tomcat/deploy.sh
```
#!/bin/bash

INSTANCE_PATH="/apps/tomcat/instances"
APPLICATION_PATH="/pgms/tomcat/service/$1"

BACKUP_PATH="$APPLICATION_PATH/backup"
WARS_PATH="$APPLICATION_PATH/wars"
APPS_PATH="$APPLICATION_PATH/webapps/ROOT"

usage() {
	echo "****************************************"
	echo "Usage: $0 [module] [action] [object]"
	echo "****************************************"
	echo "module : project name"
	echo "action : deploy/backup ..."
	echo "object : resource(filename/path)"
	echo "****************************************"
}

getArchiveFilename() {
	if [ -z $1 ]
	then
		#echo `find WARS_PATH -type f -ctime -$1 -name "*.war" | tail -1`
		echo `find $WARS_PATH -type f -name "*.war" | sort | tail -1`
	else
		echo $WARS_PATH/$1
	fi
}

getRestoreDirectory() {
	if [ -z $1 ]
	then
		echo `find $BACKUP_PATH -depth -mindepth 1 -maxdepth 1 -type d -name "*" | sort | tail -1`
	else
		echo $BACKUP_PATH/$1
	fi
}

backup() {
	currDateTime=$(date +"%Y%m%d%H%M%S") #$(date +"%Y-%m-%d %H:%M:%S")

	cp -R $APPS_PATH $BACKUP_PATH/$currDateTime
	echo "Backup at $BACKUP_PATH/$currDateTime"
}

ret=0
case $2 in
	start | stop | restart)
		if [ "$#" -ne 3 ]
		then
			for instance_path in $INSTANCE_PATH/$1*; do
				echo "\"$(basename $instance_path)\" instance $2..."
				$instance_path/bin/tomcat.sh $2
				echo "complete..."
			done
		else
			instance_path=$INSTANCE_PATH/$1.$3
			echo "\"$(basename $instance_path)\" instance $2..."
			$instance_path/bin/tomcat.sh $2
			echo "complete..."
		fi
		;;
	deploy)
		archiveFilename=$(getArchiveFilename $3)

		echo "Deploy WAR.Filename : $archiveFilename"
		if [ -f $archiveFilename ];
		then
			rm -fr $APPS_PATH
			unzip $archiveFilename -d $APPS_PATH
		else
			echo 'Error : File Not Found'
			exit 0
		fi
		;;
	backup)
		backup
		;;
	restore)
		restoreDirectory=$(getRestoreDirectory $3)

		echo "Restore Directory : $restoreDirectory"
		if [ -d $restoreDirectory ];
		then
			rm -fr $APPS_PATH
			cp -R $restoreDirectory $APPS_PATH
			exit 0
		else
			echo 'Error : Directory Not Found'
			exit 0
		fi
		;;
	*)
		usage
		ret=1
esac

exit $ret
```

<details>
<summary>ctrl.sh : last</summary>
<div markdown="1">

>$ vi /pgms/tomcat/ctrl.sh
>```
>#!/bin/bash
>
>INSTANCE_PATH="/apps/tomcat/instances"
>
>BACKUP_PATH="/pgms/tomcat/backup/$1"
>WARS_PATH="/pgms/tomcat/wars/$1"
>APPS_PATH="/pgms/tomcat/webapps/$1"
>
>usage() {
>	echo "****************************************"
>	echo "Usage: $0 [module] [action] [object]"
>	echo "****************************************"
>	echo "module : project name"
>	echo "action : deploy/backup ..."
>	echo "object : resource(filename/path)"
>	echo "****************************************"
>}
>
>getArchiveFilename() {
>	if [ -z $1 ]
>	then
>		#echo `find WARS_PATH -type f -ctime -$1 -name "*.war" | tail -1`
>		echo `find $WARS_PATH -type f -name "*.war" | sort | tail -1`
>	else
>		echo $WARS_PATH/$1
>	fi
>}
>
>getRestoreDirectory() {
>	if [ -z $1 ]
>	then
>		echo `find $BACKUP_PATH -depth -mindepth 1 -maxdepth 1 -type d -name "*" | sort | tail -1`
>	else
>		echo $BACKUP_PATH/$1
>	fi
>}
>
>backup() {
>	currDateTime=$(date +"%Y%m%d%H%M%S") #$(date +"%Y-%m-%d %H:%M:%S")
>
>	cp -R $APPS_PATH $BACKUP_PATH/$currDateTime
>	echo "Backup at $BACKUP_PATH/$currDateTime"
>}
>
>ret=0
>case $2 in
>	start | stop | restart)
>		if [ "$#" -ne 3 ]
>		then
>			for instance_path in $INSTANCE_PATH/$1*; do
>				echo "\"$(basename $instance_path)\" instance $2..."
>				$instance_path/bin/tomcat.sh $2
>				echo "complete..."
>			done
>		else
>			instance_path=$INSTANCE_PATH/$1.$3
>			echo "\"$(basename $instance_path)\" instance $2..."
>			$instance_path/bin/tomcat.sh $2
>			echo "complete..."
>		fi
>		;;
>	deploy)
>		archiveFilename=$(getArchiveFilename $3)
>
>		echo "Deploy WAR.Filename : $archiveFilename"
>		if [ -f $archiveFilename ];
>		then
>			rm -fr $APPS_PATH
>			unzip $archiveFilename -d $APPS_PATH
>		else
>			echo 'Error : File Not Found'
>			exit 0
>		fi
>		;;
>	backup)
>		backup
>		;;
>	restore)
>		restoreDirectory=$(getRestoreDirectory $3)
>
>		echo "Restore Directory : $restoreDirectory"
>		if [ -d $restoreDirectory ];
>		then
>			rm -fr $APPS_PATH
>			cp -R $restoreDirectory $APPS_PATH
>			exit 0
>		else
>			echo 'Error : Directory Not Found'
>			exit 0
>		fi
>		;;
>	*)
>		usage
>		ret=1
>esac
>
>exit $ret
>```

</div>
</details>

#### HTTP Header Security Filter

Refused to display 'http://domain/uri' in a frame because it set 'X-Frame-Options' to 'deny'.

$ vi $CATALINA_BASE/conf/web.xml
```
...
	<!-- A filter that sets various security related HTTP Response headers. -->  
	<!-- This filter supports the following initialization parameters -->  
	<filter>
		<filter-name>httpHeaderSecurity</filter-name>  
		<filter-class>org.apache.catalina.filters.HttpHeaderSecurityFilter</filter-class>  
		<async-supported>true</async-supported>  
		<init-param>
			<param-name>antiClickJackingOption</param-name>  
			<param-value>SAMEORIGIN</param-value>
		</init-param>
	</filter>
	...  
	<!-- The mapping for the HTTP header security Filter -->  
	<filter-mapping>
		<filter-name>httpHeaderSecurity</filter-name>  
		<url-pattern>/*</url-pattern>  
		<dispatcher>REQUEST</dispatcher>
	</filter-mapping>
...
```

#### JNDI Resource

$ vi $CATALINA_BASE/conf/server.xml  
```
	...
	<Resource name="dreamdb2" auth="Container"
		factory="com.zaxxer.hikari.HikariJNDIFactory" type="javax.sql.DataSource"
		dataSourceClassName="com.mysql.jdbc.jdbc2.optional.MysqlDataSource"
		dataSource.url="jdbc:mysql://192.168.2.190,192.168.2.127:3306/dreamsearch?autoReconnect=true"  
		dataSource.user="dreamsearch"  
		dataSource.password="password"  
		dataSource.cachePrepStmts="true"  
		dataSource.prepStmtCacheSize="250"  
		dataSource.prepStmtCacheSqlLimit="2048"  
		dataSource.logger="com.mysql.jdbc.log.StandardLogger"  
		dataSource.logSlowQueries="false"  
		dataSource.dumpQueriesOnException="false"  
		poolName="dreamdb2"  
		minimumIdle="30"  
		maximumPoolSize="30"  
		maxLifetime="0"  
		connectionTimeout="3000"  
		validationQuery="SELECT 1"  
		validationInterval="240000"  
		testWhileIdle="true"  
		registerMbeans="true" />
	...
```

## Migration Guide Tomcat 8 - Apache Tomcat

#### 1. How can I make Tomcat accept unescaped brackets in URLs

`error message`  
java.lang.IllegalArgumentException: Invalid character found in the request target. The valid characters are defined in RFC 7230 and RFC 3986

보안상의 이유로 Tomcat 7.0.73, 8.0.39, 8.5.7 버전 부터는 호출 주소에 특수문자가 들어갈 시 차단  
RFC 7230 : Hypertext Transfer Protocol (HTTP/1.1): Message Syntax and Routing  
RFC 3986 : Uniform Resource Identifier (URI): Generic Syntax

`solution`  
1. Allow for changes to HTTP request validation
$ vi CATALINA_BASE/conf/catalina.properties  
```
...  
# Allow for changes to HTTP request validation  
# WARNING: Using this option will expose the server to CVE-2016-6816  
tomcat.util.http.parser.HttpParser.requestTargetAllow=|{}
```
2. charater encoding  
encodeURI() : URI에 대한 예약 문자들(A-Z a-z 0-9 ; , / ? : @ & = + $ - _ . ! ~ * ' ( ) #)은 인코딩하지 않는다.  
encodeURIComponent() : 알파벳, 0~9의 숫자, - _ . ! ~ * ' ( )를 제외한 모든 문자를 이스케이프 처리한다.  
escape(str) : The escape function is a property of the global object. Special characters are encoded with the exception of: @*_+-./

3. tomcat down grade..

`reference`  
https://tomcat.apache.org/tomcat-8.5-doc/config/systemprops.html
https://tools.ietf.org/html/rfc3986#section-2

ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789-._~:/?#[]@!$&'()*+,;=`.

#### 2. How to change Cookie Processor to LegacyCookieProcessor in tomcat 8

`error message`  
Caused by: java.lang.IllegalArgumentException: An invalid domain [.dreamsearch.or.kr] was specified for this cookie  
....

`solution`  
$ vi CATALINA_BASE/conf/context.xml  
```
<Context>  
...  

<CookieProcessor className="org.apache.tomcat.util.http.LegacyCookieProcessor" />

</Context>
```

`reference`  
https://tomcat.apache.org/tomcat-8.0-doc/config/cookie-processor.html

#### 3. Nginx + Tomcat 연동 시 request.getHeader("remote_addr")에서 nginx 서버 IP 리턴이 될 경우

`situation`  
Tomcat port of mod_remoteip, this valve replaces the apparent client remote IP address and hostname for the request with the IP address list presented by a proxy or a load balancer via a request headers (e.g. "X-Forwarded-For").

nginx 와 tomcat 을 이용하여 웹 서비스 구축 시 nginx 로 들어온 요청을 reverse proxy형태로 tomcat에 요청 할 경우 client remote ip의 ip가 넘어오는게 아니라 nginx의 remote ip 가 넘어오는 현상

```
HTTP HTTP ( reverse proxy )

Client(192.168.0.200) -----> Nginx(192.168.0.100) -----> Tomcat(192.168.0.150)
```

위의 구조일 경우 Tomcat에서 request.getHeader("remote_addr") 로 불러오거나 access log에 Client의 IP ( 192.168.0.200 ) 을 가져오는 것이 아닌 NginX ( 192.168.0.100 ) 이 오기때문에 모든 요청이 Nginx의 IP를 찍게 되는 현상

`solution`  
- nginx  
  $ vi $NGINX_HOME/conf/
  ```
  server {  
    ...  
    location / {
      proxy_set_header Host $http_host;  
      proxy_set_header X-Real-IP $remote_addr;  
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
      proxy_set_header X-Forwarded-Proto $scheme;  
      proxy_set_header X-NginX-Proxy true;  
      ...
    }
  }
  ```

- tomcat  
$ vi $CATALINA_BASE/conf/server.xml  
  ```
  ...  
  <Host name="${tomcat.instance.hostname}" appBase="webapps" unpackWARs="true" autoDeploy="true">  
    ...
    <Valve className="org.apache.catalina.valves.RemoteIpValve" />

    <!-- Access log processes all example.
    Documentation at: /docs/config/valve.html
    Note: The pattern used is equivalent to using pattern="common" -->

    <Valve className="org.apache.catalina.valves.AccessLogValve" directory="/logs/tomcat/${tomcat.instance.name}"
      prefix="${tomcat.instance.hostname}_access" suffix=".log"  
      pattern="%{x-forwarded-for}i %h %l %u %t &quot;%r&quot; %s %b" />
  </Host>  
  ...
  ```

`reference`  
https://tomcat.apache.org/tomcat-8.0-doc/config/cookie-processor.html

#### 4. insufficient free space cache memory

`error message`  
24-Apr-2018 09:51:02.387 경고 [www.dreamsearch.or.kr-startStop-1] org.apache.catalina.webresources.Cache.getResource Unable to add the resource at [/WEB-INF/classes/] to the cache because there was insufficient free space available after evicting expired cache entries - consider increasing the maximum size of the cache

`solution`  
$ vi CATALINA_BASE/conf/context.xml  
```
<Context>
	...
	<Resources cachingAllowed="true" cacheMaxSize="100000" />
</Context>
```

## Configuration Ex

#### tomcat configuration : server.xml
$ vi $CATALINA_BASE/conf/server.xml  
```
<?xml version="1.0" encoding="UTF-8"?>
<!--
  Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<!-- Note:  A "Server" is not itself a "Container", so you may not
     define subcomponents such as "Valves" at this level.
     Documentation at /docs/config/server.html
 -->
<Server port="${tomcat.port.shutdown}" shutdown="SHUTDOWN">
  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
  <!-- Security listener. Documentation at /docs/config/listeners.html
  <Listener className="org.apache.catalina.security.SecurityListener" />
  -->
  <!--APR library loader. Documentation at /docs/apr.html -->
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
  <!-- Prevent memory leaks due to use of particular java/javax APIs-->
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />

  <!-- Global JNDI resources
       Documentation at /docs/jndi-resources-howto.html
  -->
  <GlobalNamingResources>
    <!-- Editable user database that can also be used by
         UserDatabaseRealm to authenticate users
    -->
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
  </GlobalNamingResources>

  <!-- A "Service" is a collection of one or more "Connectors" that share
       a single "Container" Note:  A "Service" is not itself a "Container",
       so you may not define subcomponents such as "Valves" at this level.
       Documentation at /docs/config/service.html
   -->
  <Service name="Catalina">

    <!--The connectors can use a shared executor, you can define one or more named thread pools-->
    <!--
    <Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
        maxThreads="150" minSpareThreads="4"/>
    -->


    <!-- A "Connector" represents an endpoint by which requests are received
         and responses are returned. Documentation at :
         Java HTTP Connector: /docs/config/http.html
         Java AJP  Connector: /docs/config/ajp.html
         APR (HTTP/AJP) Connector: /docs/apr.html
         Define a non-SSL/TLS HTTP/1.1 Connector on port ${tomcat.port.http}
    -->
    <Connector executor="tomcatThreadPool"
               port="${tomcat.port.http}" protocol="HTTP/1.1"
               connectionTimeout="20000"
               maxThreads="5000" minSpareThreads="100" acceptCount="10"
    />

    <!--Connector port="${tomcat.port.http}"
            protocol="org.apache.coyote.http11.Http11NioProtocol" enableLookups="false" tcpNoDelay="true"  compression="off" 
            maxThreads="5000" minSpareThreads="50" acceptCount="10" connectionTimeout="8000"
            maxKeepAliveRequests="-1" maxHttpHeaderSize="40960" /-->

    <!--Connector port="${tomcat.port.https}" keystoreFile="/usr/local/tomcat7/conf/SSL_mediacategory_2018/STAR.mediacategory.com.keystore"
               keystorePass="Dlsfkdlvmf2017" protocol="org.apache.coyote.http11.Http11NioProtocol"
               maxThreads="4000" SSLEnabled="true" scheme="https" secure="true"
               clientAuth="false" connectionTimeout="8000" maxHttpHeaderSize="40960" /-->

    <!-- A "Connector" using the shared thread pool-->
    <!--
    <Connector executor="tomcatThreadPool"
               port="${tomcat.port.http}" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="${tomcat.port.https}" />
    -->
    <!-- Define a SSL/TLS HTTP/1.1 Connector on port ${tomcat.port.https}
         This connector uses the NIO implementation. The default
         SSLImplementation will depend on the presence of the APR/native
         library and the useOpenSSL attribute of the
         AprLifecycleListener.
         Either JSSE or OpenSSL style configuration may be used regardless of
         the SSLImplementation selected. JSSE style configuration is used below.
    -->
    <!--
    <Connector port="${tomcat.port.https}" protocol="org.apache.coyote.http11.Http11NioProtocol"
               maxThreads="150" SSLEnabled="true">
        <SSLHostConfig>
            <Certificate certificateKeystoreFile="conf/localhost-rsa.jks"
                         type="RSA" />
        </SSLHostConfig>
    </Connector>
    -->
    <!-- Define a SSL/TLS HTTP/1.1 Connector on port ${tomcat.port.https} with HTTP/2
         This connector uses the APR/native implementation which always uses
         OpenSSL for TLS.
         Either JSSE or OpenSSL style configuration may be used. OpenSSL style
         configuration is used below.
    -->
    <!--
    <Connector port="${tomcat.port.https}" protocol="org.apache.coyote.http11.Http11AprProtocol"
               maxThreads="150" SSLEnabled="true" >
        <UpgradeProtocol className="org.apache.coyote.http2.Http2Protocol" />
        <SSLHostConfig>
            <Certificate certificateKeyFile="conf/localhost-rsa-key.pem"
                         certificateFile="conf/localhost-rsa-cert.pem"
                         certificateChainFile="conf/localhost-rsa-chain.pem"
                         type="RSA" />
        </SSLHostConfig>
    </Connector>
    -->

    <!-- Define an AJP 1.3 Connector on port ${tomcat.port.ajp} -->
    <!--Connector port="${tomcat.port.ajp}" protocol="AJP/1.3" redirectPort="${tomcat.port.https}" /-->


    <!-- An Engine represents the entry point (within Catalina) that processes
         every request.  The Engine implementation for Tomcat stand alone
         analyzes the HTTP headers included with the request, and passes them
         on to the appropriate Host (virtual host).
         Documentation at /docs/config/engine.html -->

    <!-- You should set jvmRoute to support load-balancing via AJP ie :
    <Engine name="Catalina" defaultHost="localhost" jvmRoute="jvm1">
    -->
    <Engine name="Catalina" defaultHost="localhost">

      <!--For clustering, please take a look at documentation at:
          /docs/cluster-howto.html  (simple how to)
          /docs/config/cluster.html (reference documentation) -->
      <!--
      <Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"/>
      -->

      <!-- Use the LockOutRealm to prevent attempts to guess user passwords
           via a brute-force attack -->
      <Realm className="org.apache.catalina.realm.LockOutRealm">
        <!-- This Realm uses the UserDatabase configured in the global JNDI
             resources under the key "UserDatabase".  Any edits
             that are performed against this UserDatabase are immediately
             available for use by the Realm.  -->
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
      </Realm>

      <Host name="localhost"  appBase="/pgms/tomcat/service/mobon.product/webapps"
            unpackWARs="true" autoDeploy="true">

        <!--Context path="/product" docBase="mobon.product" reloadable="false" /-->

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
    </Engine>
  </Service>
</Server>
```

## 9. Appendix

#### reference site
* Tomcat 무분별하게 catalina.out 크기 커지는것 막기  
https://enterkey.tistory.com/396

* 리눅스 로그파일 관리(logrotate)  
https://www.linux.co.kr/linux/logrotate/page04.htm9. Appendix


