## installation php (optional)

### A. add remi, WebTatic repository
PHP 최신 버전을 제공하는 외부 repository 중 유명한 곳은 webtatic과 remi 등이 있다.

#### CentOS 6
Installing the epel repository  
```bash
$ rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
```

Installing the WebTatic repository  
```bash
$ rpm -Uvh http://mirror.webtatic.com/yum/el6/latest.rpm
```

Installing the Remi repository  
```bash
$ rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
```

#### CentOS 7  
Installing the epel repository  
```bash
$ rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
```
 
Installing the WebTatic repository  
```bash
$ rpm -Uvh https://mirror.webtatic.com/yum/el7/epel-release.rpm
$ rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
```

Installing the Remi repository  
```bash
$ rpm -Uvh http://mirror.premi.st/epel/7/x86_64/e/epel-release-7-5.noarch.rpm 
$ rpm -Uvh http://rpms.remirepo.net/enterprise/remi-release-7.rpm
```

레파지토리 목록 확인  
```bash
$ yum repolist  
$ ll /etc/yum.repos.d
```

### B. PHP FastCGI Process Manager

#### 설치 PHP 확인 및 제거
```
$ yum list installed | cut -d " " -f 1 | grep php  
$ yum remove -y `yum list installed | cut -d " " -f 1 | grep php`
```

#### 1. 설치  : 7.2버전
>PHP가 버전업되면 운영중인 프로그램과의 호환에 문제가 발생할 수 특정 버전을 고정하여 사용하는 것이 더 안전할 수 있다.(권장)  
php-common" 외의 패키지는 상황에 맞게 필요하면 설치한다.  
```bash
$ yum install -y php72w-common php72w-fpm php72w-cli \
php72w-process \
php72w-opcache php72w-pecl-apcu \
php72w-mysqlnd php72w-pdo \
php72w-gd \
php72w-mbstring php72w-xml \
php72w-pecl-zip \
php72w-bcmath
```
>기본적으로 맨 윗줄 라이브러리만 설치. 워드프레스를 설치하고 이미지 자르기 기능을 위해서는 php72w-gd를 설치해야한다.

#### 2. 설치 : 최신 버전
```bash
$ yum install -y php-common php-fpm php-cli \  
php-process \  
php-opcache php-pecl-apcu \  
php-mysqlnd php-pdo \  
php-gd \  
php-mbstring php-xml \  
php-pecl-zip \  
php-bcmath  
```
> 최신 버전 사용시 삭제하고 다시 설치하지 않고 update를 할 수도 있다.  
$ yum update php-*

#### 버전 확인  
$ php -v

#### Configuration
$ vi /etc/php-fpm.conf

$ vi /etc/php-fpm.d/www.conf  
```
[www]
listen = /var/run/php-fpm/www.sock
;listen.owner = nginx
listen.owner = app
listen.group = app
listen.mode = 0660

;user = nginx
user = app
group = app

pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35
;pm.max_requests = 500

;request_terminate_timeout = 0
;request_slowlog_timeout = 0
slowlog = /var/log/php-fpm/www-slow.log

chdir = /pgms/www

catch_workers_output = yes

php_admin_value[error_log] = /var/log/php-fpm/www-error.log
php_admin_flag[log_errors] = on

php_value[session.save_handler] = files
php_value[session.save_path] = /var/lib/php/session
php_value[soap.wsdl_cache_dir] = /var/lib/php/wsdlcache
```

$ vi /apps/nginx/1.14.0/conf/sites-available/php-fpm.conf
```
server {  
    listen 80;  
    server_name house.ruaniz.com;
    
    access_log /logs/nginx/php-fpm_access.log;
    
    location / {
        root /pgms/www;
        index index.html index.htm;
    }
    
    # redirect server error pages to the static page /50x.html
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root html;
    }
    
    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    location ~ \.php$ {
        try_files $uri =404;
        root /pgms/www;
        fastcgi_pass unix:/var/run/php-fpm/www.sock;
        
        # fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
    
}
```

$ ln -s /apps/nginx/1.14.0/conf/sites-available/php-fpm.conf /apps/nginx/1.14.0/conf/sites-enabled/php-fpm.conf

#### add service and execution
> php-fpm service on boot  

##### CentOS 6  
$ chkconfig --level 35 php-fpm on  
$ service php-fpm start  

##### CentOS 7  
$ systemctl enable php-fpm.service
$ systemctl start php-fpm.service

> 서비스 상태 확인 (오류 확인)

$ systemctl status php-fpm.service

> ref.  $ php-fpm --fpm-config /etc/php-fpm.conf

## 9. Appendix

### reference site

* PHP 7.2 설치(업그레이드) [CentOS 7 / remi RPM repository]
https://blog.asamaru.net/2018/02/14/install-php-7-2-on-centos-with-remi-rpm-repository/

* RHEL/CentOS 5,6,7 에 EPEL 과 Remi/WebTatic Repository 설치하기
https://www.lesstif.com/pages/viewpage.action?pageId=6979743

*Webtatic Yum Repository
https://webtatic.com/projects/yum-repository/
