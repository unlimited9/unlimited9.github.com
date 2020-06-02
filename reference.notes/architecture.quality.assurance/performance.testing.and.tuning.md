# 1. System Tuning
>- UT : Unit Testing
>- **ST : System Testing**
>- SIT : System integration testing
>- UAT : User Acceptance Testing

## System Testing

#### performance testing
>Created 월요일 06 8월 2019
>
>jmeter : application is open source software, a 100% pure Java application designed to load test functional behavior and measure performance.

서비스 및 시스템의 성능을 확인을 위해 실제 사용 환경과 비슷한 환경에서 테스트를 수행한다.
* 성능(Performance) Test : 성능 측정  
  응답시간, 처리량, 병목구간, 서버리소스(CPU, Memory, DISK) 측정, 문제점 확인 및 개선
* 부하(Load : Volume, Endurance) Test : 임계치 및 튜닝 포인트 확인  
  부하를 지속적으로 증가시켜서 시스템 상태 및 임계치 확인
* 스트레스(Stress) Test : 장애 조치 및 시스템 성능 한계 확인  
  과부하 및 비정상적인 상황에 대한 장애 조치와 복구 절차 확인 및 시스템 최고 성능 한계 확인
* 안정성(Stability: Soak) Test : 자원(메모리 등) 누수 확인  
  지속적인 부하에 대한 시스템 리소스 및 성능 정보 변화 확인
* 스파이크(Spike) Test : 부하 집중 테스트  
  짧은 시간에 부하 증가에 대한 처리 및 업무 부하(workload) 감소 시 안정화 확인
* 피로(Fatigue) Test : 대역폭 용량을 뛰어넘는 부하에 대한 테스트

#### Environment Setup

* nGrinder : [Install and setup nGrinder](install.n.setup.ngrinder.md)  
Enterprise level performance testing solution based on The Grinder
* Scouter : [Install and setup Scouter](install.n.setup.scouter.md)  
SCOUTER is an open source APM like new relic and appdynamics

## Tuning

---

## Troubleshooting

시스템에 증가하는 부하가 WAS까지 도달하지 못하는 경우 프록시 서버 로그를 확인한다.

#### Nginx Error Log : upstream timed out
$ vi /logs/nginx/error.log
```
...
2019/08/22 05:59:21 [error] 4730#0: *13332943 connect() failed (111: Connection refused) while connecting to upstream, client: 172.20.0.107, server: product.mobon.net, request: "POST /dspt/product HTTP/1.1", upstream: "http://172.20.0.103:16080/dspt/product", host: "product.mobon.net:99"
2019/08/22 05:59:24 [error] 4730#0: *13323058 upstream timed out (110: Connection timed out) while reading response header from upstream, client: 172.20.0.107, server: product.mobon.net, request: "POST /dspt/product HTTP/1.1", upstream: "http://172.20.0.103:15080/dspt/product", host: "product.mobon.net:99"
...
```

Local TCP Port 고갈에 따른 오류 확인  

`Check Network Socket Status`  
$ ss -s
```
Total: 1925 (kernel 2483)
TCP:   34210 (estab 993, closed 32591, orphaned 0, synrecv 0, timewait 32502/0), ports 1319

Transport Total     IP        IPv6
*	  2483      -         -        
RAW	  0         0         0        
UDP	  7         5         2        
TCP	  1619      1607      12       
INET	  1626      1612      14       
FRAG	  0         0         0        
```
$ netstat -napo | grep -i time_wait
```
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address               Foreign Address             State       PID/Program name    Timer
'''
tcp        0      0 172.20.0.103:19080          172.20.0.102:57910          TIME_WAIT   -                   timewait (39.81/0/0)
tcp        0      0 172.20.0.103:19080          172.20.0.102:59722          TIME_WAIT   -                   timewait (42.19/0/0)
tcp        0      0 172.20.0.103:15080          172.20.0.102:54368          TIME_WAIT   -                   timewait (0.00/0/0)
tcp        0      0 172.20.0.103:15080          172.20.0.102:16366          TIME_WAIT   -                   timewait (5.74/0/0)
tcp        0      0 172.20.0.103:17080          172.20.0.102:25334          TIME_WAIT   -                   timewait (0.00/0/0)
tcp        0      0 172.20.0.103:19080          172.20.0.102:10898          TIME_WAIT   -                   timewait (52.68/0/0)
tcp        0      0 172.20.0.103:17080          172.20.0.102:15506          TIME_WAIT   -                   timewait (49.98/0/0)
tcp        0      0 172.20.0.103:16080          172.20.0.102:17482          TIME_WAIT   -                   timewait (12.73/0/0)
tcp        0      0 172.20.0.103:17080          172.20.0.102:36576          TIME_WAIT   -                   timewait (0.00/0/0)
tcp        0      0 172.20.0.103:17080          172.20.0.102:64346          TIME_WAIT   -                   timewait (39.93/0/0)
tcp        0      0 172.20.0.103:19080          172.20.0.102:58350          TIME_WAIT   -                   timewait (40.30/0/0)
'''
```
$ netstat -napo | grep -ic 172.20.0.103
```
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
30931
```

`Increase TCP Port Range with net.ipv4.ip_local_port_range Kernel Parameter`  
$ cat /proc/sys/net/ipv4/ip_local_port_range
```
32768	60999
```
$ echo "10240 65535" > /proc/sys/net/ipv4/ip_local_port_range
> $ echo "10240 65535" | sudo tee -a /proc/sys/net/ipv4/ip_local_port_range

$ cat /proc/sys/net/ipv4/ip_local_port_range
```
10240	65535
```

`Reuse TCP Port with Kernel Parameter`  
$ sysctl -a | grep "net.ipv4.tcp_tw_reuse"
```
net.ipv4.tcp_tw_reuse = 0
```
$ sudo sysctl -w "net.ipv4.tcp_timestamps=1"
$ sudo sysctl -w "net.ipv4.tcp_tw_reuse=1"

$ sysctl -a | grep "net.ipv4.tcp_tw_reuse
```
net.ipv4.tcp_tw_reuse = 1
```

`Nginx TCP keepalive`  
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

#### Nginx Error Log : Too many open files
$ vi /logs/nginx/error.log
```
...
2020/04/01 10:37:47 [crit] 60336#0: *27361 open() "/apps/nginx/1.17.9/html/50x.html" failed (24: Too many open files), client: 61.253.101.72, server: tk.mediacategory.com, request: "POST /aggregate/mssg/common HTTP/1.1", upstream: "http://10.251.0.184:80/tk/aggregate/mssg/common", host: "tk.mediacategory.com", referrer: "http://www.sappun.co.kr/shop/shopbrand.html?xcode=045&type=P"
...
```

`Nginx worker processes and limit files`  
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

+ worker_processes : nginx가 싱글스레드로 동작하기 때문에 core 수를 설정  
core수 보다 높은 숫자를 저정해도 문제는 없으며, auto로 지정하는 경우에는 자동으로 지정해준다.
  >`linux core 수`  
  >$ grep processor /proc/cpuinfo |wc

+ worker_rlimit_nofile : 프로세스당 파일 디스크립터의  상한(上限)수  
일반적으로 worker_connections 3~4배  
  >`linux file description`  
  >$ cat /proc/sys/fs/file-max

+ worker_connections : 하나의 worker가 동시에 처리할수 있는 접속수


#### su: cannot set user id: Resource temporarily unavailable 에러
#### su: failed to execute /bin/bash: Resource temporarily unavailable

`kernel log`  
$ dmesg -T | tail -2  

`system log`  
$ vi /var/log/messages  
$ vi /var/log/secure  
```
...
Jun  2 09:31:35 mpk-cluster-02 sshd[14445]: error: do_exec_pty: fork: Resource temporarily unavailable
...
```

`사용 프로세스 수 확인`  
$ ps -ef -L -u app | wc -l  

$ ulimit -a  

When the system runs into a limitation in the number of processes, increase the nproc value in /etc/security/limits.conf or /etc/security/limits.d/90-nproc.conf depending on RHEL version. The limit can be increased for a specific user or all users.  
$ vi /etc/security/limits.conf
```
'''
* hard nproc 65536
* soft nproc 65536
'''
```
> ```
>...
>* hard nofile 65536
>* soft nofile 65536
>...
>```

$ vi /etc/security/limits.d/90-nproc.conf  
```
# Default limit for number of user's processes to prevent
# accidental fork bombs.
# See rhbz #432903 for reasoning.

*          soft    nproc     65536
root       soft    nproc     unlimited
```

>`NIC 2가지 모드`  
>1. Normal mode : 인터페이스로 들어오는 패킷의 목적지 주소를 확인하고 목적지가 자신인 패킷과 브로드캐스트 패킷만 받아들인다.  
>2. promiscuos mode : 인터페이스로 들어오는 패킷들을 MAC주소와 상관없이 모든 패킷을 받는다.  


## 9. Appendix

### reference site

#### 테스트
* 성능 엔지니어링 대한 접근 방법 (Performance tuning)
https://bcho.tistory.com/787
* 결제 시스템 성능, 부하, 스트레스 테스트
http://woowabros.github.io/experience/2018/05/08/billing-performance_test_experience.html
* 성능, 부하, 스트레스 테스트에 대하여
https://nesoy.github.io/articles/2018-08/Testing-Performance

#### 튜닝
- 1.AWS Beanstalk을 이용한 성능 튜닝 시리즈 - DB Connection Pool
https://jojoldu.tistory.com/318
- 2.AWS Beanstalk을 이용한 성능 튜닝 시리즈 - 커널 파라미터 튜닝
https://jojoldu.tistory.com/319
- 3.AWS Beanstalk을 이용한 성능 튜닝 시리즈 - Nginx 튜닝
https://jojoldu.tistory.com/322?category=777282

 + CLOSE_WAIT & TIME_WAIT 최종 분석
http://tech.kakao.com/2016/04/21/closewait-timewait/
 + DevOps와 SE를 위한 리눅스 커널 이야기
https://m.blog.naver.com/PostView.nhn?blogId=jevida&logNo=221249096145&proxyReferer=https%3A%2F%2Fwww.google.com%2F

#### 모니터링
+ 성능테스트시 서버 모니터링 방법 정리
https://chigon.tistory.com/entry/%EC%84%B1%EB%8A%A5-%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%8B%9C-%EC%84%9C%EB%B2%84-%EB%AA%A8%EB%8B%88%ED%84%B0%EB%A7%81-%EB%B0%A9%EB%B2%95-%EC%A0%95%EB%A6%AC
+ 리눅스 명령어를 이용한 시스템 모니터링하기
https://www.whatap.io/blog/10/

#### JMeter
- JMeter - KIMSTAR 3.0
http://kimstar.kr/8282/
- Jmeter를 이용한 성능테스트 정보 공유
https://www.sten.or.kr/club/club_main.php?cb_id=cb_Jmeter
- jmeter-plugins : Transactions per Second
https://jmeter-plugins.org/wiki/TransactionsPerSecond/
