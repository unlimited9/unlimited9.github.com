# Install and setup : gradle

>Created 목요일 30 11월 2017  

build tool

## Pre-installation setup

#### creating required operating system group and user
[Create operating system group and user](../../TA/system/management.account.n.group.md)

#### install dependency packages

#### creating base directory
[Create operating system drectory](../../TA/system/management.directory.md)

#### firewall configuration

## installation setup : app

#### change account
$ su - app

#### creating application directory
$ mkdir -p /apps/gradle

$ cd /apps/install

#### decompress manual
`download`  
>https://gradle.org/releases/

`unpack the distribution`  
$ unzip -d /apps/gradle /apps/install/gradle-6.0.1-bin.zip  

`configure your system environment`  
$ export PATH=$PATH:/opt/gradle/gradle-6.0.1/bin

#### package install : case of ubuntu
`저장소 추가`  
$ sudo add-apt-repository ppa:cwchien/gradle  
$ sudo apt-get update

`gradle  설치`  
$ apt-get install gradle

#### Verify your installation
`gradle  version`  
$ gradle -v

## trouble-shooting

#### add-apt-repository: command not found
$ sudo add-apt-repository ppa:cwchien/gradle  
```
bash: add-apt-repository: command not found
```
$ sudo apt-get update
$ sudo apt-get install software-properties-common  
>Reading package lists... Done  
Building dependency tree  
Reading state information... Done  
E: Unable to locate package software-properties-common  
$ apt-get update  

## Appendix

#### reference site

* Gradle build tool : Installation  
https://gradle.org/install/


