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
CREATE TABLE IF NOT EXISTS MOBON_ANALYSIS.DOMAIN_LOG (
	seqno		UInt64 default (select plus(max(seqno), 1) from MOBON_ANALYSIS.DOMAIN_LOG),
	type 		String,
	id		String,
	url		String,
	referrer	String,
	solution 	String,
	auid 		String,
	remoteip	String,
	device		String,
	charset 	String,
	createdDate	Datetime default now()
)
ENGINE = MergeTree()
PARTITION BY toYYYYMMDD(createdDate)
ORDER BY (type, id, auid)
PRIMARY KEY (type, id, auid)
SAMPLE BY id
TTL createdDate + INTERVAL 8 DAY
SETTINGS index_granularity=8192

```



