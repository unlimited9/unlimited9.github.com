# dmp.inflow.manager

```
SELECT toYear(createdDate)       AS year
     , toMonth(createdDate)      AS month
     , toDayOfMonth(createdDate) AS day   --, toYYYYMMDD(createdDate) AS date
     , toHour(createdDate)       as hour
     , adverId
     , count(auid)               as hits
     , count(distinct concat(auid, toString(toYYYYMMDD(createdDate)), toString(toHour(createdDate)), toString(toMinute(createdDate)), toString(toInt8(toSecond(createdDate)/3)))) AS hits_afs3
     , count(distinct auid)      as visitors
FROM MOBON_ANALYSIS.ADVER_DOMAIN_LOG
GROUP BY year, month, day, hour, adverId
ORDER BY year, month, day, hour, adverId;
```





## 9. Appendix

#### reference site

* ClickHouse  
https://clickhouse.tech/

+ How To Install And Get Started With ClickHouse On CentOS 7  
https://phoenixnap.com/kb/how-to-install-clickhouse-centos  
