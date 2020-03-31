# administration script : kafka

#### get topic info
/apps/kafka/2.13-2.4.0/bin/kafka-topics.sh --describe --zookeeper mobon-data-zookeeper-client-svc:7214

#### increase topic partition
/apps/kafka/2.13-2.4.0/bin/kafka-topics.sh --alter --topic advertiser.product --partitions 6 --zookeeper mobon-data-zookeeper-client-svc:7214
/apps/kafka/2.13-2.4.0/bin/kafka-topics.sh --alter --topic advertiser.common --partitions 6 --zookeeper mobon-data-zookeeper-client-svc:7214
/apps/kafka/2.13-2.4.0/bin/kafka-topics.sh --alter --topic advertiser.inclinations.product --partitions 6 --zookeeper mobon-data-zookeeper-client-svc:7214

#### delete topic
/apps/kafka/2.13-2.4.0/bin/kafka-topics.sh --delete --topic advertiser.inclinationsproduct --zookeeper mobon-data-zookeeper-client-svc:7214

## 9. Appendix

#### reference site

+ [kafka] kafka broker topic 정보 변경 - alter kafka topic  
https://m.blog.naver.com/PostView.nhn?blogId=juner84&logNo=220339785479&proxyReferer=https%3A%2F%2Fwww.google.com%2F

* Kafka & Elasticsearch - 토픽 삭제 (Topic delete)  
https://jusunglee.tistory.com/entry/%ED%86%A0%ED%94%BD-%EC%82%AD%EC%A0%9C-Topic-delete
