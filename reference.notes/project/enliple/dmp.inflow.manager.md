# data.management.platform

## dmp.inflow.manager

#### data store : clickhouse
`create database`  

_`usage`_
```
CREATE DATABASE db_name [ON CLUSTER cluster_id]  
CREATE DATABASE db_name [ENGINE = engine(…)]  
```
_`example`_  
```
CREATE DATABASE IF NOT EXISTS MOBON_ANALYSIS;
```

_`jdbc connection url`_  
```
jdbc:clickhouse://192.168.2.111:8123/MOBON_ANALYSIS  
```

`create table`  

_`usage`_  
```
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1] [TTL expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2] [TTL expr2],
    ...
    INDEX index_name1 expr1 TYPE type1(...) GRANULARITY value1,
    INDEX index_name2 expr2 TYPE type2(...) GRANULARITY value2
) ENGINE = MergeTree()
[PARTITION BY expr]
[ORDER BY expr]
[PRIMARY KEY expr]
[SAMPLE BY expr]
[TTL expr [DELETE|TO DISK 'xxx'|TO VOLUME 'xxx'], ...]
[SETTINGS name=value, ...]
```

_`example`_  
```sql
CREATE TABLE IF NOT EXISTS MOBON_ANALYSIS.ADVER_DOMAIN_LOG (
	adverId		String,
	domain		String,
	url		String,
	referrer	String,
	solutionType	String,
	auid 		String,
	remoteIp	String,
	device		String,
	userAgent	String,
	charset 	String,
	siteCode	String,
	dsck		String,
	inflowRoute	String,
	createdDate	Datetime default now()
)
ENGINE = MergeTree()
PARTITION BY toYYYYMMDD(createdDate)
ORDER BY (adverId, auid)
PRIMARY KEY (adverId, auid)
SAMPLE BY adverId
TTL createdDate + INTERVAL 8 DAY
SETTINGS index_granularity=8192

```
```sql
CREATE TABLE IF NOT EXISTS MOBON_ANALYSIS.ADVER_CONV_LOG (
	adverId		String,
	domain		String,
	url		String,
	referrer	String,
	auid 		String,
	remoteIp	String,
	device		String,
	userAgent	String,
	charset 	String,
	orderCode	String,
	totalPrice	Int32,
	totalQty	Int16,
	product		String, -- productArr	Array(String),
	paymentType	String,
	siteCode	String,
	dsck		String,
	inflowRoute 	String,
	createdDate	DateTime default now()
)
ENGINE = MergeTree()
PARTITION BY toYYYYMMDD(createdDate)
ORDER BY (adverId, auid)
PRIMARY KEY (adverId, auid)
SAMPLE BY adverId
TTL createdDate + INTERVAL 8 DAY
SETTINGS index_granularity=8192;
```
```sql
CREATE TABLE IF NOT EXISTS MOBON_ANALYSIS.MEDIA_CLICKVIEW_LOG (
	mediaId 	String,
	inventoryId	String,
	frameId		String,

	logType		String,

	adType 		String,
	adProduct	String,
	adCampain	String,

	adverId 	String,
	productCode	String,

	cpoint		Decimal(13, 2),
	mpoint		Decimal(13, 2),

	auid 		String,
	remoteIp	String,

	platform	String,
	device  	String,
	browser 	String,

	createdDate	Datetime default now()
)
ENGINE = MergeTree()
PARTITION BY toYYYYMMDD(createdDate)
ORDER BY (mediaId, inventoryId, adverId)
PRIMARY KEY (mediaId, inventoryId, adverId)
SAMPLE BY mediaId
TTL createdDate + INTERVAL 8 DAY
SETTINGS index_granularity=8192;
```

#### queries

`광고주 시간대별 유입수 조회`  
```
SELECT toYear(createdDate)       AS year
     , toMonth(createdDate)      AS month
     , toDayOfMonth(createdDate) AS day   --, toYYYYMMDD(createdDate) AS date
     , toHour(createdDate)       as hour
     , adverId
     , count(auid)               as hits
     , uniqExact(concat(auid, toString(toYYYYMMDDhhmmss(createdDate)))) AS hits_afs1
     , uniqExact(concat(auid, toString(toYYYYMMDD(createdDate)), toString(toHour(createdDate)), toString(toMinute(createdDate)), toString(toInt8(toSecond(createdDate)/3)))) AS hits_afs3
     , uniqExact(concat(auid, toString(toYYYYMMDD(createdDate)), toString(toHour(createdDate)), toString(toMinute(createdDate)), toString(toInt8(toSecond(createdDate)/5)))) AS hits_afs5
     , uniqExact(auid)           as visitors
FROM MOBON_ANALYSIS.ADVER_DOMAIN_LOG
WHERE createdDate >= parseDateTimeBestEffort('2020-05-11 10:00:00') AND createdDate < parseDateTimeBestEffort('2020-05-11 11:00:00')
GROUP BY year, month, day, hour, adverId
ORDER BY year, month, day, hour, adverId;
```

`매체 시간대별 호출수 및 정산비용 조회`  
```sql
SELECT toYear(createdDate)       AS year
     , toMonth(createdDate)      AS month
     , toDayOfMonth(createdDate) AS day   --, toYYYYMMDD(createdDate) AS date
     , toHour(createdDate)       AS hour
     , mediaId
     , count(*)                  AS HITS
     , uniqExact(concat(mediaId, toString(toYYYYMMDDhhmmss(createdDate)))) AS hits_afs1
     , uniqExact(concat(mediaId, toString(toYYYYMMDD(createdDate)), toString(toHour(createdDate)), toString(toMinute(createdDate)), toString(toInt8(toSecond(createdDate)/3)))) AS hits_afs3
     , uniqExact(concat(mediaId, toString(toYYYYMMDD(createdDate)), toString(toHour(createdDate)), toString(toMinute(createdDate)), toString(toInt8(toSecond(createdDate)/5)))) AS hits_afs5
     , sum(mpoint)               AS MEDIA_SETT_SUM
     , avg(mpoint)               AS MEDIA_SETT_AVG
FROM MOBON_ANALYSIS.MEDIA_CLICKVIEW_LOG
WHERE createdDate >= parseDateTimeBestEffort('2020-05-25 17:00:00') AND createdDate < parseDateTimeBestEffort('2020-05-25 18:00:00')
GROUP BY year, month, day, hour, mediaId
ORDER BY MEDIA_SETT_SUM DESC, year, month, day, hour, mediaId;
```



`create table with kafka engine`  
The delivered messages are tracked automatically, so each message in a group is only counted once. If you want to get the data twice, then create a copy of the table with another group name.

_`usage`_  
```
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
    ...
) ENGINE = Kafka()
SETTINGS
    kafka_broker_list = 'host:port',
    kafka_topic_list = 'topic1,topic2,...',
    kafka_group_name = 'group_name',
    kafka_format = 'data_format'[,]
    [kafka_row_delimiter = 'delimiter_symbol',]
    [kafka_schema = '',]
    [kafka_num_consumers = N,]
    [kafka_max_block_size = 0,]
    [kafka_skip_broken_messages = N,]
    [kafka_commit_every_batch = 0]
```

_`example`_  
```sql
  CREATE TABLE MOBON_ANALYSIS.ADVER_DOMAIN_LOG2 (
	adverId		String,
	domain		String,
	url		String,
	referrer	String
  ) ENGINE = Kafka SETTINGS kafka_broker_list = '10.251.0.75:9092,10.251.0.76:9092,10.251.0.77:9092,10.251.0.78:9092,10.251.0.79:9092',
                            kafka_topic_list = 'advertiser.common',
                            kafka_group_name = 'clickhouse.inflow.table',
                            kafka_format = 'JSONEachRow',
                            kafka_num_consumers = 6,
			    kafka_max_block_size = 1000;
```

>`create materialized view`  
>```
>CREATE MATERIALIZED VIEW IF NOT EXISTS MOBON_ANALYSIS.MV_ADVER_DOMAIN_LOG_HOURLY
>ENGINE = SummingMergeTree()
>PARTITION BY (year, month)
>ORDER BY (year, month, day, hour, adverId)
>AS
>SELECT toYear(createdDate)       AS year
>     , toMonth(createdDate)      AS month
>     , toDayOfMonth(createdDate) AS day   --, toYYYYMMDD(createdDate) AS date
>     , toHour(createdDate)       as hour
>     , adverId
>     , count(auid)               as hits
>     , count(distinct concat(auid, toString(toYYYYMMDDhhmmss(createdDate)))) AS hits_afs1
>     , count(distinct concat(auid, toString(toYYYYMMDD(createdDate)), toString(toHour(createdDate)), toString(toMinute(createdDate)), toString(toInt8(toSecond(createdDate)/3)))) AS hits_afs3
>     , count(distinct concat(auid, toString(toYYYYMMDD(createdDate)), toString(toHour(createdDate)), toString(toMinute(createdDate)), toString(toInt8(toSecond(createdDate)/5)))) AS hits_afs5
>     , count(distinct auid)      as visitors
>FROM MOBON_ANALYSIS.ADVER_DOMAIN_LOG
>GROUP BY year, month, day, hour, adverId
>ORDER BY year, month, day, hour, adverId;
>```

`delete data/rows`  

```
ALTER TABLE MOBON_ANALYSIS_DEV.ADVER_DOMAIN_LOG DELETE WHERE 1=1;
```

#### kafka topic list

- RfData, advertiser.common :  광고주/도메인
- ShopInfoData, advertiser.product : 상품
- ConversionData, advertiser.conversion : 매출/전환
- ClickViewData : 노출/클릭


## 8. trouble-shooting

#### DB::Exception: Memory limit (for query) exceeded: would use 9.31 GiB (attempt to allocate chunk of 4390664 bytes), maximum: 9.31 GiB (version 19.16.12.49)

```
set max_bytes_before_external_group_by=20000000000;  --20 GB for external group by
set max_memory_usage=40000000000; --40GB for memory limit
```

## 9. Appendix

#### reference site
* Clickhouse documents : Overview  
https://clickhouse.tech/docs/en/  

* Engines / Table Engines / Integrations / Kafka  
https://clickhouse.tech/docs/en/engines/table-engines/integrations/kafka/  
