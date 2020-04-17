
# spring quartz batch

## @Scheduled 
단순 Scheduled Trigger를 위해서라면 Spring @Scheduled로도 대부분의 기능구현이 가능  
서비스 영속성을 위한 이중화/HA 환경에서는 중복수행 문제 발생
- ShedLock 등을 사용하여 서비스 수행 제어를 하거나 
- quartz clustering을 적용

## Quartz

scheduler.scheduleJob(jobDetail, trigger);

jobDetail에는 Job의 실제 구현 내용과 Job 실행에 필요한 제반 상세 정보가 담겨 있다.
trigger에는 Job을 언제, 어떤 주기로, 언제부터 언제까지 실행할지에 대한 정보가 담겨 있다.
scheduler는 jobDetail과 trigger에 담긴 정보를 이용해서 실제 Job의 실행 스케줄링을 담당한다.

## 9. Appendix

#### reference site

+ [Quartz-1] Quartz Job Scheduler란?  
https://blog.advenoh.pe.kr/spring/Quartz-Job-Scheduler%EB%9E%80/

+ [Quartz-3] Multi WAS 환경을 위한 Cluster 환경의 Quartz Job Scheduler 구현  
https://blog.advenoh.pe.kr/spring/Multi-WAS-%ED%99%98%EA%B2%BD%EC%9D%84-%EC%9C%84%ED%95%9C-Cluster-%ED%99%98%EA%B2%BD%EC%9D%98-Quartz-Job-Scheduler-%EA%B5%AC%ED%98%84/

+ kenshin579 / tutorials-java  
https://github.com/kenshin579/tutorials-java/tree/master/springboot-quartz-cluster


+ [Spring fw] Spring Batch + Quartz (스프링 배치+쿼츠) 정리필요  
https://developyo.tistory.com/205

+ Quartz + Spring Batch 조합하기  
https://blog.kingbbode.com/posts/spring-batch-quartz

+ [Quartz-2] Spring Boot + Quartz을 이용한 Job Scheduler 구현 (In-memory)
https://blog.advenoh.pe.kr/spring/Spring-Boot-Quartz%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%9C-Job-Scheduler-%EA%B5%AC%ED%98%84-In-memory/

+ Java Quartz Scheduler - Job Chaining 구현  
https://homoefficio.github.io/2018/08/12/Java-Quartz-Scheduler-Job-Chaining-%EA%B5%AC%ED%98%84/

+ Quartz 스케줄러 적용 아키텍처 개선 - 1  
https://homoefficio.github.io/2019/09/28/Quartz-%EC%8A%A4%EC%BC%80%EC%A4%84%EB%9F%AC-%EC%A0%81%EC%9A%A9-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98-%EA%B0%9C%EC%84%A0-1/

---
+ Spring Batch / 5. Spring Batch 가이드 - Spring Batch Scope & Job Parameter  
https://jojoldu.tistory.com/330

---
+ Spring Scheduler Lock 설정 in 멀티 클러스터
https://laba.tistory.com/14
