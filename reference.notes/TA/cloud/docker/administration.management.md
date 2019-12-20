# Administration : management and dockerize

>Created 월요일 20 11월 2017
linux container

## 1. change container repository directory

#### make docker configuration file
$ sudo vi /etc/docker/daemon.json
```
{
	"graph": "/data/docker"
}
```
> $ mv /var/lib/docker /data/docker

$ sudo systemctl restart docker

## 2. command

####  컨테이너 실행하기
```bash
$ docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
```
 옵션 | 설명 
:---:|:---
-d | detached mode 흔히 말하는 백그라운드 모드|
-p | 호스트와 컨테이너의 포트를 연결 (포워딩)
-v | 호스트와 컨테이너의 디렉토리를 연결 (마운트)
-e | 컨테이너 내에서 사용할 환경변수 설정
–name | 컨테이너 이름 설정
–rm | 프로세스 종료시 컨테이너 자동 제거
-it | -i와 -t를 동시에 사용한 것으로 터미널 입력을 위한 옵션
–link | 컨테이너 연결 [컨테이너명:별칭]

#### centos/ubuntu container 생성/실행
$ docker run -d -it --name centos.7.app centos:7 /bin/bash
> $ docker run -d -it --name ubuntu.18.04.app ubuntu:18.04 /bin/bash

> `run` : local repository에 해당 이미지가 있는지 확인하고 없다면 local repository에 다운로드(`pull`)를 한 후 컨테이너를 생성(`create`)하고 시작(`start`) 
> `/bin/bash` : 컨테이너 내부에 들어가기 위해 bash 쉘을 실행
> -d : detached mode (daemon process)
> `--name` : optional. 생략하면 도커가 자동 이름을 등록
> `-it` : 키보드 입력 옵션

> $ docker run -it --rm --name centos.7.app centos:7 /bin/bash
> 실행된 bash 쉘에서 `exit`로 bash 쉘을 종료 시 컨테이너도 같이 종료
> `--rm` : 프로세스가 종료 시 컨테이너 자동으로 삭제 옵션

#### How to enter in a Docker container already running with a new TTY
>$ docker exec -it [container-id] /bin/bash

$ docker exec -it centos.7.app /bin/bash

#### container list
```bash
$ docker ps -a
```

#### container stop/start/restart
```bash
$ docker stop/start/restart [container-id]
```

#### container remove (rm)
```bash
$ docker rm [OPTIONS] CONTAINER [CONTAINER...]
```

> 중지된 컨테이너 ID를 가져와서 한번에 삭제
> $ docker rm -v $(docker ps -a -q -f status=exited)

## 3. Network

####  Docker0 inteface
$ ifconfig docker0
```
docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:2cff:fe43:7a  prefixlen 64  scopeid 0x20<link>
...
```

> virtual ethernet bridge
- container가 통신하기 위한 가상 linux bridge
(bridge는 기본적으로 L2 통신 기반이며, container가 외부로 통신할 때는 무조건 docker0 interface를 지나야 한다.)
- container가 생성되면 이 bridge에 container의 interface가 binding
(container가 running 될때마다 vethXXXX 라는 이름의 interface가 attach)

docker0 의 IP는 자동으로 172.17.xxx.1로 설정되며, subnet mask는 255.255.0.0 즉, 172.17.0.0/16 으로 설정된다.
(subnet 정보는 container가 할당 받게 될 IP의 range를 결정한다. 즉, 모든 container는 172.17.xxx.xxx 대역에서 IP를 할당 받는다.)

#### Docker container IP 확인
$ docker inspect -f "{{ .NetworkSettings.IPAddress }}" [CONTAINER_ID or NAME]
> $ docker  inspect [CONTAINER_ID or NAME]

#### Network 목록 조회
$ docker network ls

#### Network 구성 조회 (구성 컨테이너/IP)
$ docker network inspect [NETWORK ID or NAME]

## 4. Dockerize

#### Dockerfile 기본 명령어

##### FROM
```
FROM <image>:<tag>
```
> FROM ubuntu:16.04
> 
베이스 이미지를 지정합니다. 반드시 지정해야 하며 어떤 이미지도 베이스 이미지가 될 수 있습니다. tag는 될 수 있으면 latest(기본값)보다 구체적인 버전(16.04등)을 지정하는 것이 좋습니다. 이미 만들어진 다양한 베이스 이미지는  [Docker hub](https://hub.docker.com/explore/)에서 확인할 수 있습니다.

##### MAINTAINER
```
MAINTAINER <name>
```
> MAINTAINER sjlee@enliple.com
> 
Dockerfile을 관리하는 사람의 이름 또는 이메일 정보를 적습니다. 빌드에 딱히 영향을 주지는 않습니다.

##### COPY
```
COPY <src>... <dest>
```
> COPY . /usr/src/app
> 
파일이나 디렉토리를 이미지로 복사합니다. 일반적으로 소스를 복사하는 데 사용합니다.  `target`디렉토리가 없다면 자동으로 생성합니다.

#### ADD
```
ADD <src>... <dest>
```
> ADD . /usr/src/app
> 
`COPY`명령어와 매우 유사하나 몇가지 추가 기능이 있습니다.  `src`에 파일 대신 URL을 입력할 수 있고  `src`에 압축 파일을 입력하는 경우 자동으로 압축을 해제하면서 복사됩니다.

##### RUN
```
RUN <command>
RUN ["executable", "param1", "param2"]
```
> RUN bundle install
> 
가장 많이 사용하는 구문입니다. 명령어를 그대로 실행합니다. 내부적으로  `/bin/sh -c`  뒤에 명령어를 실행하는 방식입니다.

##### CMD
```
CMD ["executable","param1","param2"]
CMD command param1 param2
```
> CMD bundle exec ruby app.rb
> 
도커 컨테이너가 실행되었을 때 실행되는 명령어를 정의합니다. 빌드할 때는 실행되지 않으며 여러 개의  `CMD`가 존재할 경우 가장 마지막  `CMD`만 실행됩니다. 한꺼번에 여러 개의 프로그램을 실행하고 싶은 경우에는  `run.sh`파일을 작성하여 데몬으로 실행하거나  [supervisord](http://supervisord.org/)나  [forego](https://github.com/ddollar/forego)와 같은 여러 개의 프로그램을 실행하는 프로그램을 사용합니다.

##### WORKDIR
```
WORKDIR /path/to/workdir
```

RUN, CMD, ADD, COPY등이 이루어질 기본 디렉토리를 설정합니다. 각 명령어의 현재 디렉토리는 한 줄 한 줄마다 초기화되기 때문에  `RUN cd /path`를 하더라도 다음 명령어에선 다시 위치가 초기화 됩니다. 같은 디렉토리에서 계속 작업하기 위해서  `WORKDIR`을 사용합니다.

##### EXPOSE
```
EXPOSE <port> [<port>...]
```
> EXPOSE 4567
> 
도커 컨테이너가 실행되었을 때 요청을 기다리고 있는(Listen) 포트를 지정합니다. 여러개의 포트를 지정할 수 있습니다.

##### VOLUME
```
VOLUME ["/data"]
```
> VOLUME ["/data"]
> 
컨테이너 외부에 파일시스템을 마운트 할 때 사용합니다. 반드시 지정하지 않아도 마운트 할 수 있지만, 기본적으로 지정하는 것이 좋습니다.

##### ENV
```
ENV <key> <value>
ENV <key>=<value> ...
```
> ENV DB_URL mysql
> 
컨테이너에서 사용할 환경변수를 지정합니다. 컨테이너를 실행할 때  `-e`옵션을 사용하면 기존 값을 오버라이딩 하게 됩니다.

#### Build Docker Image

##### dockerize file
$ vi Dockerfile
```
FROM scratch
COPY
RUN
CMD 
```

##### build image
$ docker build --help
```
Usage:	docker build [OPTIONS] PATH | URL | -

Build an image from a Dockerfile

Options:
      --add-host list           Add a custom host-to-IP mapping (host:ip)
      --build-arg list          Set build-time variables
      --cache-from strings      Images to consider as cache sources
      --cgroup-parent string    Optional parent cgroup for the container
      --compress                Compress the build context using gzip
      --cpu-period int          Limit the CPU CFS (Completely Fair Scheduler) period
      --cpu-quota int           Limit the CPU CFS (Completely Fair Scheduler) quota
  -c, --cpu-shares int          CPU shares (relative weight)
      --cpuset-cpus string      CPUs in which to allow execution (0-3, 0,1)
      --cpuset-mems string      MEMs in which to allow execution (0-3, 0,1)
      --disable-content-trust   Skip image verification (default true)
  -f, --file string             Name of the Dockerfile (Default is 'PATH/Dockerfile')
      --force-rm                Always remove intermediate containers
      --iidfile string          Write the image ID to the file
      --isolation string        Container isolation technology
      --label list              Set metadata for an image
  -m, --memory bytes            Memory limit
      --memory-swap bytes       Swap limit equal to memory plus swap: '-1' to enable unlimited swap
      --network string          Set the networking mode for the RUN instructions during build (default "default")
      --no-cache                Do not use cache when building the image
      --pull                    Always attempt to pull a newer version of the image
  -q, --quiet                   Suppress the build output and print image ID on success
      --rm                      Remove intermediate containers after a successful build (default true)
      --security-opt strings    Security options
      --shm-size bytes          Size of /dev/shm
  -t, --tag list                Name and optionally a tag in the 'name:tag' format
      --target string           Set the target build stage to build.
      --ulimit ulimit           Ulimit options (default [])
```

$ sudo docker build -t dev_img:latest .

#### Create Docker Image from Container

##### 컨테이너 변경 내용 확인
$ docker diff xxx_container

#### 이미지 생성
$ docker commit --help
```
Usage:	docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

Create a new image from a container's changes
Dockerfile
Options:
  -a, --author string    Author (e.g., "John Hannibal Smith <hannibal@a-team.com>")
  -c, --change list      Apply Dockerfile instruction to the created image
  -m, --message string   Commit message
  -p, --pause            Pause container during commit (default true)
```

$ docker commit -a "sjlee@enliple.com" -m "xxx 컨테이너 이미지 생성" xxx_container mobon/xxx:0.2

#### Remove Docker Image
$ docker rmi hello
```
Untagged: hello:latest
Deleted: sha256:359f84fca603062a5115202f6ab06874c40de46c8c6c56ac11b3d82cb5faa4dd
```
>```
>Error response from daemon: conflict: unable to remove repository reference "hello" (must force) - container 5f021e7ab9b1 is using its referenced image d07e04ed5fdd
>```
>이미지를 사용하는 컨테이너가 있으면 위 메세지를 리턴한다.
>
>##### 컨테이너 조회
>$ docker ps -a
>
>##### 컨테이너 삭제
>$ docker rm 5f021e7ab9b1

`Remove unused images`  
$ docker image prune -a  
> $ docker image rm -f $(docker image ls -aq)

#### Docker Image 

##### 로그인
$ sudo docker login
```
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: mobon
Password:
Login Succeeded
```

##### Push
$ docker push --help
```
Usage:	docker push [OPTIONS] NAME[:TAG]

Push an image or a repository to a registry

Options:
      --disable-content-trust   Skip image signing (default true)
arch@arch-X230:~$ 
```

$ sudo docker push mobon/xxx:0.2

> docker hub 등에 이미지를 push하기 위해 서는 local과 같은 이름을 사용해야한다.
> 이를 위해 tag를 사용하여 다른 이름을 지정(alias)할 수 있다.

$ sudo docker image tag xxx:0.2 mobon/xxx:0.2

##### docker hub에서 image 확인
https://hub.docker.com/r/mobon/xxx/

##### docker 다른 계정(root) 으로 컨테이너 접근
$ docker exec -it -u root -w /root redis.5 /bin/bash  
> starting container process caused "chdir to cwd (\"/apps\") set in config.json failed: permission denied": unknown

## 9. Appendix

#### reference site

* dockerhub  
https://hub.docker.com/

+ 초보를 위한 도커 안내서 - 설치하고 컨테이너 실행하기  
https://subicura.com/2017/01/19/docker-guide-for-beginners-2.html

+ 초보를 위한 도커 안내서 - 이미지 만들고 배포하기SERIES 3/3  
https://subicura.com/2017/02/10/docker-guide-for-beginners-create-image-and-deploy.html

+ Docker Network 구조(1) - docker0와 container network 구조  
https://bluese05.tistory.com/15

+ Docker 설치 후 이미지 보관 디렉토리 변경  
http://dveamer.github.io/backend/DockerImageDirectory.html

+ docker 이미지 저장 경로 변경하기  
https://doogle.link/docker-%EC%9D%B4%EB%AF%B8%EC%A7%80-%EC%A0%80%EC%9E%A5-%EA%B2%BD%EB%A1%9C-%EB%B3%80%EA%B2%BD%ED%95%98%EA%B8%B0/

+ Docker: 나의 첫 Docker Image  
https://www.sauru.so/blog/build-my-first-docker-image/

+ docker 컨테이너 이미지 생성  
http://forum.falinux.com/zbxe/index.php?document_srl=806457

+ Docker container IP 확인  
https://bluese05.tistory.com/36

+ Docker – change container configuration in 4 ways  
https://bobcares.com/blog/docker-change-container-configuration/

- Docker / 컨테이너 제거, 삭제 및 정리
https://riptutorial.com/ko/docker/example/3955/%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88-%EC%A0%9C%EA%B1%B0--%EC%82%AD%EC%A0%9C-%EB%B0%8F-%EC%A0%95%EB%A6%AC
