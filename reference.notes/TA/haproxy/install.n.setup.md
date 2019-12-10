# Install and setup : Haproxy

>Created 목요일 30 11월 2017  
Installation/Setup/Configuration Server.setting - proxy

## 1. Pre-installation setup

#### A. creating required operating system group and user
[Create operating system group and user](../system/management.account.n.group.md)


#### B. install dependency packages
`dependency libray`  
* case of redhat  
$ sudo yum -y install make gcc perl pcre-devel zlib-devel openssl-devel
> $ sudo yum -y install make gcc perl pcre-devel zlib-devel openssl-devel --downloadonly --downloaddir=/apps/install/rpms/haproxy

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
$ tar -zxvf /apps/install/haproxy-2.0.10.tar.gz  

#### compile and install
$ cd /apps/install/haproxy-2.0.10

$ make TARGET=linux-glibc USE_PCRE=1 USE_OPENSSL=1 USE_ZLIB=1 USE_SYSTEMD=1
>$ make TARGET=linux-glibc USE_PCRE=1 USE_OPENSSL=1 USE_ZLIB=1  
>$ make TARGET=linux2628 USE_PCRE=1 USE_OPENSSL=1 USE_ZLIB=1  
>$ make TARGET=linux2628 USE_PCRE=1 USE_OPENSSL=1 USE_ZLIB=1 USE_CRYPT_H=1 USE_LIBCRYPT=1

> `Error Occurred`  
> [ALERT] 339/174150 (17206) : master-worker mode with systemd support (-Ws) requested, but not compiled. Use master-worker mode (-W) if you are not using Type=notify in your unit file or recompile with USE_SYSTEMD=1.  
>
>`RedHat/CentOS`  
>$ yum install systemd-devel  
>`Debian/Ubuntu`  
>$ apt-get install libsystemd-dev


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
> The USR2 signal instructs HAProxy to reload its configuration without bringing it down.
> USR1 brings down HAProxy, allowing processes to finish what they were doing before exiting.

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
    maxconn     4000
    ulimit-n    16384
    log         127.0.0.1 local0 info
    user        app
    group       app
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    stats socket           /var/run/haproxy.sock mode 666 level admin
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
    stats auth admin:P@ssw0rd

frontend kubernetes
    bind 10.251.0.194:6443
    option tcplog
    mode tcp
    default_backend kubernetes-master-nodes

backend kubernetes-master-nodes
    mode tcp
    balance roundrobin
    option tcp-check
    server k8s-master-0 10.251.0.191:6443 check fall 3 rise 2
    server k8s-master-1 10.251.0.192:6443 check fall 3 rise 2
    server k8s-master-2 10.251.0.193:6443 check fall 3 rise 2

#frontend main
#    bind :::80 v4v6
#    option                      http-server-close
#    acl api.k8s.mobon.platform.01 hdr(host) -i MPK-Cluster-api-01
#    acl api.k8s.mobon.platform.02 hdr(host) -i MPK-Cluster-api-02
#    use_backend api.k8s.01 if api.k8s.mobon.platform.01
#    use_backend api.k8s.02 if api.k8s.mobon.platform.02
#    default_backend             default
#
#backend default
#    balance     roundrobin
#    server  MPK-Cluster-09 10.251.0.191:6443 check
#    server  MPK-Cluster-10 10.251.0.192:6443 check
#    server  MPK-Cluster-11 10.251.0.193:6443 check
#
#backend api.k8s.01
#    balance roundrobin
#    server  MPK-Cluster-09 10.251.0.191:6443 check
#    server  MPK-Cluster-10 10.251.0.192:6443 check
#    server  MPK-Cluster-11 10.251.0.193:6443 check
#
#backend api.k8s.02
#    balance roundrobin
#    server  MPK-Cluster-09 10.251.0.191:6443 check
#    server  MPK-Cluster-10 10.251.0.192:6443 check
#    server  MPK-Cluster-11 10.251.0.193:6443 check

```

## 4. execution(starting and stopping daemon services)

#### start service
$ systemctl start haproxy.service

> `Error Occurred`
>```
>Job for haproxy.service failed because the control process exited with error code. See "systemctl status haproxy.service" and "journalctl -xe" for details.
>```
>$ systemctl status haproxy.service
>```
>● haproxy.service - HAProxy
>   Loaded: loaded (/etc/systemd/system/haproxy.service; disabled; vendor preset: disabled)
>   Active: failed (Result: exit-code) since 금 2019-12-06 17:47:02 KST; 11s ago
>  Process: 19310 ExecStop=/bin/kill -USR1 $MAINPID (code=exited, status=1/FAILURE)
>  Process: 19308 ExecStart=/apps/haproxy/2.0.10/sbin/haproxy -f $CONFIG_FILE -p $PID_FILE $CLI_OPTIONS (code=exited, status=1/FAILURE)
> Main PID: 19308 (code=exited, status=1/FAILURE)
> ...
>``` 
> $ journalctl -xe
>```
>...
>[ALERT] 339/174702 (19308) : Starting frontend GLOBAL: cannot bind UNIX socket [/var/lib/haproxy/stats]
>...
>```

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
