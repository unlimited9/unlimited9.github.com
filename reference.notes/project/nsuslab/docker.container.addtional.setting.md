# docker container addtional setting

## SSH

#### install openssh-server
>apt update -y  

apt-get install openssh-server  

service ssh start  
>systemctl start ssh  

netstat -tnlp  

#### set configuration 
vi /etc/ssh/sshd_config
```
#PermitRootLogin prohibit-password
PermitRootLogin yes
...
#PasswordAuthentication yes
PasswordAuthentication yes
``` 
service ssh restart  
>systemctl restart ssh  

## Appendix

#### reference.site

* [Linux] Ubuntu 18.04 SSH서버 구축하기 및 SSH Root 계정 접속 설정 (Ubuntu OpenSSH Server)  
https://antdev.tistory.com/48  

