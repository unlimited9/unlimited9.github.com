# Remove unused resources

## p


· docker container prune은 중지된 모든 컨테이너를 삭제한다.



· docker image prune은 이름 없는 모든 이미지를 삭제한다.



· docker network prune은 사용되지 않는 도커 네트워크를 모두 삭제한다.



· docker volume prune은 도커 컨테이너에서 사용하지 않는 모든 도커 볼륨을 삭제한다.



· docker system prune -a는 중지된 모든 컨테이너, 사용되지 않은 모든 네트워크, 하나 이상의 컨테이너에서 사용되지 않는 모든 이미지를 삭제한다. 따라서 남아있는 컨테이너 또는 이미지는 현재 실행 중인 컨테이너에서 필요하다.


사용하지 않는 이미지 삭제
docker rmi $(docker images -f dangling=true -q)
특정 이미지 전체 삭제
docker rmi $(docker images 이미지이름 -q)
docker rmi $(docker images 이미지이름앞부분/* -q)
사용하지 않는 컨테이너 삭제
docker rm $(docker ps -aq -f status=exited)





## appendix

#### reference.site

- [docker] 도커 환경 정리 커맨드- docker prune 커맨드
https://knight76.tistory.com/entry/docker-도커-환경-정리-커맨드-docker-prune-커맨드

- docker local image 정리  
https://devnote.niceilm.net/docker-local-image/
