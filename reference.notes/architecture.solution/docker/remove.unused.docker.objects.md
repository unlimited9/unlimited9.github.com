# remove unused docker objects

## Prune unused Docker objects

docker container prune  
docker image prune  
docker network prune  
docker volume prune  

> docker system prune -a  

#### remove unused images
docker rmi $(docker images -f dangling=true -q)  

docker rmi $(docker images image_name -q)  
docker rmi $(docker images image_name/* -q)  

docker rm $(docker ps -aq -f status=exited)  

## appendix

#### reference.site

- [docker] 도커 환경 정리 커맨드- docker prune 커맨드
https://knight76.tistory.com/entry/docker-도커-환경-정리-커맨드-docker-prune-커맨드

- docker local image 정리  
https://devnote.niceilm.net/docker-local-image/
