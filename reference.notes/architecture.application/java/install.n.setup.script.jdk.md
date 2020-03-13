## Script to install and setup java

su - app

cd /apps/install  
curl -O https://download.java.net/java/GA/jdk12.0.2/e482c34c86bd4bf8b56c0b35558996b9/10/GPL/openjdk-12.0.2_linux-x64_bin.tar.gz  
tar xvf openjdk-12.0.2_linux-x64_bin.tar.gz

mkdir /apps/jdk  
mv /apps/install/jdk-12.0.2 /apps/jdk/12.0.2 

su -  
vi /etc/profile.d/jdk.sh
```
export JAVA_HOME=/apps/jdk/12.0.2
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/jre/lib:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```
source /etc/profile

su - app  
java -version
```
openjdk version "12.0.2" 2019-07-16
OpenJDK Runtime Environment (build 12.0.2+10)
OpenJDK 64-Bit Server VM (build 12.0.2+10, mixed mode, sharing)
```
