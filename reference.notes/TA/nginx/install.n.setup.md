# 01.install.n setup : Nginx

>Created 목요일 30 11월 2017  
Installation/Setup/Configuration Server.setting - proxy

## 1. Pre-installation setup

#### A. creating required operating system group and user
[Create operating system group and user](/reference.notes/TA/system/create.account.n.group.md)


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

#### 디렉토리 생성
[Create operating system drectory](/reference.notes/TA/system/create.directory.md)

### D. firewall configuration

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

#### restart iptalbes service 
$ service iptables restart

$ iptables -nL

## 2. installation setup : app

### A. change account

$ su - app

### B. creating application directory

#### make directory
$ mkdir -p /apps/nginx  
$ mkdir -p /data/nginx  
$ mkdir -p /logs/nginx

### C. download

#### Nginx(http://nginx.org/)  

$ curl -O http://nginx.org/download/nginx-1.14.2.tar.gz -P /apps/install  
~~$ wget http://nginx.org/download/nginx-1.14.2.tar.gz -P /apps/install~~

### D. install

#### decompress tarball
$ tar -zxvf /apps/install/nginx-1.14.2.tar.gz

$ cd /apps/install/nginx-1.14.2

#### configure 
```bash
$ ./configure --prefix=/apps/nginx/1.14.2 --user=app --group=app \
--with-pcre --with-http_ssl_module --with-http_realip_module --with-http_stub_status_module --with-debug 
```
~~$ ./configure --prefix=/apps/nginx/1.14.2 --user=nginx --group=app \
--with-pcre --with-http_ssl_module --with-http_realip_module --with-http_stub_status_module --with-debug~~
```
Configuration summary
  + using system PCRE library
  + using system OpenSSL library
  + using system zlib library

  nginx path prefix: "/apps/nginx/1.14.2"
  nginx binary file: "/apps/nginx/1.14.2/sbin/nginx"
  nginx modules path: "/apps/nginx/1.14.2/modules"
  nginx configuration prefix: "/apps/nginx/1.14.2/conf"
  nginx configuration file: "/apps/nginx/1.14.2/conf/nginx.conf"
  nginx pid file: "/apps/nginx/1.14.2/logs/nginx.pid"
  nginx error log file: "/apps/nginx/1.14.2/logs/error.log"
  nginx http access log file: "/apps/nginx/1.14.2/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"
```
#### configuration error  
./configure: error: SSL modules require the OpenSSL library.  
You can either do not enable the modules, or install the OpenSSL library  
into the system, or build the OpenSSL library statically from the source  
with nginx by using --with-openssl=<path> option.

##### Debian/Ubuntu  
$ aptitude install libssl-dev  
##### RedHat/CentOS  
$ yum install openssl-devel.x86_64

>#### ref.  
>--conf-path=/etc/nginx/nginx.conf \  
--sbin-path=/usr/sbin \  
--http-log-path=/logs/nginx/access.log \  
--error-log-path=/logs/nginx/error.log \  
--pid-path=/var/run/nginx.pid \  
--lock-path=/var/lock/nginx.lock \  
--http-client-body-temp-path=/var/lib/nginx/body \  
--http-proxy-temp-path=/var/lib/nginx/proxy \  
--http-fastcgi-temp-path=/var/tmp/nginx/fastcgi_temp \  
--http-uwsgi-temp-path=/var/tmp/nginx/fastcgi_temp \  
--http-scgi-temp-path=/var/tmp/nginx/fastcgi_temp \
>
>--with-zlib=../zlib-1.2.8 --with-pcre=../pcre-8.32 --with-openssl=../openssl-1.0.1e  
--add-module=../echo-nginx-module-0.45

##### sample.  
```bash
./configure  
--sbin-path=/usr/sbin/nginx  
--conf-path=/usr/local/nginx/nginx.conf  
--pid-path=/run/nginx.pid  
--with-openssl=../openssl-1.0.2j  
--with-http_ssl_module  
--with-http_realip_module  
--with-http_addition_module  
--with-http_sub_module  
--with-http_dav_module  
--with-http_flv_module  
--with-http_mp4_module  
--with-http_gunzip_module  
--with-http_gzip_static_module  
--with-http_random_index_module  
--with-http_secure_link_module  
--with-http_stub_status_module  
--with-http_auth_request_module  
--with-http_xslt_module=dynamic  
--with-http_image_filter_module=dynamic  
--with-http_geoip_module=dynamic  
--with-http_perl_module=dynamic  
--with-threads  
--with-stream  
--with-stream_ssl_module  
--with-stream_realip_module  
--with-stream_geoip_module=dynamic  
--with-http_slice_module  
--with-mail  
--with-mail_ssl_module  
--with-file-aio  
--with-http_v2_module  
--add-module=../nginx-ct-master --add-module=../headers-more-nginx-module --add-module=../ngx_brotli
```

#### compile and install  
$ make

$ make install

$ ls /apps/nginx/1.14.2/  
>conf : 설정파일  
html : 기본 document_root  
logs : 로그 파일  
sbin : nginx 실행파일

## 3. post-installation setup

### A. create shell script

#### nginx
$ vi /apps/nginx/nginx  
> ref. https://www.nginx.com/resources/wiki/start/topics/examples/redhatnginxinit/

```
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15
# description:  NGINX is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /etc/nginx/nginx.conf
# config:      /etc/sysconfig/nginx
# pidfile:     /var/run/nginx.pid

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0

#nginx="/usr/sbin/nginx"
nginx="/apps/nginx/1.14.2/sbin/nginx"  
prog=$(basename $nginx)

#NGINX_CONF_FILE="/etc/nginx/nginx.conf"
NGINX_CONF_FILE="/apps/nginx/1.14.2/conf/nginx.conf"  

[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx

lockfile=/var/lock/subsys/nginx

make_dirs() {
   # make required directories
   user=`$nginx -V 2>&1 | grep "configure arguments:.*--user=" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
   if [ -n "$user" ]; then
      if [ -z "`grep $user /etc/passwd`" ]; then
         useradd -M -s /bin/nologin $user
      fi
      options=`$nginx -V 2>&1 | grep 'configure arguments:'`
      for opt in $options; do
          if [ `echo $opt | grep '.*-temp-path'` ]; then
              value=`echo $opt | cut -d "=" -f 2`
              if [ ! -d "$value" ]; then
                  # echo "creating" $value
                  mkdir -p $value && chown -R $user $value
              fi
          fi
       done
    fi
}

start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    make_dirs
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}

restart() {
    configtest || return $?
    stop
    sleep 1
    start
}

reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}

force_reload() {
    restart
}

configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}

rh_status() {
    status $prog
}

rh_status_q() {
    rh_status >/dev/null 2>&1
}

case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```

$ chmod 755 /apps/nginx/nginx

$ cp /apps/nginx/nginx /etc/rc.d/init.d/nginx

#### B. add service

##### case of redhat
$ chkconfig --add nginx  
$ chkconfig --level 35 nginx on
$ chkconfig --list | grep nginx  
```
nginx 0:off 1:off 2:off 3:on 4:off 5:on 6:off
```

##### case of debian
$ sudo update-rc.d -f nginx defaults

#### C. configure
$ mkdir /apps/nginx/1.14.2/conf/sites-available
$ mkdir /apps/nginx/1.14.2/conf/sites-enabled  

$ cp /apps/nginx/1.14.2/conf/nginx.conf /apps/nginx/1.14.2/conf/nginx.conf.default
$ vi /apps/nginx/1.14.2/conf/nginx.conf
```
#user nginx app;
user app app;
worker_processes 1;

#error_log logs/error.log;
#error_log logs/error.log notice;
#error_log logs/error.log info;

pid logs/nginx.pid;

events {
    use epoll;
    worker_connections  1024;
}

http {  
    ####################  
    # Basic Settings  
    ####################  
    sendfile on;  
    tcp_nopush on;  
    tcp_nodelay on;  
    keepalive_timeout 65;  
    types_hash_max_size 2048;  
    # server_tokens off;
    
    # server_names_hash_bucket_size 64;  
    # server_name_in_redirect off;
    
    include mime.types;  
    default_type application/octet-stream;
    
    ####################  
    # Logging Settings  
    ####################  
    access_log /logs/nginx/access.log;  
    error_log /logs/nginx/error.log;
    
    ####################  
    # Gzip Settings  
    ####################  
    gzip on;  
    gzip_disable "msie6";
    
    # gzip_vary on;  
    # gzip_proxied any;  
    # gzip_comp_level 6;  
    # gzip_buffers 16 8k;  
    # gzip_http_version 1.1;  
    # gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;
    
    ####################  
    # nginx-naxsi config : Uncomment it if you installed nginx-naxsi  
    ####################  
    #include /etc/nginx/naxsi_core.rules;
    
    ####################  
    # nginx-passenger config : Uncomment it if you installed nginx-passenger  
    ####################  
    #passenger_root /usr;  
    #passenger_ruby /usr/bin/ruby;
    
    ####################  
    # Virtual Host Configs  
    ####################  
    include conf.d/*.conf;  
    include sites-enabled/*;
    
}
```

##### load-balancing was(tomcat) instances
$ vi /apps/nginx/1.14.2/conf/sites-available/api.mobon.net.conf  
```
#map $http_upgrade $connection_upgrade {
#    default     "upgrade";
#}

upstream tomcat {
    #LB method : least_conn, ip_hash  
    ip_hash;
    
    ## proxy server  
    server 127.0.0.1:8080;

    keepalive 1024;
}  

server {  
    listen 80;  
    server_name api.mobon.net;
    
    access_log /logs/nginx/tomcat_access.log;
    
    keepalive_timeout 10;
    
    location / {  
        root /pgms/www;  
        index index.html index.htm;  
    }
    
    # redirect server error pages to the static page /50x.html  
    error_page 500 502 503 504 /50x.html;  
    location = /50x.html {
        root html;
    }
    
    location /api {
        #proxy_pass http://127.0.0.1:8080;
        proxy_pass http://tomcat;
        proxy_redirect off;

        proxy_http_version  1.1;
#        proxy_set_header    Connection          $connection_upgrade;
#        proxy_set_header    Upgrade             $http_upgrade;

        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-NginX-Proxy true;

        charset utf-8;
        
        index index.jsp index.html;
        
        if ($request_filename ~* ^.*?/([^/]*?)$) {
            set $filename $1;
        }
        
        if ($filename ~* ^.*?\.(eot)|(ttf)|(woff)$) {
            add_header Access-Control-Allow-Origin *;
        }
    
    }
    
}
```

$ ln -s /apps/nginx/1.14.2/conf/sites-available/api.mobon.net.conf /apps/nginx/1.14.2/conf/sites-enabled/api.mobon.net.conf

## 4. execution(starting and stopping daemon services)

#### A. start service
$ service nginx start

#### B. reload configuration
$ service nginx reload

#### C. stop service
$ service nginx stop

## 8. trouble-shooting

## 9. Appendix

#### reference site

* Nginx HTTP Server - 2. 설치  
http://ohgyun.com/478

* [Ubuntu] 우분투 NGINX(엔진엑스) Configure 옵션  
http://webdir.tistory.com/238

* Nginx 소스 컴파일 설치 및 HTTPS 설정하기 우분투 16.04  
https://www.wsgvet.com/bbs/board.php?bo_table=ubuntu&wr_id=[[https://www.wsgvet.com/bbs/board.php?bo_table=ubuntu&wr_id=68|68]]

* NGINX 컴파일  
https://opentutorials.org/module/384/4511

* NGINX Init Scripts  
https://www.nginx.com/resources/wiki/start/topics/examples/initscripts/

* nginx.conf / sites-enabled/ sites-available  
http://i5on9i.blogspot.kr/2016/01/nginxconf-sites-enabled-sites-available.html

* 강력한 웹서버 NGINX  
http://egloos.zum.com/repository/v/5676451

* How To Optimize Nginx Configuration  
https://www.digitalocean.com/community/tutorials/how-to-optimize-nginx-configuration

* nginx Performance  
http://kwonnam.pe.kr/wiki/nginx/performance

* Nginx 이해하기 및 기본 환경설정 세팅하기  
http://whatisthenext.tistory.com/123

* NGINX 기본 환경 설정 튜닝 및 설명  
https://extrememanual.net/9976

* Nginx + tomcat 연동 시리즈 3탄 - 연동하기  
http://storyofdream.tistory.com/124


- php fpm 설치 및 설정  
https://www.lesstif.com/pages/viewpage.action?pageId=2444507[[https://www.lesstif.com/pages/viewpage.action?pageId=24445073|3]]

- nginx php-fpm 설치  
http://itraveler.tistory.com/28

