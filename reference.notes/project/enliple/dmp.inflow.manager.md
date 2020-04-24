# data.management.platform

## dmp.inflow.manager

#### data store : clickhouse
```
-- database 
CREATE DATABASE IF NOT EXISTS MOBON_ANALYSIS;

>jdbc:clickhouse://192.168.2.111:8123/MOBON_ANALYSIS  



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


```
