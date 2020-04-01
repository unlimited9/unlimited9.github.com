#### Optimize(performance tunning)

#### Nginx TCP keepalive 
$ vi /apps/nginx/1.14.2/conf/sites-enabled/product.mobon.net.conf
```
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
    ...
```

#### Nginx worker processes and limit files
$ vi /apps/nginx/1.14.2/conf/nginx.conf
```
...
worker_processes auto;
...
worker_rlimit_nofile 16384;
...
events {
    use epoll;
    worker_connections 4096;
}
```

`worker_processes`  
nginx가 싱글스레드로 동작하기 때문에 core 수를 설정
core수 보다 높은 숫자를 저정해도 문제는 없으며, auto로 지정하는 경우에는 자동으로 지정해준다.

>`linux core 수`  
>$ grep processor /proc/cpuinfo |wc

`worker_rlimit_nofile` : 프로세스당 파일 디스크립터의  상한(上限)수  
일반적으로 worker_connections 3~4배  

>`linux file description`  
>$ cat /proc/sys/fs/file-max

`worker_connections` : 하나의 worker가 동시에 처리할수 있는 접속수

##### worker_processes
> 프로세스 수  : CPU 혹은 CPU Core 수와 같이 설정한다. (보통은 4개 정도가 넘어가면 이미 최대 성능치일 경우가 많다.)  

$ grep processor /proc/cpuinfo | wc -l

##### worker_connection
> 하나의 worker_process가 받을 수 있는 클라이언트 갯수  
최대 접속자수(MaxClients) = worker_processes * worket_connections  
Reverse Proxy 상태에서는 worker_processes * worker_connections / 4 이 값은 ulimit -n의 결과값(open files)보다 작아야 한다. 보통 1024면 충분하다. 

$ ulimit -n
```
worker_rlimit_nofile 65535;
```

$ vi /etc/security/limits.conf  
```
...  
-   hard nofile 65536
-   soft nofile 65536
```

##### buffer size
Proxy를 사용할 경우 버퍼의 크기가 너무 작으면 nginx는 임시 파일을 만들어 proxy에서 전달되는 내용을 저장해 빈번한 DISK IO 발생. 장비의 메모리 상황등을 참조하여 적당한 수준으로 늘려줘야 한다.

- client_body_buffer_size : 클라이언트 버퍼 사이즈, POST Action 과 연관이 있다.
- client_header_buffer_size : 클라이언트 헤더 버퍼 사이즈 1KB
- client_max_body_size : 클라이언트에서 허용되는 최대 사이즈, 초과하게 되면 413 error를 뱉거나 Request Entity Too Large 를 리턴
- large_client_header_buffers : 큰 클라이언트의 헤더를 위한 개수와 사이즈 지정
```
client_body_buffer_size 8K;  
client_header_buffer_size 1k;  
client_max_body_size 2m; # 파일 업로드를 2mb 이상할 예정이라면 이 값을 늘려줘야 한다.  
large_client_header_buffers 2 1k;
```

##### Timeouts  
성능을 개선할수 있다.
- client_body_timeout
- client_header_timeout

이 두개는 client 가 요청수 서버의 응답을 기다리는 시간. 초과시 408 에러, request time out
- keepalive_timeout : 클라이언트의 keepalive connection 의 시간 설정
- send_timeout : 특정(지정한) 시간 이후에 클라이언트가 아무것도 하지 않으면 연결 종료시킨다.

##### GZIP Compression  
nginx 에서 처리해야 하는 네트워크의 양을 줄인다. gzip_comp_level을 너무 높이면 cpu cycle을 낭비하게 된다.

##### Static File Caching  
header 에 쓰이는 파일(변하지 않고 서버에서 정기적으로 제공하는)에 만기(expire)를 설정해라. server 블록에 있다.

##### Logging  
nginx에 모든 요청 로그를 남긴다. 꺼라

##### 결론  
알맞게 설정된 서버가 가장 중요한데 이것은 모니터링을 해보고 적절히 수정해 봐야 한다. 위의 설정 중 영원한 것은 없으며, 각각의 독특한 경우에 맞게 조정이 필요하다. 그리고 나서 로드 밸런싱이나 수평적 확장을 알아보면 된다.
