# logrotate

## logrotate 관리 정책 작성
특정 로그 파일에 대한 관리 정책을 /etc/logrotate.d 경로에 설정 파일로 등록해두면, logrotate가 크론 잡에 의해 매일 1회 자동으로 실행되어 로그 파일을 관리해준다.

#### nginx
$ sudo vi /etc/logrotate.d/nginx
```
/logs/nginx/tk.mediacategory.com_access.log {
    daily
    dateext
    rotate 30
    copytruncate
    missingok
}

/logs/nginx/tk.mediacategory.com_error.log {
    daily
    dateext
    rotate 30
    copytruncate
    missingok
}
```

#### options
- daily 옵션은 로그 파일을 일 단위로 분리하겠다는 의미이다. 크론 잡이 이 역할을 수행해준다.
- dateext 옵션은 로그 파일 분리시 분리된 파일에 대한 작명을 로그파일명_YYYYMMDD로 하여 저장하겠다는 의미이다. (ex: access.log-20191025)
- rotate 30 옵션은 분리된 로그 파일을 최대 30개 보관하고, 나머지 오래된 파일은 삭제하겠다는 의미이다. 위 예제에서는 앞서 daily 옵션에 의해 일 단위로 로그 파일을 격리하도록 설정했으므로, 최대 30일을 보관하겠다는 의미가 된다.
- copytruncate 옵션은 원본 로그 파일을 유지한채 분리한 부분만 제거하겠다는 의미이다. 특정 애플리케이션은 로그 파일의 이동을 허용하지 않는 경우가 있어, 이 경우 권장되는 옵션이다. 단점은 분리한 부분을 제거하는 아주 짦은 시간동안 새로운 유입 로그가 유실될 가능성이 있다.
- ssingok 옵션은 대상 로그 파일이 존재하지 않을 경우, 오류 메시지 없이 계속 작업을 진행하겠다는 의미이다.

#### Don't do anything, just test (implies -v)
$ sudo logrotate -df /etc/logrotate.d/nginx

#### Run right away
$ sudo logrotate -vf /etc/logrotate.d/nginx

## Appendix

#### reference site

* logrotate(8) - Linux man page
https://linux.die.net/man/8/logrotate
