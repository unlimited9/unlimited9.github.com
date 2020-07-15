# container restart at daemon startup

#### --restart option : container auto-start

docker container가 죽는 경우 재시작을 할 수 있도록 설정

- no: container를 재시작 시키지 않는다. (default)
- on-failure[:max-retries]: container가 정상적으로 종료되지 않은 경우(exit code가 0이 아님)에만 재시작 시킨다.  
    max-retries도 함께 주면 재시작 최대 시도횟수를 지정할 수 있고, 테스트 서버 등과 같은 리모트에 설정하면 좋을 것 같다.
- always: container를 항상 재시작시킨다. exit code 상관 없이 항상 재시작 된다.
- unless-stopped: container를 stop시키기 전 까지 항상 재시작 시킨다.

.example
```
docker run --name nsuslab.ggcore.mssql \
    -p 1401:1433 -d \
    -v C:/ggcore/data/mssql:/var/opt/mssql \
    -e ACCEPT_EULA=Y -e MSSQL_SA_PASSWORD=P@ssw0rd \
    --restart=unless-stopped \
    microsoft/mssql-server-linux:latest
```

## appendix

#### reference.site

- [Docker] Docker 데몬 시작 시 마다 container 함께 시작시키기  
https://blog.leocat.kr/notes/2019/11/20/docker-restart-container-whenever-start-docker-desktop
