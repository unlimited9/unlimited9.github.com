# 10.install.n setup : h2database

>Created 월요일 04 12월 2017
h2database server setting

# Installation/Setup/Configuration

## 1. Pre-installation setup

### A. creating required operating system group and user

#### create the os application group(typically, app)  
$ groupadd -g 3000 app

> 어플리케이션 단위 관리 주체가 다르지 않고 권한을 나눌 필요가 없어 통합 계정을 생성하여 관리

#### create the software unified account (typically, app)
$ useradd -d /apps -g 3000 -m -u 3000 -s /bin/bash app
$ passwd app

### B. install dependency packages

### C. creating base directory

### D. firewall configuration

## 2. installation setup

### A. change account

$ su - app

### B. creating application directory

#### make directory
>$ mkdir -p /apps/nexus

$ mkdir -p /data/nexus

>$ mkdir -p /logs/nexus

### C. download

#### H2 Downloads
$ curl -O http://h2database.com/h2-2019-10-14.zip  
~~$ wget http://h2database.com/h2-2019-10-14.zip

### D. install

$ unzip h2-2019-10-14.zip

$ cp -R h2 /apps/h2/1.4.200


### E. configure

### F. run application

$ java -cp /apps/h2/1.4.200/bin/h2*.jar org.h2.tools.Server -?
```
Starts the H2 Console (web-) server, TCP, and PG server.
Usage: java org.h2.tools.Server <options>
When running without options, -tcp, -web, -browser and -pg are started.
Options are case sensitive. Supported options are:
[-help] or [-?]         Print the list of options
[-web]                  Start the web server with the H2 Console
[-webAllowOthers]       Allow other computers to connect - see below
[-webDaemon]            Use a daemon thread
[-webPort <port>]       The port (default: 8082)
[-webSSL]               Use encrypted (HTTPS) connections
[-browser]              Start a browser connecting to the web server
[-tcp]                  Start the TCP server
[-tcpAllowOthers]       Allow other computers to connect - see below
[-tcpDaemon]            Use a daemon thread
[-tcpPort <port>]       The port (default: 9092)
[-tcpSSL]               Use encrypted (SSL) connections
[-tcpPassword <pwd>]    The password for shutting down a TCP server
[-tcpShutdown "<url>"]  Stop the TCP server; example: tcp://localhost
[-tcpShutdownForce]     Do not wait until all connections are closed
[-pg]                   Start the PG server
[-pgAllowOthers]        Allow other computers to connect - see below
[-pgDaemon]             Use a daemon thread
[-pgPort <port>]        The port (default: 5435)
[-properties "<dir>"]   Server properties (default: ~, disable: null)
[-baseDir <dir>]        The base directory for H2 databases (all servers)
[-ifExists]             Only existing databases may be opened (all servers)
[-trace]                Print additional trace information (all servers)
[-key <from> <to>]      Allows to map a database name to another (all servers)
The options -xAllowOthers are potentially risky.
For details, see Advanced Topics / Protection against Remote Access.
See also http://h2database.com/javadoc/org/h2/tools/Server.html
```

## 3. post-installation setup

### A. create shell script

$ vi /apps/h2/startNetworkServer.sh
```
export H2_HOME="/apps/h2/1.4.200"
export H2_OPTS="-webDaemon -tcpDaemon -webAllowOthers -tcpAllowOthers -baseDir /data/database.h2/mobon -ifExists"

java -cp $H2_HOME/bin/h2*.jar org.h2.tools.Server $H2_OPTS &
```

### B. H2 console
http://172.20.0.103:8082/

#### Login
Saved Settings: Generic H2 (Server)
Driver Class: org.h2.Driver
JDBC URL : jdbc:h2:tcp://172.20.0.103/mobon;MODE=MYSQL;DATABASE_TO_UPPER=FALSE

## 9. Appendix

### reference site

#### H2 Tutorial
http://www.h2database.com/html/tutorial.html#using_server
