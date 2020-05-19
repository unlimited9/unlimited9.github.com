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
	totalPrice	Int16,
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
     , uniqExact(auid)           as visitors
FROM MOBON_ANALYSIS.ADVER_DOMAIN_LOG
WHERE createdDate >= parseDateTimeBestEffort('2020-05-11 10:00:00') AND createdDate < parseDateTimeBestEffort('2020-05-11 11:00:00')
GROUP BY year, month, day, hour, adverId
ORDER BY year, month, day, hour, adverId;
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


