
#### mongos
$ /apps/mongodb/3.4.22/bin/mongos --port 10001 --fork --quiet --maxConns 500 --logpath /logs/mongodb/mongos.log --configdb config-repl/192.168.2.120:30001,192.168.2.122:30001,192.168.2.124:30001,192.168.2.130:30001,192.168.2.132:30001  

#### keyword_mongos
$ /apps/mongodb/3.4.22/bin/mongos --port 13001 --fork --quiet --maxConns 500 --logpath /logs/mongodb/keyword_mongos.log --configdb config-repl/192.168.2.113:30001,192.168.2.113:30002,192.168.2.114:30003  

#### adid_mongos
$ /apps/mongodb/3.4.22/bin/mongos --port 11001 --fork --quiet --maxConns 500 --logpath /logs/mongodb/adid_mongos.log --configdb config-repl/192.168.2.160:20001,192.168.2.161:20001,192.168.2.162:20001,192.168.2.163:20001,192.168.2.164:20001,192.168.2.165:20001,192.168.2.166:20001,192.168.2.167:20001,192.168.2.168:20001,192.168.2.169:20001,192.168.2.170:20001,192.168.2.171:20001  
