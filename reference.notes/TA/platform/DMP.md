# a.DMP(Data Management Platform)

>Created 목요일 30 11월 2017

ETT : 추출(Extraction), 변형(Transformation), 이동(Transportation)
ETL : 추출(Extraction), 변환(Transformation), 적재(Loading)

## Conceptual model

*  Ad-tech(ad-exchange) : DSP+DMP, SSP
* DMP
	* 비식별 빅데이터를 세분화 시켜서 광고에 반응할 잠재고객 데이터를 수집
	* DMP는 “정확한 소비자”를 찾아 광고를 노출시킬 모수를 확보하는 역할

## Requirement
* Functional requirement
	* 데이터 유실 방지 및 서비스 영속성을 위한 레이어별 장애 처리
	* 공통, 상품수집, 클릭, 컨버전 처리를 위한 API 구현
	* TO-BE 데이터 모델 설계 및 AS-IS 데이터 연계 : 한시적으로 데이터 2중으로 데이터 적재
* Non Functional requirement
	* 서버의 장애가 연동 고객사에 영향이 가지 않도록 (장애전파x)  구현
	* 데이터 수집 응답시간 100ms 비동기 처리
	* 데이터/트래픽 증가에 따른 수평확장
	* 서비스 모듈화 및 데이터 분산 (수직분할)
		* 다양한 데이터 수집/적재/분석 모듈 추가를 위한 기반 구축

## 3th Party Solution
* Apache Kafka (MQ) : 데이터 수집, 적재 분리. 비동기 처리
* MongoDB (No-SQL) : 상품 데이터 저장/조회
* ELK Stack(Elasticsearch, Logstash, Kibana) : 로그 데이터 저장/조회

