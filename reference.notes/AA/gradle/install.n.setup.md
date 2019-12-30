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

#### download

#### package install : case of ubuntu
`저장소 추가`  
$ sudo add-apt-repository ppa:cwchien/gradle  
$ sudo apt-get update

`gradle  설치`  
$ apt-get install gradle

`gradle  version`  
$ gradle -v

## trouble-shooting

#### add-apt-repository: command not found
$ sudo add-apt-repository ppa:cwchien/gradle  
```
bash: add-apt-repository: command not found
```

$ sudo apt-get install software-properties-common  
>Reading package lists... Done  
Building dependency tree  
Reading state information... Done  
E: Unable to locate package software-properties-common  
$ apt-get update  

## 9. AppendixDistributed Coordination Service

#### reference site

* referer...


