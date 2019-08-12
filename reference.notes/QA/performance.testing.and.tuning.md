## System Tuning
>UT : Unit Testing
**ST : System Testing**
SIT : System integration testing
UAT : User Acceptance Testing

### System Testing

#### performance testing
>Created 월요일 04 12월 2017
jmeter : application is open source software, a 100% pure Java application designed to load test functional behavior and measure performance.

서비스 및 시스템의 성능을 확인하기 위해서 실제 사용 환경과 비슷한 환경에서 테스트를 진행하는 것을 말한다.
* 성능(Performance) Test : 성능 측정
응답시간, 처리량, 병목구간, 서버리소스(CPU, Memory, DISK) 측정, 문제점 확인 및 개선
* 부하(Load : ~Volume,\ Endurance~) Test : 임계치 및 튜닝 포인트 확인
부하를 지속적으로 증가시켜서 시스템 상태 및 임계치 확인
* 스트레스(Stress) Test : 장애 조치 및 시스템 성능 한계 확인
과부하 및 비정상적인 상황에 대한 장애 조치와 복구 절차 확인 및 시스템 최고 성능 한계 확인
* 안정성(Stability: ~Soak~) Test : 자원(메모리 등) 누수 확인
지속적인 부하에 대한 시스템 리소스 및 성능 정보 변화 확인
* 스파이크(Spike) Test : 부하 집중 테스트
짧은 시간에 부하 증가에 대한 처리 및 업무 부하(workload) 감소 시 안정화 확인
* 피로(Fatigue) Test : 대역폭 용량을 뛰어넘는 부하에 대한 테스트

### Environment Setup

#### nGrinder
ref. 10.install.n.setup.ngrinder

#### Scouter
ref. 10.install.n.setup.scouter

## 9. Appendix

### reference site

#### 성능 엔지니어링 대한 접근 방법 (Performance tuning)
https://bcho.tistory.com/787

#### 결제 시스템 성능, 부하, 스트레스 테스트
http://woowabros.github.io/experience/2018/05/08/billing-performance_test_experience.html

#### 성능, 부하, 스트레스 테스트에 대하여
https://nesoy.github.io/articles/2018-08/Testing-Performance


#### 성능테스트시 서버 모니터링 방법 정리
https://chigon.tistory.com/entry/%EC%84%B1%EB%8A%A5-%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%8B%9C-%EC%84%9C%EB%B2%84-%EB%AA%A8%EB%8B%88%ED%84%B0%EB%A7%81-%EB%B0%A9%EB%B2%95-%EC%A0%95%EB%A6%AC

#### 리눅스 명령어를 이용한 시스템 모니터링하기
https://www.whatap.io/blog/10/


#### JMeter - KIMSTAR 3.0
http://kimstar.kr/8282/

#### Jmeter를 이용한 성능테스트 정보 공유
https://www.sten.or.kr/club/club_main.php?cb_id=cb_Jmeter

#### jmeter-plugins : Transactions per Second
https://jmeter-plugins.org/wiki/TransactionsPerSecond/


