# spring cache

Cache는 조회 조건에 따른 결과가 항상 같은 혹은 잘 변하지 않는 조회 비용이 높은 객체 및 데이터를 대상으로 낮은 비용으로 빠르게 접근할 수 있는 메모리 등 저장소를 임시로 활용하는 것이다.
- 반복적으로 조회되는 데이터에 캐시를 적용하여 응답시간을 줄이고 DBMS에 부하를 낮출 수 있다.
- 반복적으로 조회되는 데이터에 캐시를 적용하여 응답시간을 줄이고 DBMS에 부하를 낮출 수 있다.

(많은 양의 데이터나 오랜 기간 유지하기 위한 용도로 사용할 수는 있으나 이 경우 캐시를 써야하는 타당성을 다시 생각해 볼 필요가 있다.)


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

