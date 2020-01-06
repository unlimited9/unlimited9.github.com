## iptables

#### 포트(서비스) 확인
$ netstat -nap  
$ netstat -nap | grep 80  
$ netstat -nap | grep LISTEN

#### 방화벽 확인 
$ iptables -nL  

#### 포트 열기
$ iptables -I INPUT 1 -p tcp --dport 80 -j ACCEPT
> 방화벽 1번 규칙으로 80 포트로 들어오는 접속 허용  
> I : 새로운 규칙 추가 
> p : 패킷의 프로토콜 명시 
> j : 규칙에 해당되는 패킷 처리

#### 포트 삭제
$ iptables -D INPUT -p tcp --dport 80 -j ACCEPT


## port forwarding

Linux/Unix에 보안적인 문제로 1024 이하의 포트(well-known port)들은 추가 설정(sudoers 등)을 하지 않으면 일반 유저 권한에서 안되고 root로만 바인딩된다.

>#### 커널변수에 IP포워딩 가능하도록 설정
>$ sudo bash -c 'echo 1 > /proc/sys/net/ipv4/ip_forward'  
>> $ echo 1 > /proc/sys/net/ipv4/ip_forward

#### 추가
`port : 80`  
$ sudo iptables -t nat -A OUTPUT -d localhost -p tcp --dport 80 -j REDIRECT --to-ports 8080  
$ sudo iptables -t nat -A PREROUTING -d localhost -p tcp --dport 80 -j REDIRECT --to-ports 8080  
`port : 443`  
$ sudo iptables -t nat -A OUTPUT -d localhost -p tcp --dport 443 -j REDIRECT --to-ports 843  
$ sudo iptables -t nat -A PREROUTING -d localhost -p tcp --dport 443 -j REDIRECT --to-ports 8443  

>$ sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE  
>$ sudo iptables -t nat -A PREROUTING  -p tcp --dport 80   -j REDIRECT --to-ports 8080  
>$ sudo iptables -t nat -A PREROUTING  -p tcp --dport 443  -j REDIRECT --to-ports 8443  
>> $ sudo iptables -t nat -A PREROUTING -p tcp -d 서버ip --dport 80 -j REDIRECT --to-port 8080

#### 삭제
`port : 80`  
$ sudo iptables -t nat -D OUTPUT -d localhost -p tcp --dport 80 -j REDIRECT --to-ports 8080  
$ sudo iptables -t nat -D PREROUTING -d localhost -p tcp --dport 80 -j REDIRECT --to-ports 8080  
`port : 443`  
$ sudo iptables -t nat -D OUTPUT -d localhost -p tcp --dport 443 -j REDIRECT --to-ports 843  
$ sudo iptables -t nat -D PREROUTING -d localhost -p tcp --dport 443 -j REDIRECT --to-ports 8443  

>$ sudo iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE  
>$ sudo iptables -t nat -D PREROUTING  -p tcp --dport 80   -j REDIRECT --to-ports 8080  
>$ sudo iptables -t nat -D PREROUTING  -p tcp --dport 443  -j REDIRECT --to-ports 8443  
>> $ sudo iptables -t nat -D PREROUTING -p tcp -d 서버ip --dport 80 -j REDIRECT --to-port 8080

#### 저장
$ service iptables save

#### 확인
$ iptables -t nat -L

#### 재시작
$ service iptables restart

## 9. Appendix

#### reference site

+ [LINUX] 포트 열기, 확인하기  
https://sssunho.tistory.com/67

+ PORT REDIRECT [80 to 8080]    
https://lofty87.tistory.com/34

+ Linux / iptables를 이용한 Port forwarding 구성 방법  
https://m.blog.naver.com/tkstone/50193603872

+ [Linux] 리눅스 NAT : SNAT & DNAT  
https://itragdoll.tistory.com/5

+ Port 80 redirect does not work for localhost  
https://serverfault.com/questions/464410/port-80-redirect-does-not-work-for-localhost

+ Redirect port 80 to 8080 and make it work on local machine  
https://askubuntu.com/questions/444729/redirect-port-80-to-8080-and-make-it-work-on-local-machine

+ iptables설정  
https://linuxism.ustd.ip.or.kr/685

- Tomcat 80포트로 바인딩하기(80,443 포트 권한 바인딩)  
https://sysadm.kr/287

- TOMCAT 80 포트 운용방법  
https://cafe.naver.com/egosproject/29
