# Install and setup JAVA

>Created 목요일 30 11월 2017  
Java Development Kit

# Installation/Setup/Configuration

[Script install and setup Java](install.n.setup.script.md)

## 1. Pre-installation setup

### A. creating required operating system group and user

#### create the os application group(typically, app)
$ groupadd -g 3000 app

#### create the software unified account (typically, app)
$ useradd -d /apps -g 3000 -m -u 3000 -s /bin/bash app
$ passwd app

### B. install dependency packages

### C. creating base directory

#### 디렉토리 생성
~~$ mkdir -p /apps~~
$ mkdir -p /apps/install

#### 소유권 변경
$ chown -R app:app /apps

### D. firewall configuration

## 2. installation setup : app

### A. change account

$ su - app

### B. creating application directory

#### make directory
$ mkdir -p /apps/jdk

### C. download

#### https://jdk.java.net/ - JDK 11.0.2 General-Availability Release
$ curl -O https://download.java.net/java/GA/jdk11/9/GPL/openjdk-11.0.2_linux-x64_bin.tar.gz -P /apps/install
>~~$ wget https://download.java.net/java/GA/jdk11/9/GPL/openjdk-11.0.2_linux-x64_bin.tar.gz -P /apps/install~~

>#### http://oracle.com - Java SE Development Kit 11.0.2(LTS)
>>for accept header and license
>
>~~$ curl -LO -H "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/11.0.2+9/f51449fcd52f4d52b93a989c5c56ed3c/jdk-11.0.2_linux-x64_bin.tar.gz -P /apps/install~~
>>~~$ curl -O -v -j -k -L -H "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/11.0.2+9/f51449fcd52f4d52b93a989c5c56ed3c/jdk-11.0.2_linux-x64_bin.tar.gz -P /apps/install~~
>
>~~$ wget http://download.oracle.com/otn-pub/java/jdk/11.0.2+9/f51449fcd52f4d52b93a989c5c56ed3c/jdk-11.0.2_linux-x64_bin.tar.gz -H --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" -P /apps/install~~

### D. install

#### decompress tarball
$ tar -zxvf /apps/install/openjdk-11.0.2_linux-x64_bin.tar.gz

#### copy application directory
$ cp -R /apps/install/jdk-11.0.2 /apps/jdk/11.0.2

#### package install
##### cenos
~~$ sudo yum install java-11-openjdk-devel~~
##### ubuntu
~~$ sudo apt-get install openjdk-11-jdk~~

## 3. post-installation setup

### A. set environment variable

> add JAVA_HOME environment variable to /etc/profile file or $HOME/.bash_profile file package install path : /usr/lib/jvm

$ su - 
$ cat > /etc/profile.d/java11.sh <<EOF  
export JAVA_HOME=$(dirname $(dirname $(readlink $(readlink $(which javac)))))  
export PATH=\$PATH:\$JAVA_HOME/bin  
export CLASSPATH=.:\$JAVA_HOME/jre/lib:\$JAVA_HOME/lib:\$JAVA_HOME/lib/tools.jar  
EOF

$ source /etc/profile.d/java11.sh

>$ vi /etc/profile.d/jdk11.sh
>```
># create new
>export JAVA_HOME=/apps/jdk/11.0.2
>export PATH=$PATH:$JAVA_HOME/bin
>```
>$ source /etc/profile.d/jdk11.sh
>>$ source /etc/profile

>$ vi /etc/profile
>```
>...
>export JAVA_HOME="/apps/jdk/11.0.2"
>export PATH=$PATH:$JAVA_HOME/bin:
>...
>```
>$ source /etc/profile

### B. Selecting Java Versions with Alternatives

##### centos
$ sudo alternatives --list

#### swap java versions
$ sudo alternatives --config java  
>  java with [+] is currently on use

$ sudo alternatives --install java  
> $ alternatives --install /usr/bin/java java $JAVA_HOME/bin/java 1  
> $ update-alternatives --install /usr/bin/java java $JAVA_HOME/bin/java 1

also do the same for :  javac, jar, javaws, Java Browser (Mozilla) Plugin ...  
> Install javac only if you installed JDK (Java Development Kit) package  
> $ alternatives --install /usr/bin/javac javac $JAVA_HOME/bin/javac 1  
> $ alternatives --install /usr/bin/jar jar $JAVA_HOME/bin/jar 1  
> $ alternatives --install /usr/bin/javaws javaws $JAVA_HOME/jre/bin/javaws 1  
> $ alternatives --install /usr/lib/mozilla/plugins/libjavaplugin.so libjavaplugin.so $JAVA_HOME/jre/lib/i386/libnpjp2.so 1  
> $ alternatives --install /usr/lib64/mozilla/plugins/libjavaplugin.so libjavaplugin.so.x86_64 $JAVA_HOME/jre/lib/amd64/libnpjp2.so 1

##### ubuntu
$ sudo update-java-alternatives -l
```
java-1.11.0-openjdk-amd64      1111       /usr/lib/jvm/java-1.11.0-openjdk-amd64
java-1.8.0-openjdk-amd64       1081       /usr/lib/jvm/java-1.8.0-openjdk-amd64
```

$ sudo update-java-alternatives -s java-1.8.0-openjdk-amd64

## 9. Appendix

### reference site

- How to Install Java 11 (OpenJDK 11) on RHEL 8  
https://computingforgeeks.com/how-to-install-java-11-openjdk-11-on-rhel-8/

- How to Install Java 11 on Ubuntu 18.04 /16.04 / Debian 9  
https://computingforgeeks.com/how-to-install-java-11-on-ubuntu-18-04-16-04-debian-9/

- How to Install Java 11 on CentOS 7 / Fedora 29 / Fedora 28  
https://computingforgeeks.com/how-to-install-java-11-on-centos-7-fedora-29-fedora-28/
