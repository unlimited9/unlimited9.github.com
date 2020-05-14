
-- �뀒�뒪�듃�꽌踰�(106) 移댄봽移� 硫붾땲吏�癒쇳듃
http://172.20.0.106:9000

-- server.properties
# 留덉씠洹몃옒�씠�뀡�슜 �꽕�젙 ( 吏��젙�빐�넃怨� 媛곴컖 釉뚮줈而� �옱�떆�옉�븯硫대맖 )
inter.broker.protocol.version=1.0




-- 171025 kafka �슫�쁺
	bin/kafka-reassign-partitions.sh --zookeeper localhost:2181 --reassignment-json-file increase-replication-factor.json --execute



	cd C:\w\refer\kafka_2.11-1.0.0

-- 二쇳궎�띁�꽌踰�
	bin\windows\zookeeper-server-start.bat config\zookeeper.properties

-- 移댄봽移댁꽌踰�
	bin\windows\kafka-server-start.bat config\server.properties


-- �넗�뵿 由ъ뒪�듃蹂닿린
	bin\windows\kafka-topics.bat --list --zookeeper localhost:2181
	bin\windows\kafka-topics.bat --describe --zookeeper localhost:2181 --topic ClickViewData
	bin/kafka-topics.sh --list --zookeeper localhost:2181
	bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic ClickViewData
	bin/kafka-topics.sh --list --zookeeper localhost:2181 

-- �넗�뵿�쓽 �뙆�떚�뀡�젙蹂�
	bin\windows\kafka-topics.bat --describe --zookeeper localhost:2181 --topic ClickViewData
	bin\windows\kafka-topics.bat --describe --zookeeper localhost:2181
	bin/kafka-topics.sh --describe --zookeeper 61.110.214.248:2181,61.110.214.249:2181,61.110.214.250:2181 --topic test
	bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test
	bin/kafka-preferred-replica-election.sh --zookeeper 61.110.214.248:2181,61.110.214.249:2181,61.110.214.250:2181/kfakf
	bin/kafka-topics.sh --describe --zookeeper localhost:2181

	
-- �뙆�떚�뀡蹂� offset �냼吏� 吏꾨룄瑜� �솗�씤�븳�떎.
	bin\windows\kafka-consumer-groups.bat --new-consumer --describe --group group2 --bootstrap-server localhost:9092
	bin\windows\kafka-consumer-groups.bat --new-consumer --describe --group test --bootstrap-server 192.168.72.131:9092
	bin/kafka-consumer-groups.sh --new-consumer --describe --group group1 --bootstrap-server 192.168.72.134:9092,192.168.72.135:9092,192.168.72.136:9092
	bin/kafka-consumer-groups.sh --describe --group group1 --bootstrap-server localhost:9092


-- �넗�뵿 �깮�꽦�븯湲� �뙆�떚�뀡, 由ы뵆由ъ��씠�뀡 	ClickViewData,ConversionData,ShopStatsInfoData,ShopInfoData
	bin\windows\kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 21 --topic ClickViewData
	bin\windows\kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 21 --topic ExternalData
	bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 5 --topic ClickViewData
	bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 5 --topic ClickViewPointData
	bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 5 --topic SDKPackageData
	bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 5 --topic ConversionData
	bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 5 --topic ShopStatsInfoData
	bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 5 --topic ShopInfoData
	bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 5 --topic ExternalData
	bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 5 --topic test


-- �넗�뵿 �뙆�떚�뀡 媛��닔 �닔�젙 ( �뒛由ш린留� )
	bin\windows\kafka-topics.bat --zookeeper localhost:2181 --alter --partitions 21 --topic ClickViewData
	bin/kafka-reassign-partitions.sh --zookeeper localhost:2181 --topics-to-move-json-file topics-to-move.json --broker-list "1,2,3" --generate
	bin/kafka-topics.sh --alter --zookeeper localhost:2181 --partitions 6 --topic ClickViewData
	bin/kafka-topics.sh --alter --zookeeper localhost:2181 --partitions 42 --topic SDKPackageData
	bin/kafka-topics.sh --alter --zookeeper localhost:2181 --partitions 42 --topic ConversionData
	bin/kafka-topics.sh --alter --zookeeper localhost:2181 --partitions 42 --topic ShopStatsInfoData
	bin/kafka-topics.sh --alter --zookeeper localhost:2181 --partitions 42 --topic ShopInfoData
	bin/kafka-topics.sh --alter --zookeeper localhost:2181 --partitions 42 --topic ExternalData
	bin/kafka-topics.sh --alter --zookeeper localhost:2181 --partitions 42 --topic test
	
	bin/kafka-topics.sh --alter --zookeeper localhost:2181 --partitions 63 --topic ClickViewData


-- �넗�뵿 吏��슦湲�
	bin/kafka-topics.sh --delete --zookeeper localhost:2181 --topic ClickViewData
	bin/kafka-topics.sh --delete --zookeeper localhost:2181 --topic ClickViewPointData
	bin/kafka-topics.sh --delete --zookeeper localhost:2181 --topic ExternalData

-- 洹몃９ 吏��슦湲�
	bin/kafka-consumer-groups.sh --zookeeper localhost:2181 --delete --group group-dev2

-- offset reset
	bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group group-dev2 --reset-offsets --to-latest --all-topics --execute


-- �뀍�뵆 �봽濡쒕��꽌
	bin/kafka-console-producer.sh --broker-list 10.1.6.2:9092,10.1.6.3:9092,10.1.6.4:9092 --topic test
	bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
	bin/kafka-console-producer.sh --broker-list 172.20.0.106:9092 --topic test
	bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test


-- �뀍�뵆 而⑥뒋癒� 援ы쁽
	bin/kafka-console-consumer.sh --zookeeper 61.110.214.248:2181,61.110.214.249:2181,61.110.214.250:2181 --topic test --from-beginning
	bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning



-- topic�쑝濡� 硫붿떆吏�瑜� 200000 媛� 蹂대궦�떎.
	bin\windows\kafka-verifiable-producer.bat --topic test --max-messages 100 --broker-list localhost:9092 batch.size=1024
	bin/kafka-verifiable-producer.sh --topic test --max-messages 100 --broker-list 61.110.214.248:9092,61.110.214.249:9092,61.110.214.250:9092 batch.size=1024



-- �봽濡쒕��꽌 而⑥뒋癒� 踰ㅼ튂留덊겕
	bin\windows\kafka-producer-perf-test --topic test --num-records 1000000 --record-size 1000 --throughput -1 --producer-props acks=1 bootstrap.servers=localhost:9092 buffer.memory=33554432 linger.ms=500 batch.size=2048
	bin\windows\kafka-producer-perf-test --topic test --num-records 1000000 --record-size 1000 --throughput -1 --producer-props acks=1 bootstrap.servers=localhost:9092 buffer.memory=5000000 linger.ms=500 batch.size=3072
	bin\windows\kafka-producer-perf-test --topic test --num-records 1000000 --record-size 1000 --throughput -1 --producer-props acks=1 bootstrap.servers=localhost:9092 buffer.memory=10000000 linger.ms=10000 batch.size=16384
	bin\windows\kafka-producer-perf-test --topic test --num-records 10000000 --record-size 1000 --throughput -1 --producer-props acks=1 bootstrap.servers=localhost:9092 buffer.memory=1000000 linger.ms=100000 batch.size=65536
	bin/kafka-producer-perf-test.sh --topic test --num-records 100 --record-size 1000 --throughput -1 --producer-props acks=1 bootstrap.servers=localhost:9092 buffer.memory=1000000 linger.ms=100000 batch.size=65536


	bin/kafka-consumer-perf-test.sh --group group1 --topic test --zookeeper localhost:2181 --threads 3 --messages 1000000
	bin/kafka-consumer-perf-test.sh --group group1 --topic test --zookeeper localhost:2181 --threads 3 --messages 1000000 --show-detailed-stats
	bin/kafka-consumer-perf-test.sh --zookeeper localhost:2181 --threads 4 --topic test --messages 1000000



http://192.168.2.78:9000/api/status/kafka/brokers
http://192.168.2.78:9000/api/status/kafka/brokers/extended
http://192.168.2.78:9000/api/status/kafka/topics
http://192.168.2.78:9000/api/status/kafka/topicIdentities
http://192.168.2.78:9000/api/status/clusters
http://192.168.2.78:9000/api/status/kafka/ClickViewData/underReplicatedPartitions
http://192.168.2.78:9000/api/status/kafka/ClickViewData/unavailablePartitions
http://192.168.2.78:9000/api/status/kafka/group1/ClickViewData/KF/topicSummary
http://192.168.2.78:9000/api/status/kafka/group1/KF/groupSummary
http://192.168.2.78:9000/api/status/kafka/consumersSummary




-- �넗�뵿 �슫�쁺
	http://mrsence.tistory.com/75
	
	
-- 紐쎄퀬 紐⑤땲�꽣留�
mongotop --port 20001
mongostat --port 10001 --discover -i


-- 紐쎄퀬 �깶�뵫荑쇰━
db.consumer_thread.drop();
db.seq_date.drop();

db.createCollection('consumer_thread');
sh.shardCollection ("billing.consumer_thread", {_id:"hashed"});

db.createCollection('seq_date');
sh.shardCollection ("billing.seq_date", {_id:"hashed"});


db.createCollection('insert_maria_error');
sh.shardCollection ("billing.insert_maria_error", {_id:"hashed"});
db.insert_maria_error.createIndex( {"insertDate":-1, "collectionName":1} );



SELECT * FROM MOB_ADVER_STATS WHERE ADVRTS_AMT>0;
SELECT * FROM MOB_MEDIA_SCRIPT_STATS WHERE ADVRTS_AMT>0;
SELECT * FROM MOB_MEDIA_STATS WHERE ADVRTS_AMT>0;

ALTER TABLE CONVERSION_LOG ADD PARTITION (PARTITION CONVERSION_LOG201709 VALUES IN(20171001,20171002,20171003,20171004,20171005,20171006,20171007,20171008,20171009,20171010,20171011,20171012,20171013,20171014,20171015,20171016,20171017,20171018,20171019,20171020,20171021,20171022,20171023,20171024,20171025,20171026,20171027,20171028,20171029,20171030))




