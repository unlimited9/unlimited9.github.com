# 04.implementation : elasticsearch

>Created 목요일 30 11월 2017
concept theorem

## 1. JAVA

## connection

### Rest Client

#### I/O reactor status appears when using elasticsearch: STOPPED
When using the elasticsearch client, this exception occurs due to high concurrency due to the client being closed after the code is executed.
```
java.lang.IllegalStateException: 
Request cannot be executed; I/O reactor status: STOPPED
```
#### Solve
```java
try (RestClient restClient = initRestClient()) {
	...
} catch {
	...
}
```
`try-with-resource`Will automatically close after the logic is executed`resource`, will appear in high concurrency scenarios when the client has been closed


### Transport Client


## 9. Appendix

### reference site

#### Java REST Client [master] » Java High Level REST Client
https://www.elastic.co/guide/en/elasticsearch/client/java-rest/master/java-rest-high.html

#### Java REST Client [master] » Java Low Level REST Client » Getting started » Initialization
https://www.elastic.co/guide/en/elasticsearch/client/java-rest/master/java-rest-low-usage-initialization.html

#### Java REST Client [7.1] » Java High Level REST Client » Using Java Builders » Building Queries
https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-query-builders.html

#### Java REST Client [master] » Java High Level REST Client » Snapshot APIs » Create Snapshot API
https://www.elastic.co/guide/en/elasticsearch/client/java-rest/master/java-rest-high-snapshot-create-snapshot.html

#### Elasticsearch - 6. Elasticsearch Java Client !(엘라스틱서치 자바 클라이언트,High-Level Rest Client)
https://coding-start.tistory.com/172

#### [elasticsearch] mapping 분석 및 이해.  
https://jjeong.tistory.com/854






#### 라라벨 프레임워크 - 엘라스틱서치 사용 경험기 : 초기 작업 수행
https://www.popit.kr/%EB%9D%BC%EB%9D%BC%EB%B2%A8-%EC%97%98%EB%9D%BC%EC%8A%A4%ED%8B%B1%EC%84%9C%EC%B9%98-%EC%82%AC%EC%9A%A9-%EA%B2%BD%ED%97%98%EA%B8%B0-%EC%B4%88%EA%B8%B0%EC%9E%91%EC%97%85/

#### [ELK] 엘라스틱서치 배우기 - 검색API
https://12bme.tistory.com/400

#### [Elasticsearch] 입문하기(2) - 기본 API( index, document CRUD )
https://victorydntmd.tistory.com/312

#### [Elasticsearch] Snapshot and Restore 알아보기  
https://jjeong.tistory.com/1276

#### logstash apache COMMONAPACHELOG 
https://ddakker.tistory.com/category/Tools






#### I/O reactor status appears when using elasticsearch: STOPPED
http://www.programmersought.com/article/1013933196/
