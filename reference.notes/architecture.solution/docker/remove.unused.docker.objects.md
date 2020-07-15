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

#### Dangling Image 한번에 제거하기
docker images -f dangling=true | awk 'NR > 1 { print $3 }' | xargs -I{} docker rmi {}  

#### 종료된 컨테이너들만 지워주는 명령
docker ps -a -f status=exited | awk 'NR > 1 { print $1 }' | xargs -I{} docker rm {}  

## appendix

#### reference.site

- [docker] 도커 환경 정리 커맨드- docker prune 커맨드  
https://knight76.tistory.com/entry/docker-도커-환경-정리-커맨드-docker-prune-커맨드  

- docker local image 정리  
https://devnote.niceilm.net/docker-local-image/

- Dangling Image 한번에 제거하기  
https://blog.kesuskim.com/archives/rm-dangling-docker-images/  
