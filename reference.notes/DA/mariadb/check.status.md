
#### 프로세스 개수  
mysql> show processlist;  

#### 쓰레드 개수  
mysql> show status like 'Threads_connected';

#### 상태 조회
$ /usr/local/mysql/bin/mysqladmin status -p
>> mysql> show status;

- Aborted_clients : 클라이언트가 연결을 적절히 닫지않아서 죽었기때문에 끊어진 연결수.  
- Aborted_connects : 연결실패된 mysql서버에 연결시도 수.  
- Bytes_received : 모든 클라이언트로 부터 받은 바이트 수  
- Bytes_sent : 모든 클라이언트에게 보낸 바이트수  
- Connections : mysql서버에 연결시도한 수  
- Created_tmp_disk_tables : sql문을 실행하는 동안 생성된 디스크에 존재하는 임시테이블 수  
- Created_tmp_tables : sql문을 실행하는 동안 생성된 메모리에 존재하는 임시테이블 수  
- Created_tmp_files : 얼마나 많은 임시파일을 mysqld가 생성했는가  
- Delayed_insert_threads : 사용중인 insert handler threads가 지연되고 있는 수  
- Delayed_writes : INSERT DELAYED로 쓰여진 rows수  
- Delayed_errors : 어떤 에러(duplicate key로인한 때문에 INSERT DELAYED로 쓰여진 rows수  
- Flush_commands : 초과 flush명령수  
- Handler_delete : 테이블로 부터 지워진 rows수  
- Handler_read_first : 인덱스로 부터 읽혀진 처음 entry수. 이것이 높으면 서버는 많은 full index scans를 하고 있다는 것을 의미. 예를 들어 SELECT col1 FROM foo는 col1은 인덱스되었다는 것을 추정.  
- Handler_read_key : 키가 존재하는 row를 읽는 요청수. 이것이 높으면 당신의 쿼리와 테이블 이 적절히 인덱스화되었다는 좋은 지적이 된다.  
- Handler_read_next : 키순서대로 다음 row를 읽는 요청수. 이것은 만약 range constraint와 함께 인덱스컬럼을 쿼리할 경우 높아질 것이다. 이것은 또한 인덱스 스캔 하면 높아질 것이다.  
- Handler_read_rnd : 고정된 위치에 존재하는 row를 읽는 요청수. 이것은 결과를 정렬하기를 요하는 많은 쿼리를 한다면 높아질 것이다.  
- Handler_read_rnd_next : 데이타파일에서 다음 row를 읽기를 요청수. 이것은 많은 테이블 스캔을 하면 높아질 것이다.  
- Handler_update : Number of requests to update a row in a table. 한테이블에 한 row를 업데이트를 요청하는 수  
- Handler_write :  한테이블에 한 row를 insert요청하는 수  
- Key_blocks_used :  key 캐시에서 블럭을 사용하는 수  
- Key_read_requests : 캐시에서 키블럭을 읽기를 요청하는 수  
- Key_reads : 디스크로부터 키블럭을 물리적으로 읽는 수  
- Key_write_requests : 캐시에서 키블럭을 쓰기위해 요청하는 수  
- Key_writes :  디스크에 키블럭을 물리적으로 쓰는 수  
- Max_used_connections : 동시사용 연결 최대수  
- Not_flushed_key_blocks : 키캐시에서 키블럭이 바뀌지만 디스크에는 아직 flush되지 않는다.  
- Not_flushed_delayed_rows :  INSERT DELAY queue에서 쓰여지기를 기다리는 row수  
- Open_tables : 현재 오픈된 테이블수  
- Open_files : 현재 오픈된 파일수  
- Open_streams : 주로 logging에 사용되는 현재 오픈된 stream수  
- Opened_tables : 지금까지 오픈된 테이블 수  
- Select_full_join : 키없이 조인된 수(0이 되어야만 한다)  
- Select_full_range_join : reference table에서 range search를 사용한 조인수  
- Select_range : 첫번째 테이블에 range를 사용했던 조인수. 보통 이것이 크더라도 위험하진 않다.  
- Select_scan : 첫번째 테이블을 스캔했던 조인수  
- Select_range_check : 각 row후에 key usage를 체크한 키없이 조인한 수(0이어야만 한다)  
- Questions : 서버에서 보낸 쿼리수  
- Slave_open_temp_tables : 현재 slave thread에 의해 오픈된 임시 테이블 수  
- Slow_launch_threads : 연결된 slow_launch_time보다 더 많은 수를 갖는 쓰레드수  
- Slow_queries : long_query_time보다 더 많은 시간이 걸리는 쿼리 수. Slow Query Log참고  
- Sort_merge_passes : 정렬해야만 하는 merge수. 이 값이 크면 sort_buffer를 증가하는것에 대해 고려해야 한다.  
- Sort_range : Number of sorts that where done with ranges.  
- Sort_rows : 정렬된 row수  
- Sort_scan : 테이블 스캔에 의해 행해진 정렬수  
- Table_locks_immediate : 즉시 획득된 테이블 lock 시간 (3.23.33부터 추가된 항목)  
- Table_locks_waited : 즉시 획득되지 않고 기다림이 필요한 테이블 lock 시간. 이것이 높아지면 성능에 문제가 있으므로, 먼저 쿼리를 최적화 시키고, 테이블을 분산시키거나 복제를 사용해야 한다. (3.23.33부터 추가된 항목)  
- Threads_cached : 스레드 캐시에서 쓰레드 수  
- Threads_connected : 현재 오픈된 연결수  
- Threads_created : 연결을 다루기위해 생성된 쓰레드 수  
- Threads_running : sleeping하지 않는 쓰레드 수  
- Uptime : 서버가 스타트된 후로 지금까지의 시간  

## 9. Appendix

#### reference site

+ (MySQL) 현재 상태 확인하는 명령어 (현재 connection개수 확인 등)  
https://it77.tistory.com/366
