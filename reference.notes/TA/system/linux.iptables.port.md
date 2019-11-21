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


## port routing

#### 추가
$ iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 8080

#### 삭제
$ iptables -D PREROUTING -t nat -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 8080

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
