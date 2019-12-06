# Install and setup : Haproxy

>Created 목요일 30 11월 2017  
Installation/Setup/Configuration Server.setting - proxy

## 1. Pre-installation setup

#### A. creating required operating system group and user
[Create operating system group and user](../system/management.account.n.group.md)


#### B. install dependency packages
`dependency libray`  
* case of redhat  
$ yum install pcre pcre-devel openssl libssl-dev  
* case of debian  
$ apt-get install libpcre3 libpcre3-dev openssl libssl-dev

`utility libray`  
* case of redhat  
$ yum install zlib*  
* case of debian  
$ apt-get install zlib1g zlib1g-dev

#### C. creating base directory
[Create operating system drectory](../system/management.directory.md)

#### D. firewall configuration

$ vi /etc/sysconfig/iptables
```
...  
## nginx : 3020  
# http  
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT  
# https  
-A INPUT -m state --state NEW -m tcp -p tcp --dport 443 -j ACCEPT
...
```

`restart iptalbes service`  
$ service iptables restart

$ iptables -nL

## 2. installation setup : app

#### A. change account
>$ su - app

#### B. creating application directory
$ mkdir -p /apps/haproxy  
$ mkdir -p /data/haproxy  
$ mkdir -p /logs/haproxy

#### C. download

`Nginx(http://nginx.org/)`  
$ curl -O http://www.haproxy.org/download/2.0/src/haproxy-2.0.10.tar.gz -P /apps/install  
~~$ wget http://www.haproxy.org/download/2.0/src/haproxy-2.0.10.tar.gz -P /apps/install~~

#### D. install

#### decompress tarball

$ sudo yum -y install make gcc perl pcre-devel zlib-devel openssl-devel
> $ sudo yum -y install make gcc perl pcre-devel zlib-devel openssl-devel --downloadonly --downloaddir=/apps/install/rpms/haproxy

$ tar -zxvf /apps/install/haproxy-2.0.10.tar.gz  

#### compile and install
$ cd /apps/install/haproxy-2.0.10

$ make TARGET=linux-glibc USE_PCRE=1 USE_OPENSSL=1 USE_ZLIB=1  
>$ make TARGET=linux2628 USE_PCRE=1 USE_OPENSSL=1 USE_ZLIB=1  
>$ make TARGET=linux2628 USE_PCRE=1 USE_OPENSSL=1 USE_ZLIB=1 USE_CRYPT_H=1 USE_LIBCRYPT=1

$ make PREFIX=/apps/haproxy/2.0.10 install

## 3. post-installation setup

#### create systemd unit file
$ sudo vi /etc/systemd/system/haproxy.service
```
[Unit]
Description=HAProxy
After=syslog.target network.target

[Service]
Type=notify
EnvironmentFile=/etc/sysconfig/haproxy
ExecStart=/apps/haproxy/2.0.10/sbin/haproxy -f $CONFIG_FILE -p $PID_FILE $CLI_OPTIONS
ExecReload=/bin/kill -USR2 $MAINPID
ExecStop=/bin/kill -USR1 $MAINPID

[Install]
WantedBy=multi-user.target
```

#### create the systemd environment file
$ sudo vi /etc/sysconfig/haproxy
```
# Command line options to pass to HAProxy at startup
# The default is:  
#CLI_OPTIONS="-Ws" #be able to notify systemd when it is done starting
CLI_OPTIONS="-Ws"

# Specify an alternate configuration file. The default is:
#CONFIG_FILE=/etc/haproxy/haproxy.conf
CONFIG_FILE=/etc/haproxy/haproxy.conf

# File used to track process IDs. The default is:
#PID_FILE=/var/run/haproxy.pid
PID_FILE=/var/run/haproxy.pid
```

#### reload the system configuration
$ systemctl daemon-reload

#### configure
$ sudo mkdir /etc/haproxy

$ sudo vi /etc/haproxy/haproxy.conf
```
global
    log         127.0.0.1 local2 info
 
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    stats socket           /var/run/haproxy.sock mode 666 level admin
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon
 
    stats socket /var/lib/haproxy/stats
 
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option                  http-server-close
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000
 
listen stats
    bind :::8888 v4v6
    mode http
    stats enable
    stats hide-version
    stats uri /
    stats realm Haproxy\ Statistics
    stats auth user:password
 
frontend  main
    bind :::80 v4v6
    option                      http-server-close
    acl host_home1 hdr(host) -i home1.wb.com
    acl host_home2 hdr(host) -i home2.wb.com
    use_backend backend_1 if host_home1
    use_backend backend_2 if host_home2
    default_backend             default
 
backend default
    balance     roundrobin
    server  server1 192.168.56.204:8001 check
    server  server2 192.168.56.205:8001 check
 
backend backend_1
    balance roundrobin
    server  server1 192.168.56.204:8002 check
    server  server2 192.168.56.205:8002 check
 
backend backend_2
    balance roundrobin
    server  server1 192.168.56.204:8003 check
    server  server2 192.168.56.205:8003 check
```

## 4. execution(starting and stopping daemon services)

#### start service
$ systemctl start haproxy.service

#### configure haproxy to start at boot
$ systemctl enable haproxy

#### check process
$ ps -ef | grep haproxy | cut -c 1-100

## 8. trouble-shooting

## 9. Appendix

#### reference site

* HAProxy - The Reliable, High Performance TCP/HTTP Load Balancer  
http://www.haproxy.org/

+ HAProxy-1.9.8 설치 (on CentOS 7)    
https://hbesthee.tistory.com/1622

+ Installing HAProxy From Source On CentOS 7  
https://tylersguides.com/guides/installing-haproxy-from-source-on-centos-7/

+ HAProxy 설치  
https://sarc.io/index.php/miscellaneous/1487-haproxy
