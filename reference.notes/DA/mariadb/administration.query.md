# useful mariadb queries

## monitoring

#### process
```
SHOW PROCESSLIST;
SHOW FULL PROCESSLIST;

kill 프로세스ID;

```

#### server
```
SHOW STATUS;
```

#### variables
show variables;
SHOW SESSION VARIABLES;
SHOW GLOBAL VARIABLES 

#### table
SHOW OPEN TABLES FROM dbname

## lock
READ Lock : 락을 명시적으로 사용한 세션과 모든 세션에서 insert, update, delete가 불가능하고 select만 가능  
WRITE Lock : 락을 명시적으로 사용한 세션에서의 쓰레드만 read, wrtite가 가능.  
````
/* Read Lock */ lock tables table_name READ;
/* Write Lock */ lock tables table_name WRITE;
/* 여러개도 가능함. */ lock tables table_name WRITE, table_name2 READ;
/* 락 걸려있을 때 해제 */ UNLOCK TABLES;
/* Global Lock */ FLUSH TABLES WITH READ LOCK;
/* Innodb Dead Lock 확인 */ SHOW innodb status show engine innodb status
````
 
 
## partition
```
/* 이미 존재하는 테이블에 LIST 형태로 파티션 등록 */
ALTER table `tablename` PARTITION BY LIST (dt)
(
    PARTITION p20140409 VALUES IN (20140409),
    PARTITION p20140410 VALUES IN (20140410)
)

/* 이미 존재하는 테이블에 RANGE 형태로 파티션 등록(datetime 필드를 사용하는 경우) */
ALTER TABLE `tablename` PARTITION BY RANGE (UNIX_TIMESTAMP(stamp_inserted)) 
(
    PARTITION p2014082620 VALUES LESS THAN (1365990900),
    PARTITION p2014082621 VALUES LESS THAN (1365991801)
);  

/* 이미 파티션이 정의된 테이블에 파티션 추가 */
ALTER TABLE `tablename` ADD PARTITION (
    PARTITION p20140423 VALUES IN (20140423)
);

/* 파티션 존재 여부 확인 */
SELECT * FROM information_schema.partitions WHERE table_name='tablename'

/* 파티션 삭제 */
ALTER TABLE tablename DROP PARTITION partition_name;

/* 파티션 재배치. p201410 파티션이 있는 상태에서, 분리한다. */
ALTER TABLE `tablename` REORGANIZE PARTITION p201410 INTO (
    PARTITION p201409 VALUES LESS THAN (5415777),
    PARTITION p201410 VALUES LESS THAN MAXVALUE
);
```

## Appendix

#### reference site

* MySQL 자주 사용하는 쿼리 모음과 관리 팁  
http://blog.kichul.co.kr/2017/03/13/2017-03-13-mysql-note/
