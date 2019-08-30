# Install and setup Docker

>Created 요일 20 월 2017  
Installation/Setup/Configuration independent container platform

## 1. Pre-installation setup

#### A. creating required operating system group and user
[Create operating system group and user](/reference.notes/TA/system/management.account.n.group.md)

#### B. install dependency packages

#### C. creating base directory
[Create operating system drectory](/reference.notes/TA/system/management.directory.md)

#### D. firewall configuration

## 2. installation setup

#### A. change account
$ su - app

#### B. creating application directory
$ mkdir -p /data/docker

>$ mkdir -p /apps/docker
$ mkdir -p /logs/docker

#### C. download

#### D. install

>`dependency packages`  
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2 --downloadonly --downloaddir=/apps/install/rpms/yum-utils  
$ sudo rpm -Uvh --replacepkgs /apps/install/rpms/yum-utils/*rpm

>`docker packages`  
$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo  
$ sudo yum install -y docker-ce docker-ce-cli containerd.io --downloadonly --downloaddir=/apps/install/rpms/docker  
$ sudo rpm -Uvh --replacepkgs /apps/install/rpms/docker/*rpm

`excute install script`
```bash
$ curl -fsSL https://get.docker.com/ | sudo sh
```
>```bash
>$ sudo wget -qO- https://get.docker.com/ | sh
>```

>`docker-ce package dependency error`  
>Error: Package: 3:docker-ce-18.09.7-3.el7.x86_64 (docker-ce-stable)
>           Requires: container-selinux >= 2.9
>
>$ cd /apps/install  
>$ curl -O http://mirror.centos.org/centos/7/extras/x86_64/Packages/container-selinux-2.99-1.el7_6.noarch.rpm  
>$ sudo rpm -Uvh container-selinux-2.99-1.el7_6.noarch.rpm
>
>`container-selinux package dependency error`  
>경고: container-selinux-2.99-1.el7_6.noarch.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY  
>오류: Failed dependencies:  
>	policycoreutils-python is needed by container-selinux-2:2.99-1.el7_6.noarch
>
>$ sudo yum install -y policycoreutils-python

`add account(app) to group(docker)` : 도커 실행 권한 주기
```bash
$ sudo usermod -aG docker app
```
> 현재 계정을 도커그룹에 추가하여 권한주기
>```bash
>$ sudo usermod -aG docker $USER 
>```

```bash
$ docker -v
$ docker version
```
docker daemon이 실행되지 않았거나 권한이 없을 경우 에러 발생
>Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.39/version: dial unix /var/run/docker.sock: connect: permission denied
>Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
>
>`package install`
>* Ubuntu  
> $ sudo apt-get update  
> $ sudo apt-get install docker.io  
>* CentOS 6  
> $ sudo yum repolist  
> 레파지토리 추가  
> $ sudo yum install http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm  
>   >$ sudo rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm  
>
>   레파지토리 조회  
>   $ sudo yum repolist  
>   $ sudo rpm -qa | grep epel  
>
>   레파지토리 삭제  
>   $ sudo rpm -ev epel-release-6-8.noarch  
>
>   도커 설치  
>   $ sudo yum install docker-io
>* CentOS 7
>   >$ sudo yum install -y yum-utils
>   >$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo sudo yum makecache fast
>
>   $ sudo yum install -y docker-ce  
>   $ sudo systemctl start docker  
>   $ sudo systemctl enable docker  
>   $ sudo gpasswd -a app docker  
>   >Usage: gpasswd [option] GROUP
>   >
>   >Options:  
`  -a, --add USER                add USER to GROUP`  
`  -d, --delete USER             remove USER from GROUP`  
  -h, --help                    display this help message and exit  
  -Q, --root CHROOT_DIR         directory to chroot into  
`  -r, --delete-password         remove the GROUP's password`  
  -R, --restrict                restrict access to GROUP to its members  
  -M, --members USER,...        set the list of members of GROUP  
  -A, --administrators ADMIN,...

#### E. configure

#### F. run application
> systemd

`active service`  
$ sudo systemctl enable docker
> sudo chkconfig docker on

`start service`  
$ sudo systemctl start docker
>$ sudo systemctl restart docker

`check service`  
$ sudo systemctl status docker

`stop service`  
$ sudo systemctl stop docker

>`도커 데몬 시작`  
>$ service docker start
>
>`서버 재시작시에도 도커 데몬이 재시작 되도록 설정`  
>$ chkconfig docker on

## 3. post-installation setup

#### A. create shell script


## 9. Appendix

#### reference site

* Docker Documentation  
https://docs.docker.com/

* 도커 Docker 기초 확실히 다지기  
https://futurecreator.github.io/2018/11/16/docker-container-basics/

* Docker 기본 사용법  
http://pyrasis.com/Docker/Docker-HOWTO#section-5

* 우분투에서 docker 설치 방법  
https://hiseon.me/2018/02/19/install-docker/

* [Docker] Linux 환경에 Docker 설치하기 (Ubuntu, REHL, CentOS)  
https://dololak.tistory.com/354

* Docker 설치하기 (RedHat/CentOS)  
http://snowdeer.github.io/docker/2018/01/02/install-docker-on-redhat-and-centos/

* Ubuntu 18.04 설치 #2-3 Install Tensorflow with Docker  
https://eungbean.github.io/2018/11/09/Ubuntu-Installation2-3/
