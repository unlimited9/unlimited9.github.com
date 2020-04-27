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
CREATE TABLE IF NOT EXISTS MOBON_ANALYSIS.DOMAIN --ON CLUSTER cluster
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
CREATE TABLE DOMAIN (
	adver_id		VARCHAR(32) NOT NULL
	adver_id		VARCHAR(32) NOT NULL
	device_cd		CHAR(1),
	domain 			String,
	referrer_url	String,
	solution 		String,
	remote_ip		String,
	auid 			String,
	sitecode 		String
	charset 		String,
	create 			String
)
ENGINE = MergeTree()
PRIMARY KEY ClientID
ORDER BY ClientID
```



