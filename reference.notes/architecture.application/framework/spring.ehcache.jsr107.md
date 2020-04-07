# spring cache

Cache는 조회 조건에 따른 결과가 항상 같은 혹은 잘 변하지 않는 조회 비용이 높은 객체 및 데이터를 대상으로 낮은 비용으로 빠르게 접근할 수 있는 메모리 등 저장소를 임시로 활용하는 것이다.
- 한번 읽은(처리한) 데이타를 임시로 저장하고 필요에 따라 전송, 갱신, 삭제하는 기술로 보통은 데이타의 보관장소로 서버의 메모리를 사용하는 경우가 많다
- 조회 조건에 따른 결과가 항상 같은 혹은 잘 변하지 않는 데이터
- 조회 조건에 따른 결과가 항상 같은 혹은 잘 변하지 않는 데이터

- 단순한, 또는 단순한 구조의 정보
- 반복적으로 동일하게 제공해야 하거나 -> 빈번한 동일요청의  반복
- 정보의 변경주기가 빈번하지 않고, 단위처리 시간이 오래걸리는 (단위처리비용이 높은) 정보
- 정보의 최신화가 반드시 실시간으로 이뤄지지 않아도 서비스 품질에 영향을 거의 주지 않는 정보 

- 반복적으로 조회되는 데이터에 캐시를 적용하여 응답시간을 줄이고 DBMS에 부하를 낮출 수 있다.  
  (많은 양의 데이터나 오랜 기간 유지하기 위한 용도로 사용할 수는 있으나 이 경우 캐시를 써야하는 타당성을 다시 생각해 볼 필요가 있다.)

--
Cache는 조회 조건에 따른 결과가 항상 같은 혹은 잘 변하지 않는 조회 비용이 높은 객체 및 데이터를 대상으로 낮은 비용으로 빠르게 접근할 수 있는 메모리 등 저장소를 임시로 활용하는 것이다.
많은 양의 데이터나 오랜 기간 유지하기 위한 용도로 사용할 수는 있으나 이 경우 캐시를 써야하는 타당성을 다시 생각해 볼 필요가 있다.


## 9. Appendix

#### reference site

+ [DB/Redis]Redis에 대해서 공부하기, Redis vs Ehcache vs Memcached 비교하며 파악하기  
https://postitforhooney.tistory.com/entry/DBRedisRedis에-대해서-공부하기-Redis-vs-Ehcache-vs-Memcached-비교하며-파악하기

* Ehcache Documentation  
https://www.ehcache.org/documentation/

* Ehcache 3 Samples  
https://github.com/ehcache/ehcache3-samples

* Terracotta  
http://www.terracotta.org/

+ Spring @Cacheable Cache 처리  
http://dveamer.github.io/backend/SpringCacheable.html

+ 스프링 @Cacheable, 메서드단 캐싱  
https://thereclub.tistory.com/10

+ SpringBoot기반 Redis Cache 활용법  
https://yonguri.tistory.com/82

+ Ehcache 옵션 정리  
https://kimyhcj.tistory.com/253

