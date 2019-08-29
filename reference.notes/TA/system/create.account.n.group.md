# Account/Group

>Created 목요일 30 11월 2017  
Create xNix System account/group

## creating required operating system group and user

#### create the os application group(typically, app)
$ groupadd -g 3000 app

어플리케이션 단위 관리 주체가 다르지 않고 권한을 나눌 필요가 없어 통합 계정을 생성하여 관리 어플리케이션 단위 관리 주체가 다르다면 어플리케이션별 계정을 생성하여 생성하여 관리

#### create the software unified account (typically, app)
$ useradd -d /apps -g 3000 -m -u 3000 -s /bin/bash app  
$ passwd app

>#### create the nginx software owner (typically, nginx)
>$ useradd -d /apps/nginx -g 3000 -m -u 3020 -s /bin/bash nginx  
>$ passwd nginx
>
>#### 터미널 접속 로그인 제한(FTP 접속 가능)
>$ useradd -d /apps/nginx -g 3000 -m -u 3020 -s /bin/bash nginx -s /sbin/nologin  
>$ su - nginx
>```
>This account is currently not available
>```
>
>#### nologin 메세지 수정
>$ vi /etc/nologin.txt
>
> FTP 접속 제한 : login shell 등록/삭제(터미널 접속 로그인은 되지만 FTP 접속은 제한)
>$ vi /etc/shells
>```
>/bin/sh
>/bin/bash
>/sbin/nologin
>/bin/dash
>```

#### usermod (append group)
$ usermod --append --groups app nginx

#### service status
$ sudo /sbin/service --status-all

## Appendix

### reference site

- /sbin/nologin 설정
http://blog.naver.com/PostView.nhn?blogId=01191879872&logNo=10044683661
