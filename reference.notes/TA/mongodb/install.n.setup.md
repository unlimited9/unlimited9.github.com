# 10.install.n setup : mongoDB

>Created 월요일 20 11월 2017
NoSQL(Document) Database : Server.setting - Failover and high availability

# Installation/Setup/Configuration

## 1. Pre-installation setup

```
#### Default(Service) MongoDB Port
27017(2612) The default port for mongod and mongos instances. You can change this port with port or --port.
27018(2622) The default port when running with --shardsvr runtime operation or the shardsvr value for the clusterRole setting in a configuration file.
27019(2632) The default port when running with --configsvr runtime operation or the configsvr value for the clusterRole setting in a configuration file.
28017(3612) The default port for the web status page. The web status page is always accessible at a port number that is 1000 greater than the port determined by port.
```

### A. creating required operating system group and user

> 어플리케이션 통합 관리로 계정 권한/그룹 분리를 하지 않음

#### create the os application group(typically, app)  
$ groupadd -g 3000 app

#### create the os dba group(typically, dba)
~~$ groupadd -g 2000 dba~~

> 어플리케이션 단위 관리 주체가 다르지 않고 권한을 나눌 필요가 없어 통합 계정을 생성하여 관리
> ~~어플리케이션 단위 관리 주체가 다르다면 어플리케이션별 계정을 생성하여 생성하여 관리~~

#### create the software unified account (typically, app)
$ useradd -d /apps -g 3000 -m -u 3000 -s /bin/bash app
$ passwd app

#### create the mongodb software owner (typically, mongodb)  
~~$ useradd -d /apps/mongodb -g 2000 -m -u 2020 -s /bin/bash mongodb
$ passwd mongodb~~

### B. install dependency packages

### C. creating base directory

#### 디렉토리 생성
~~$ mkdir -p /apps~~
$ mkdir -p /apps/install
$ mkdir -p /pgms
$ mkdir -p /data
$ mkdir -p /logs

#### 소유권 변경
$ chown -R app:app /apps
$ chown -R app:app /pgms
$ chown -R app:app /data
$ chown -R app:app /logs

### D. firewall configuration

$ vi /etc/sysconfig/iptables
```
...
# mongodb
-A INPUT -m state --state NEW -m tcp -p tcp --dport 2612 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 2622 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 2623 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 2624 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 2632 -j ACCEPT
...
```

#### restart iptalbes service  
$ service iptables restart

$ iptables -nL

## 2. installation setup

### A. change account

$ su - app

### B. creating application directory

#### make directory
$ mkdir -p /apps/mongodb
$ mkdir -p /data/mongodb
$ mkdir -p /logs/mongodb

### C. download

#### https://www.mongodb.com/download-center
$ curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-4.0.8.tgz -P /apps/install
~~$ wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-4.0.8.tgz -P /apps/install~~

### D. install

#### decompress tarball  
$ cd /apps/install
$ tar -xvf mongodb-linux-x86_64-4.0.8.tgz

#### copy application directory  
$ cp -R /apps/install/mongodb-linux-x86_64-4.0.8 /apps/mongodb/4.0.8
~~$ mv /apps/install/mongodb-linux-x86_64-4.0.8 /apps/mongodb/4.0.8~~

### E. configure

### F. run application

#### start process
$ /apps/mongodb/4.0.8/bin/mongod --dbpath /data/mongodb/es --logpath /logs/mongodb/es.log --port 2612 --rest --fork

#### check process and port
$ ps -ef | grep mongod
```
...
/apps/mongodb/4.0.8/bin/mongod --dbpath /data/mongodb/es --logpath /logs/mongodb/es.log --port 2612 --rest --fork
...
```

$ netstat -na | grep LISTEN
```
...
tcp        0      0 0.0.0.0:2612           0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:3612           0.0.0.0:*               LISTEN
...
```

#### stop process
$ /apps/mongodb/4.0.8/bin/mongod --dbpath /data/mongodb/es --shutdown

>or mongo shell

~~$ /apps/mongodb/4.0.8/bin/mongo~~
```
> use admin
switched to db admin
> db.shutdownServer()
server should be down...
2017-10-06T04:18:25.733+0900 I NETWORK  [thread1] trying reconnect to 172.20.0.104:2612 (172.20.0.104) failed
2017-10-06T04:18:25.734+0900 W NETWORK  [thread1] Failed to connect to 172.20.0.104:2612, in(checking socket for error after poll), reason: Connection refused
2017-10-06T04:18:25.734+0900 I NETWORK  [thread1] reconnect 172.20.0.104:2612 (172.20.0.104) failed failed 
> exit
bye
```

## 2.5. Setting - sharding/replication(replica)

#### nodes
192.168.104.40  mongos01.hajimaro.com 
192.168.104.41  mongodb01.hajimaro.com 
192.168.104.42  mongodb02.hajimaro.com 
192.168.104.43  mongodb03.hajimaro.com 

### A. replication config server

> instance name : <mongdb_name><replication_set_index>_<sharding_index>

#### create security keyFile
$ mkdir -p /apps/mongodb/instances/esconfig01/config
$ openssl rand -base64 756 > /apps/mongodb/instances/esconfig01/config/mongodb-keyfile
$ chmod 400 /apps/mongodb/instances/esconfig01/config/mongodb-keyfile

>config 서버들의 리플리케이션 셋 구성을 위해  security keyFile에 같은 key를 사용해야 한다.

#### configuration
$ vi /apps/mongodb/instances/esconfig01/config/mongodb.yaml
```yaml
systemLog:
  destination: file
  logAppend: true
  path: "/logs/mongodb/esconfig01.log"
storage:
  dbPath: "/data/mongodb/esconfig01"
processManagement:
   fork: true
#   pidFilePath: "/tmp/mongod.pid"
net:
  port: 2632
  bindIp: 0.0.0.0
sharding:
  clusterRole: configsvr
replication:
  replSetName: rs_esconfig01
security:
  keyFile: "/apps/mongodb/instances/esconfig01/config/mongodb-keyfile"
```

$ mkdir -p /data/mongodb/esconfig01

#### start process
$ /apps/mongodb/4.0.8/bin/mongod --config /apps/mongodb/instances/esconfig01/config/mongodb.yaml

> 각 configuration server에서 configuration, start process 수행

> 1 node > multi instance
>```bash
>$ /apps/mongodb/4.0.8/bin/mongod --config /apps/mongodb/instances/esconfig01_01/config/mongodb.yaml
>$ /apps/mongodb/4.0.8/bin/mongod --config /apps/mongodb/instances/esconfig01_02/config/mongodb.yaml
>$ /apps/mongodb/4.0.8/bin/mongod --config /apps/mongodb/instances/esconfig01_03/config/mongodb.yaml
>```

#### add user
> security key나 옵션을 주지 않고 기동해야 계정을 생성할 수 있어 임시로 mongo config를 사용하지 않고 단일 노드로 띄운다.

$ /apps/mongodb/4.0.8/bin/mongod --port 2632 --dbpath /data/mongodb/esconfig01
$ /apps/mongodb/4.0.8/bin/mongo localhost:2632/admin
```
> db.createUser({ "user" : "admin", "pwd" : "passwd", roles : [{ "role" : "root", "db" : "admin" }]})
> show users
```
$ /apps/mongodb/4.0.8/bin/mongod --port 2632 --dbpath /data/mongodb/esdb01 --shutdown

#### (re)start process
$ /apps/mongodb/4.0.8/bin/mongod --config /apps/mongodb/instances/esconfig01/config/mongodb.yaml

> 각 configuration server에서 configuration, start process 수행
> $ /apps/mongodb/4.0.8/bin/mongod --config /apps/mongodb/instances/esconfig01_02/config/mongodb.yaml
$ /apps/mongodb/4.0.8/bin/mongod --config /apps/mongodb/instances/esconfig01_03/config/mongodb.yaml

#### replication initiate
$ /apps/mongodb/4.0.8/bin/mongo 192.168.104.41:2632/admin
```
> db.auth("admin", "passwd")
> rs.initiate({"_id" : "rs_esconfig01", "configsvr" : true, "members" : [{"_id" : 1, "host" : "192.168.104.41:2632", "priority" : 1}, {"_id" : 2, "host" : "192.168.104.42:2632", "priority" : 1}, {"_id" : 3, "host" : "192.168.104.43:2632", "priority" : 1}]})
> rs.status()
```
> 개발서버 1기에 설치 시
> $ /apps/mongodb/4.0.8/bin/mongo 172.20.0.104:12632/admin
> ```
>> rs.initiate({"_id" : "rs_esconfig01", "configsvr" : true, "members" : [{"_id" : 1, "host" : "172.20.0.104:12632", "priority" : 1}, {"_id" : 2, "host" : "172.20.0.104:12633", "priority" : 1}, {"_id" : 3, "host" : "172.20.0.104:12634", "priority" : 1}]})
>> rs.status()
> ```

#### stop process
$ /apps/mongodb/4.0.8/bin/mongod --config /apps/mongodb/instances/esconfig01/config/mongodb.yaml --shutdown


### B. replication mongodb server

#### copy security keyFile from config server
$ mkdir -p /apps/mongodb/instances/esdb01/config
$ cp /apps/mongodb/instances/esconfig01/config/mongodb-keyfile /apps/mongodb/instances/esdb01/config/mongodb-keyfile

> 샤딩을 위해 config서버에 security keyFile과 같은 key를 사용해야 하며, 리플리케이션 셋 구성을 위한 서버들도 security keyFile에 같은 key를 사용해야 한다.

#### configuration
$ vi /apps/mongodb/instances/esdb01/config/mongodb.yaml
```yaml
systemLog:
  destination: file
  path: "/logs/mongodb/esdb01.log"
  logAppend: true
  logRotate: rename
storage:
  engine: wiredTiger
  wiredTiger:
    engineConfig:
      journalCompressor: snappy
    collectionConfig:
      blockCompressor: snappy
    indexConfig:
      prefixCompression: true
  dbPath: "/data/mongodb/esdb01"
  journal:
    enabled: true
    commitIntervalMs: 300
processManagement:
  fork: true
#   pidFilePath: "/tmp/mongod.pid"
net:
  port: 2622
  bindIp: 0.0.0.0
  maxIncomingConnections: 1000
  unixDomainSocket:
    enabled: false
sharding:
  clusterRole: shardsvr
replication:
   oplogSizeMB: 1024
   replSetName: rs_esdb01
   enableMajorityReadConcern: true
security:
  keyFile: "/apps/mongodb/instances/esdb01/config/mongodb-keyfile"
  authorization: enabled
```

$ mkdir -p /data/mongodb/esdb01
$ mkdir -p /data/mongodb/esdb02
$ mkdir -p /data/mongodb/esdb03

#### add user
> security key나 옵션을 주지 않고 기동해야 계정을 생성할 수 있어 임시로 mongo config를 사용하지 않고 단일 노드로 띄운다.

##### rs_esdb01
$ /apps/mongodb/4.0.8/bin/mongod --port 2622 --dbpath /data/mongodb/esdb01
$ /apps/mongodb/4.0.8/bin/mongo localhost:2622/admin
```
> db.createUser({ "user" : "admin", "pwd" : "passwd", roles : [{ "role" : "root", "db" : "admin" }]})
> show users
```
$ /apps/mongodb/4.0.8/bin/mongod --port 2622 --dbpath /data/mongodb/esdb01 --shutdown

>##### rs_esdb02
>$ /apps/mongodb/4.0.8/bin/mongod --port 2622 --dbpath /data/mongodb/esdb02_01
>$ /apps/mongodb/4.0.8/bin/mongo localhost:2622/admin
>```
>> db.createUser({ "user" : "admin", "pwd" : "passwd", roles : [{ "role" : "root", "db" : "admin" }]})
>> show users
>```
>$ /apps/mongodb/4.0.8/bin/mongod --port 2622 --dbpath /data/mongodb/esdb02_01 --shutdown

>##### rs_esdb03
>$ /apps/mongodb/4.0.8/bin/mongod --port 2622 --dbpath /data/mongodb/esdb03_01
>$ /apps/mongodb/4.0.8/bin/mongo localhost:2622/admin
>```
>> db.createUser({ "user" : "admin", "pwd" : "passwd", roles : [{ "role" : "root", "db" : "admin" }]})
>> show users
>```
>$ /apps/mongodb/4.0.8/bin/mongod --port 2622 --dbpath /data/mongodb/esdb03_01 --shutdown

#### start process
$ /apps/mongodb/4.0.8/bin/mongod --config /apps/mongodb/instances/esdb01/config/mongodb.yaml

> 각 replication server(node)에서 configuration, start process 수행
> 계정추가도 각 노드마다 해줘야 한다.(계정은 리플리케이션 설정을 하면 복제 되는듯...)

#### replication initiate

##### rs_esdb01
$ /apps/mongodb/4.0.8/bin/mongo 192.168.104.41:2622/admin
```
> rs.initiate({"_id" : "rs_esdb01", "members" : [{"_id" : 1, "host" : "192.168.104.41:2622", "priority" : 1}, {"_id" : 2, "host" : "192.168.104.42:2623", "priority" : 1}, {"_id" : 3, "host" : "192.168.104.43:2624", "priority" : 1}]})
> rs.status()
```
> 개발서버 1기에 설치 시
$ /apps/mongodb/4.0.8/bin/mongo 172.20.0.104:12622/admin
>```
>> rs.initiate({"_id" : "rs_esdb01", "members" : [{"_id" : 1, "host" : "172.20.0.104:12622", "priority" : 1}, {"_id" : 2, "host" : "172.20.0.104:12623", "priority" : 1}, {"_id" : 3, "host" : "172.20.0.104:12624", "priority" : 1}]})
>> rs.status()
>```

or
~~$ /apps/mongodb/4.0.8/bin/mongo 192.168.104.41:2622/admin~~
```
> rsconf = {"_id" : "rs_esdb01", "members" : [{"_id" : 1, "host" : "192.168.104.41:2622", "priority" : 1}, {"_id" : 2, "host" : "192.168.104.42:2623", "priority" : 1}, {"_id" : 3, "host" : "192.168.104.43:2624", "priority" : 1}]}
> db.runCommand({"replSetInitiate" : rsconf});
```

or
~~$ /apps/mongodb/4.0.8/bin/mongo 192.168.104.41:2622/admin~~
```
> db.runCommand({"replSetInitiate" : {"_id" : "rs_esdb01", "members" : [{"_id" : 1, "host" : "192.168.104.41:2622", "priority" : 1}, {"_id" : 2, "host" : "192.168.104.42:2623", "priority" : 1}, {"_id" : 3, "host" : "192.168.104.43:2624", "priority" : 1}]}})

# replica set add
> rs.add("10.10.10.28:2625")
rs01:PRIMARY> rs.add("10.10.10.29:2626")
rs01:PRIMARY> rs.add("10.10.10.30:2627")
```

##### rs_esdb02
$ /apps/mongodb/4.0.8/bin/mongo 192.168.104.42:2622
```
> rs.initiate({"_id" : "rs_esdb02", "members" : [{"_id" : 1, "host" : "192.168.104.42:2622", "priority" : 1}, {"_id" : 2, "host" : "192.168.104.43:2623", "priority" : 1}, {"_id" : 3, "host" : "192.168.104.41:2624", "priority" : 1}]})
> rs.status()
```

> 개발서버 1기에 설치 시
$ /apps/mongodb/4.0.8/bin/mongo 172.20.0.104:22622/admin
>```
>>rs.initiate({"_id" : "rs_esdb02", "members" : [{"_id" : 1, "host" : "172.20.0.104:22622", "priority" : 1}, {"_id" : 2, "host" : "172.20.0.104:22623", "priority" : 1}, {"_id" : 3, "host" : "172.20.0.104:22624", "priority" : 1}]})
>> rs.status()
>```

##### rs_esdb03
```
$ /apps/mongodb/4.0.8/bin/mongo 192.168.104.43:2622
> rs.initiate({"_id" : "rs_esdb03", "members" : [{"_id" : 1, "host" : "192.168.104.43:2622", "priority" : 1}, {"_id" : 2, "host" : "192.168.104.41:2623", "priority" : 1}, {"_id" : 3, "host" : "192.168.104.42:2624", "priority" : 1}]})
> rs.status()
```

> 개발서버 1기에 설치 시
$ /apps/mongodb/4.0.8/bin/mongo 172.20.0.104:32622/admin
>```
>>rs.initiate({"_id" : "rs_esdb03", "members" : [{"_id" : 1, "host" : "172.20.0.104:32622", "priority" : 1}, {"_id" : 2, "host" : "172.20.0.104:32623", "priority" : 1}, {"_id" : 3, "host" : "172.20.0.104:32624", "priority" : 1}]})
>> rs.status()
>```

#### infomation
> replication 정보 조회(master)

$ /apps/mongodb/4.0.8/bin/mongo 192.168.104.41:2622/admin
```
> db.printReplicationInfo();
```

> replication 정보 조회(slave)

$ /apps/mongodb/4.0.8/bin/mongo 192.168.104.42:2623
```
> db.printSlaveReplicationInfo();
```

> 인증

$ /apps/mongodb/4.0.8/bin/mongo 192.168.104.42:2623
```
> use local
> db.addUser("repl", password);
```

### C. mongos server

#### copy security keyFile from config server
$ mkdir -p /apps/mongodb/instances/mongos01/config
$ cp /apps/mongodb/instances/esconfig01/config/mongodb-keyfile /apps/mongodb/instances/mongos01/config/mongodb-keyfile

> config서버에 security keyFile과 같은 key를 사용해야 접근 에러가 나지 않는다.

#### configuration
$ vi /apps/mongodb/instances/mongos01/config/mongos.yaml
```yaml
systemLog:
  destination: file
  logAppend: true
  path: "/logs/mongodb/mongos01.log"
processManagement:
   fork: true
#   pidFilePath: "/tmp/mongod.pid"
net:
  port: 2612
  bindIp: 0.0.0.0
sharding:
  configDB: rs_esconfig01/192.168.104.41:2632,192.168.104.42:2632,192.168.104.43:2632
security:
  keyFile: "/apps/mongodb/instances/mongos01/config/mongodb-keyfile"
```

>#### add user
> : config서버에 사용자(계정)을 추가했다면 생략
>> security key나 옵션을 주지 않고 기동해야 계정을 생성할 수 있어 임시로 mongo config를 사용하지 않고 단일 노드로 띄운다. (구성 config  서버들도 security 설정 제외하고 재기동 필요)
>
>$ /apps/mongodb/4.0.8/bin/mongos --port 2612 --configdb rs_esconfig01/192.168.104.41:2632,192.168.104.42:2632,192.168.104.43:2632
>
>> 개발서버 1기에 설치 시
>>/apps/mongodb/4.0.8/bin/mongos --port 12612 --configdb rs_esconfig01/172.20.0.104:12632,172.20.0.104:12633,172.20.0.104:12634
>
>$ /apps/mongodb/4.0.8/bin/mongo localhost:2612/admin
>```
>> db.createUser({ "user" : "admin", "pwd" : "passwd", roles : [{ "role" : "root", "db" : "admin" }]})
>```
>$ /apps/mongodb/4.0.8/bin/mongos --port 2612 --configDB rs_esconfig01/192.168.104.41:2632,192.168.104.42:2632,192.168.104.43:2632 --shutdown
>
>> 계정추가는 mongos를 여러개 띄워도 한번만 해주면 된다.

#### start process
$ /apps/mongodb/4.0.8/bin/mongos --config /apps/mongodb/instances/mongos01/config/mongos.yaml

#### stop process
$ /apps/mongodb/4.0.8/bin/mongo 192.168.104.40:2612/admin -u admin -p <password>
```
> db.shutdownServer()
```

### D. sharding

#### connect mongos
$ /apps/mongodb/4.0.8/bin/mongo 192.168.104.40:2612/admin
```
> use admin
```

> no-auth 모드일때는 안해줘도됨
```
> db.auth("admin","passwd");
```

#### add shard
```
> db.runCommand({"addshard" : "rs_esdb01/192.168.104.41:2622,192.168.104.42:2622,192.168.104.43:2622"})
> db.runCommand({"addshard" : "rs_esdb02/192.168.104.42:2623,192.168.104.43:2623,192.168.104.41:2623"})
> db.runCommand({"addshard" : "rs_esdb03/192.168.104.43:2624,192.168.104.41:2624,192.168.104.42:2624"})
> db.runCommand({listShards:1})
> db.runCommand({"enablesharding" : "esdb"})
> db.runCommand({"shardcollection" : "esdb.account", "key" : {"_id" : 1}})
```

> 개발서버 1기에 설치 시
> db.runCommand({"addshard" : "rs_esdb01/172.20.0.104:12622,172.20.0.104:12623,172.20.0.104:12624"})
> db.runCommand({"addshard" : "rs_esdb02/172.20.0.104:22622,172.20.0.104:22623,172.20.0.104:22624"})
> db.runCommand({"addshard" : "rs_esdb03/172.20.0.104:32622,172.20.0.104:32623,172.20.0.104:32624"})
> db.runCommand({listShards:1})
> db.runCommand({"enablesharding" : "esdb"})
> db.runCommand({"shardcollection" : "esdb.account", "key" : {"_id" : 1}})

>#### remove shard
>```
>> db.runCommand({"removeshard" : "rs_esdb03/192.168.104.43:2622,192.168.104.41:2623,192.168.104.42:2624"})
>```

#### add user for database(esdb)
```
> use esdb
> db.createUser({user: "es", pwd: "es", roles: ["readWrite", "dbAdmin"]})
```
> ```
>> db.changeUserPassword("es", "passwd")
> ```

### D. management

$ /apps/mongodb/4.0.8/bin/mongo 192.168.104.40:2612
```
> use config
```

#### config collection
```
> db.shards.find()
> db.databases.find();
> db.chunks.find();
```

#### 요약정보보기 
```
> db.stats()
> db.printShardingStatus();
```

## 3. post-installation setup

### A. create shell script

#### mongodb
> 등록된 인스턴스들을 일괄적으로 start/stop

$ vi /apps/mongodb/mongodb
```
#!/bin/sh
# mongodb    mongodb service shell
# chkconfig: 2345 90 90
# description: mongodb
# processname: mongod
# config: $MONGO_CONF
# pidfile:

MONGO_HOME='/apps/mongodb/4.0.8'
MONGO_BASE="/apps/mongodb/instances"

INSTANCE_NAMES=(${2-"esconfig01" "esdb01" "esdb02" "esdb03"})

serviceCtrl() {
#    find $MONGO_BASE -mindepth 1 -maxdepth 1 -type d | sort -t '\0' -n | while read instance
  for instance in "${INSTANCE_NAMES[@]}"
  do
    instance=$MONGO_BASE/$instance
    case "$1" in
      start)
        echo -en "$instance Instance...\n"
        $MONGO_HOME/bin/mongod --config $instance/config/mongodb.yaml
        ;;
      stop)
        echo -en "$instance Instance...\n"
        $MONGO_HOME/bin/mongod --config $instance/config/mongodb.yaml --shutdown
        ;;
    esac
  done
}

case "$1" in
  start)
    echo -en "Starting mongoDB...\n"
    serviceCtrl start
    echo -e "\n"
    ;;

  stop)
    echo -en  "Shutting down mongoDB...\n"
    serviceCtrl stop
    echo -e "\n"
    ;;

  restart)
    $0 stop
    sleep 5
    $0 start
    ;;

  *)
    echo "Usage: $0 {start|stop|restart}"
    exit 1
esac
exit 0
```

#### mongos
> 등록된 인스턴스들을 일괄적으로 start/stop

$ vi /apps/mongodb/mongos
```
#!/bin/sh
# mongos    mongos service shell
# chkconfig: 2345 90 90
# description: mongos
# processname: mongos
# config: $MONGO_CONF
# pidfile:

MONGO_HOME='/apps/mongodb/4.0.8'
MONGO_BASE="/apps/mongodb/instances"

INSTANCE_NAMES=(${2-"esmongos01"})

serviceCtrl() {
#    find $MONGO_BASE -mindepth 1 -maxdepth 1 -type d | sort -t '\0' -n | while read instance
  for instance in "${INSTANCE_NAMES[@]}"
  do
    instance=$MONGO_BASE/$instance
    case "$1" in
      start)
        echo -en "$instance Instance...\n"
        $MONGO_HOME/bin/mongos --config $instance/config/mongos.yaml &
        ;;
      stop)
        echo -en "$instance Instance...\n"
        $MONGO_HOME/bin/mongos --config $instance/config/mongos.yaml --shutdown
        ;;
    esac
  done
}

case "$1" in
  start)
    echo -en "Starting mongos...\n"
    serviceCtrl start
    echo -e "\n"
    ;;

  stop)
    echo -en  "Shutting down mongos...\n"
    serviceCtrl stop
    echo -e "\n"
    ;;

  restart)
    $0 stop
    sleep 5
    $0 start
    ;;chkconfig mariadb on

  *)
    echo "Usage: $0 {start|stop|restart}"
    exit 1
esac
exit 0
```

### B. add service

$ cp /apps/mongodb/mongodb /etc/rc.d/init.d/mongodb
$ chkconfig --add mongodb

### C. configure SELinux(optional)

$ vi /etc/selinux/config  
```
SELINUX=disabled
```

## 4. execution(starting and stopping daemon services)

### A. start service

$ service mongodb start

### B. restart service

$ service mongodb restart

### C. stop service

$ service mongodb stop

## 9. Appendix

### reference site

#### mongoDB Documentation  
https://docs.mongodb.com/manual/

#### mongoDB Documentation - Configuration File Options  
https://docs.mongodb.com/manual/reference/configuration-options/

#### 서버 2대로 Sharding, Replica-set 구성하기  
http://dreamshutter.tistory.com/17

#### mongodb - /etc/mongod.conf  
http://linuxism.tistory.com/2148

#### 몽고DB 샤드모드로 구동하기  
http://gjchoi.github.io/mongo/mongo-shard-setup/

#### Sample YAML Configuration Files for MongoDB?  
https://dba.stackexchange.com/questions/82591/sample-yaml-configuration-files-for-mongodb

#### MongoDB 구성하기
https://docs.ncloud.com/ko/database/database-10-3.html
