#### get topic info
/apps/kafka/2.13-2.4.0/bin/kafka-topics.sh --describe --zookeeper mobon-data-zookeeper-client-svc:7214

#### increase topic partition
/apps/kafka/2.13-2.4.0/bin/kafka-topics.sh --alter --topic advertiser.product --partitions 6 --zookeeper mobon-data-zookeeper-client-svc:7214
/apps/kafka/2.13-2.4.0/bin/kafka-topics.sh --alter --topic advertiser.common --partitions 6 --zookeeper mobon-data-zookeeper-client-svc:7214
/apps/kafka/2.13-2.4.0/bin/kafka-topics.sh --alter --topic advertiser.inclinations.product --partitions 6 --zookeeper mobon-data-zookeeper-client-svc:7214

#### delete topic
/apps/kafka/2.13-2.4.0/bin/kafka-topics.sh --delete --topic advertiser.inclinationsproduct --zookeeper mobon-data-zookeeper-client-svc:7214
