
## CLOSE_WAIT

$ lsof -i:8080

$ netstat -tonp

## TIME_WAIT

#### 네트워크 소켓/상태 조회
$ ss -ant | awk '{print $1}' | grep -v '[a-z]' | sort | uniq -c


#### 포트 범위 설정 조회
$ sysctl net.ipv4.ip_local_port_range  
```
net.ipv4.ip_local_port_range = 32768  61000
```

#### 파일시스템 최대 파일 수
$ sysctl fs.file-max  
```
fs.file-max = 262144
```

#### 소켓 재활용 설정
$ echo 1 > /proc/sys/net/ipv4/tcp_tw_reuse  
$ sysctl net.ipv4.tcp_tw_reuse  
```
net.ipv4.tcp_tw_reuse = 1
```

#### CPU 사용량  
$ top -b -n1 | grep -Po '[0-9.]+ id' | awk '{print 100-$1}'  


## 9. Appendix

#### reference site

* CLOSE_WAIT & TIME_WAIT 최종 분석  
https://tech.kakao.com/2016/04/21/closewait-timewait/

* CLOSE_WAIT 문제 해결  
http://docs.likejazz.com/close-wait/

* TIME_WAIT 상태란 무엇인가  
http://docs.likejazz.com/time-wait/

* 리눅스 CPU 사용률 확인  
https://zetawiki.com/wiki/%EB%A6%AC%EB%88%85%EC%8A%A4_CPU_%EC%82%AC%EC%9A%A9%EB%A5%A0_%ED%99%95%EC%9D%B8
