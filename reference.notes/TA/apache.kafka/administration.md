# 20.administration : kafka

>Created 월요일 20 11월 2017
operation and maintenance

## Getting Started

### Concept

- publish-subscribe model
publisher는 topic에 대한 정보만 알고 있고, 마찬가지로 subscriber도 topic만 바라본다.
publisher 와 subscriber는 서로 모르는 상태다.
- topic, partiton
topic을 여러개 partition으로 구성하여 분산/병렬 처리할 수 있다.
- producer, consumer
- broker/cluster, zookeepr
- consumer group
- replication

### A. topic management command

#### Topic 생성
$ /apps/kafka/2.12-2.2.0/bin/kafka-topics.sh --zookeeper 10.10.10.25:7214 --create --replication-factor 3 --partitions 18 --topic estopic

>하나의 node에 여러 인스턴스를 띄워야 할 경우 아래와 같이 설정파일을 분리해서 기동한다.
>/apps/kafka/2.12-2.2.0/bin/kafka-topics.sh --zookeeper 127.0.0.1:17214 --create --replication-factor 3 --partitions 18 --topic advertiser.conv

#### Topic 목록
$ /apps/kafka/2.12-2.2.0/bin/kafka-topics.sh --zookeeper 10.10.10.25:7214 --list

>/apps/kafka/2.12-2.2.0/bin/kafka-topics.sh --zookeeper 127.0.0.1:17214 --list

#### Topic 상세보기
$ /apps/kafka/2.12-2.2.0/bin/kafka-topics.sh --zookeeper 10.10.10.25:7214 --describe --topic estopic

>/apps/kafka/2.12-2.2.0/bin/kafka-topics.sh --zookeeper 127.0.0.1:17214 --describe --topic advertiser.conv

#### Topic 수정
$ /apps/kafka/2.12-2.2.0/bin/kafka-topics.sh --zookeeper 10.10.10.25:7214 --alter --topic estopic --partitions 24

>/apps/kafka/2.12-2.2.0/bin/kafka-topics.sh --zookeeper 127.0.0.1:17214 --alter --topic advertiser.conv --partitions 24

#### Topic 삭제
$ /apps/kafka/2.12-2.2.0/bin/kafka-topics.sh --zookeeper 10.10.10.25:7214 --delete --topic estopic
$ /apps/kafka/2.12-2.2.0/bin/kafka-topics.sh --zookeeper 10.10.10.25:7214 --list
>estopic - marked for deletion

$ /apps/kafka/2.12-2.2.0/bin/zookeeper-shell.sh 10.10.10.25:7214
>ls /brokers/topics
[estopic]
rmr /brokers/topics/estopic

### B. topic message control command : producer/consumer

#### Message 생성 : producer

$ /apps/kafka/2.12-2.2.0/bin/kafka-console-producer.sh --broker-list 10.10.10.25:7642 --topic estopic

>/apps/kafka/2.12-2.2.0/bin/kafka-console-producer.sh --broker-list 127.0.0.1:17642 --topic advertiser.conv

#### Message 소비 : consumer

$ /apps/kafka/2.12-2.2.0/bin/kafka-console-consumer.sh --bootstrap-server 10.10.10.25:7642 --topic estopic --from-beginning

>/apps/kafka/2.12-2.2.0/bin/kafka-console-consumer.sh --bootstrap-server 127.0.0.1:17642 --topic advertiser.conv --from-beginning

>kafka-console-consumer.sh는 실행된 시점 이후에 생산되는 메시지만 소비
>from-beginning 옵션을 주면 해당 topic의 맨 처음 메시지부터 소비

  
