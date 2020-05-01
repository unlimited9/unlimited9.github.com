# data.management.platform

## dmp.inflow.manager

#### data store : clickhouse
`create database`  

_>usage_  
```
CREATE DATABASE db_name [ON CLUSTER cluster_id]  
CREATE DATABASE db_name [ENGINE = engine(…)]  
```
_>example_  
```
CREATE DATABASE IF NOT EXISTS MOBON_ANALYSIS;
```

_>jdbc connection url_  
```
jdbc:clickhouse://192.168.2.111:8123/MOBON_ANALYSIS  
```

`create table`  

_>usage_  
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

_>example_  
```
CREATE TABLE IF NOT EXISTS MOBON_ANALYSIS.ADVER_DOMAIN_LOG (
	adverId		String,
	domain		String,
	solutionType	String,
	auid 		String,
	remoteIp	String,
	device		String,
	userAgent	String,
	charset 	String,
	siteCode	String,
	dsck		String,
	refferDomain	String,
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



