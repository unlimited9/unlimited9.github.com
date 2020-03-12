# linux : change hostname 

>Created 월요일 20 11월 2017  
Linux OS command

## CentOS

#### change hostname (재기동 시 리셋)

`hostname 확인`  
$ hostname  
```
localhost.localdomain
```

`hostname 변경`  
$ hostname new_hostname

#### set hostname (영구 변경)
`CentOS 6`  
$ vi /etc/sysconfig/network
```
HOSTNAME=new_hostname
```

`CentOS 7`  
$ hostnamectl set-hostname new_hostname

## 9. Appendix

#### reference site

+ 리눅스(linux) hostname 변경하는 방법 (CentOS 6, 7)  
https://webinformation.tistory.com/40
