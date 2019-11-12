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


## 9. Appendix

#### reference site

+ [LINUX] 포트 열기, 확인하기  
https://sssunho.tistory.com/67
