
#### 광고주 운영서버 monogodb/mongos
```
$ mongos --port 10001 --fork --quiet --maxConns 500 --logpath /home/dreamsearch/logs/mongodb/mongo_mongos.log			--configdb config-repl/192.168.2.120:30001,192.168.2.122:30001,192.168.2.124:30001,192.168.2.130:30001,192.168.2.132:30001
$ mongos --port 11001 --fork --quiet --maxConns 500 --logpath /home/dreamsearch/logs/adid_mongodb/mongo_mongos.log 	--configdb config-repl/192.168.2.160:30001,192.168.2.161:30001,192.168.2.162:30001
$ mongos --port 12001 --fork --quiet --maxConns 500 --logpath /home/dreamsearch/logs/nlp_mongodb/mongo_mongos.log 		--configdb config-repl/192.168.2.182:30001,192.168.2.183:30001,192.168.2.184:30001
$ mongos --port 13001 --fork --quiet --maxConns 500 --logpath /home/dreamsearch/logs/keyword_mongodb/mongo_mongos.log --configdb config-repl/192.168.2.113:30001,192.168.2.113:30002,192.168.2.114:30003
```

#### 광고주 2.0 10.251.0.194 monogodb/mongos
```
$ /apps/mongodb/3.4.22/bin/mongos --port 10001 --fork --quiet --maxConns 500 --logpath /logs/mongodb/mongos.log --configdb config-repl/192.168.2.120:30001,192.168.2.122:30001,192.168.2.124:30001,192.168.2.130:30001,192.168.2.132:30001
$ /apps/mongodb/3.4.22/bin/mongos --port 13001 --fork --quiet --maxConns 500 --logpath /logs/mongodb/keyword_mongos.log --configdb config-repl/192.168.2.113:30001,192.168.2.113:30002,192.168.2.114:30003
```
