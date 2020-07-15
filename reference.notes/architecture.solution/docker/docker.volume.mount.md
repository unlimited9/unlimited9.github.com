# Docker Volume

Docker 컨테이너의 생명 주기와 관계없이 데이터를 영속적으로 저장

## Volume / Bind mount

#### Volume

`create volume`  
docker volume create volume_name  

`volume list`  
docker volume ls  

`volume mount`  
docker run -v volume_name:/app --name container_name busybox touch /app/test.txt  

`docker volume path`  
ls /var/lib/docker/volumes/volume_name/_data  

`docker inspect`  
docker inspect container_name  
```
...
    "Mounts": [
        {
            "Type": "volume",
            "Name": "volume_name",
            "Source": "/var/lib/docker/volumes/volume_name/_data",
            "Destination": "/app",
            "Driver": "local",
            "Mode": "z",
            "RW": true,
            "Propagation": ""
        }
    ],
...
```

`volume mount another container`  
docker run -v volume_name:/app --name second_container_name busybox ls /app  

`remove volume`  
docker volume rm volume_name  

> docker volume prune


#### Bind mount

`bind mount`  
docker run -v `pwd`:/app --name bind_container_name busybox touch /app/bind_test.txt  

`docker inspect`  
docker inspect bind_container_name  
```
...
    "Mounts": [
        {
            "Type": "bind",
            "Source": "/root",
            "Destination": "/app",
            "Mode": "",
            "RW": true,
            "Propagation": "rprivate"
        }
...
```

#### 
docker volume create  
docker volume ls  
docker volume inspect  
docker volume rm  
docker volume prune  


## Appendix

#### reference site

* Reference documentation  
https://docs.docker.com/reference/  

* docker volume create  
https://docs.docker.com/engine/reference/commandline/volume_create/  

+ Docker 컨테이너에 데이터 저장 (볼륨/바인드 마운트)  
https://www.daleseo.com/docker-volumes-bind-mounts/
