## MySql DB/테이블 사이즈 확인을 위한 쿼리
> 쿼리 시점에 정확한 사이즈는 아니지만, 참고할 만한 데이터

#### 테이블별 사이즈 확인
```
SELECT
    table_name,
    table_rows,
    round(data_length/(1024*1024),2) as 'DATA_SIZE(MB)',
    round(index_length/(1024*1024),2) as 'INDEX_SIZE(MB)'
FROM information_schema.TABLES
where table_schema = 'dreamsearch'
GROUP BY table_name
ORDER BY data_length DESC
LIMIT 10;
```

#### Database 별 사이즈 확인
```
SELECT
	count(*) NUM_OF_TABLE,
	table_schema, concat(round(sum(table_rows)/1000000,2),'M') rowss,
	concat(round(sum(data_length)/(1024*1024*1024),2),'G') DATA,
	concat(round(sum(index_length)/(1024*1024*1024),2),'G') idx,
	concat(round(sum(data_length+index_length)/(1024*1024*1024),2),'G') total_size,
	round(sum(index_length)/sum(data_length),2) idxfrac
FROM information_schema.TABLES
GROUP BY table_schema
ORDER BY sum(data_length+index_length) DESC LIMIT 10;
```

## 9. Appendix

#### reference site

+ MySql DB/테이블 사이즈 확인을 위한 쿼리  
https://dimdim.tistory.com/entry/MySql-DB%ED%85%8C%EC%9D%B4%EB%B8%94-%EC%82%AC%EC%9D%B4%EC%A6%88-%ED%99%95%EC%9D%B8%EC%9D%84-%EC%9C%84%ED%95%9C-%EC%BF%BC%EB%A6%AC


