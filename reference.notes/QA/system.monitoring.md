#### 네트워크상태조회
$ ss -ant | awk '{print $1}' | grep -v '[a-z]' | sort | uniq -c

#### 소켓재활용
$ echo 1 > /proc/sys/net/ipv4/tcp_tw_reuse  
$ sysctl net.ipv4.tcp_tw_reuse  
```
net.ipv4.tcp_tw_reuse = 1
```


## 9. Appendix

### reference site

* CLOSE_WAIT & TIME_WAIT 최종 분석  
https://tech.kakao.com/2016/04/21/closewait-timewait/
