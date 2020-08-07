# Micro-Service Architecture

>Created 목요일 30 11월 2017  
MSA is a style of architecture that defines and creates systems through the use of small independent and self-contained services aligned closely with business activities.

## Introduction

#### Inner architecture

#### Outer architecture
1. External Gateway  
2. Service Mesh  
3. Container Management  
4. Backing Service  
5. Telemetry  
6. CI/CD Automation  

#### External Gateway
서비스 외부로부터 들어오는 접근을 내부 구조를 드러내지 않고 처리하기 위한 요소
사용자 인증(Consumer Identity Provider)과 권한 정책관리(policy management)를 수행
API Gateway는 서버 최앞단에 위치하여 모든 API 호출을 받아 서비스 인증 후 적절한 서비스들에 메세지를 전달(routing)

	기능 
		1. 인증 및 인가 ( Authentication and Authorization)
		2. 요청 절차의 단순화
		3. 라우팅 및 로드밸런싱
		4. 서비스 오케스트레이션
		5. 서비스 디스커버리

	고려사항 
		SOA에서의 ESB(Enterprise Service Bus)에서 발생했던 문제점
			ESB 내부 처리 로직을 XML을 기반으로 하였는데, XML의 파싱에 오버헤드 발생
			ESB는 가벼운 연산 뿐만 아니라, 과도한 Orchestration 등 무거운 로직 적용(ESB를 Gateway로의 특성이 아닌 시스템을 통합하기 위한 역할 구현)
		Scale-out 적용이 유연하게 일어나지 않을 경우, API Gateway가 병목지점이 되어 어플리케이션의 성능 저하의 원인
		API Gateway라는 추가적인 계층이 만들어지는 것이기 때문에, 그만큼 네트워크 latency가 증가

2. Service Mesh  
마이크로서비스 구성 요소간의 네트워크를 제어  
서비스 간에 통신을 하기 위해서는 service discovery, service routing, 트래픽 관리 및 보안 등을 담당 처리  
Service Oriented Architecture(SOA) : Enterprise Service Bus(ESB)  
  
	기능
		Service Discovery
		Load balancing (지연시간 기반 / 대기열 기반 )
		Dynamic Request Routing
		Circuit Breaking
    Retry and Timeout
		암호화 (TLS)
		보안
    Distributed Tracing
		Metric 수집
    Health check

	API Gateway vs Service Mesh 
		API Gateway는 마이크로서비스 그룹의 외부 경계에 위치하여 역할을 수행
		ServiceMesh는 경계 내부에서 그 역할을 수행

		API Gateway가 중앙집중형 아키텍쳐여서 SPOF(Single Point of Failure)을 생성
		Service Mesh는 분산형 아키텍쳐를 취하기 때문에 SPOF를 생성하지 않고 확장이 용이

		API Gateway는 일반적으로 Gateway proxy pattern을 사용해서 수행 : Consumer(호출자)은 구현 내용을 알 필요 없이 Gateway를 호출하는 방법만 알면 Gateway가 알아서 수행
		Service Mesh는 일반적으로 Sidecar proxy pattern을 사용합니다. Consumer(호출자)의 코드에는 Provider(공급자)의 주소를 찾는방법, failover와 관련된 코드 등의 내용이 들어가게 됩니다. 다만, 호출자의 코드는 어플리케이션 코드(비즈니스 로직)에 내장되는 것이 아니라 sidecar 형태로 별개로 관리됩니다.

		라우팅 주체		서버
						요청하는 서비스
		라우팅 구성요소	별도의 네트워크를 도입하는 독립적인 API gateway 구성요소
						서비스 내 sidecar로 Local network 스택의 일부가 됨
		로드 밸런싱		단일 엔드포인트를 제공하고, API Gateway 내 로드밸런싱을 담당하는 구성요소에 요청을 redirection하여 해당 구성요소가 처리
						Service Registry에서 서비스 목록을 수신함. sidecar에서 로드밸런싱 알고리즘을 통해 수행함
		네트워크 			외부 인터넷과 내부 서비스 네트워크 사이에 위치함
						내부 서비스 네트워크 사이에 위치하며, 응용프로그램의 네트워크 경계 내에서만 통신을 가능하게 함
		분석 			API에 대한 사용자 및 공급자에 대한 모든 호출에 대해 수집되고 분석됨	
						Mesh 내 모든 마이크로서비스 구성요소에 대해 분석가능




3. Container Management
컨테이너 기반 어플리케이션 운영은 유연성과 자율성을 가지며, 개발자가 손쉽게 접근 및 운영할 수 있는 인프라 관리 기술이기 때문에 MSA에 적합하다고 평가
대표적인 컨테이너 관리 환경인 Kubernetes가 Container management에 많이 사용
AWS의 EKS, Google cloud platform의 GKE는 kubernetes를 지원하는 클라우드 서비스

4. Backing Service
어플리케이션이 실행되는 가운데 네트워크를 통해서 사용할 수 있는 모든 서비스
My SQL과 같은 데이터베이스, 캐쉬 시스템, SMTP 서비스 등 어플리케이션과 통신하는 attached Resource들을 지칭하는 포괄적인 개념

MSA에서는 메세지의 송신자와 수신자가 직접 통신하지 않고 Message Queue를 활용하여 비동기적으로 통신하는 것을 지향

	Message queue
		Message Queue(MQ)는 메세지를 발행하는 생산자(producer, publisher) 와 메세지를 받는 소비자(consumer, subscriber) 사이에 위치하는 매개체역할을 수행하는 미들웨어입니다.
		Message queue를 이용하여 비동기 방식으로 통신
			비동기 / 약결합
			Redundancy : 서비스가 실패하더라도 재처리를 할 수 있습니다.
			Resilence : 전체 시스템에 대한 영향이 적고, 회복 탄력성이 높습니다.
			보장성 : Message Queue에 들어간 Message는 최소 한번은 처리됩니다.
			확장성 : 다수의 Message queue를 두어 많은 요청에 대해 유연하게 확장할 수 있습니다.
		Apache Kafka
			Kafka는 Pub/Sub 구조에 특화된 메세지 큐이고, 분산 환경에 특화되어 설계되어있습니다.
			RabbitMQ나, ActiveMQ에 비해 높은 처리량, 및 성능을 가지고 있습니다.

			특징
				대용량 및 실시간 처리에 특화되어있습니다.
				단순한 TCP 기반의 프로토콜을 사용함으로 오버헤드가 적습니다.
				메모리가 아닌 파일시스템에 메세지를 저장합니다.
				OS에서 처리하는 페이지 캐시를 이용함으로써 속도가 빠릅니다.
				consumer가 pull 방식으로 메세지를 받으며, batch 처리가 가능합니다.

5. Telemetry
Tele(먼 거리) + metry(측정) : 실시간으로 먼 거리에서 원격으로 측정
서비스들을 모니터링하고, 서비스별로 발생하는 이슈들에 대응할 수 있도록 환경을 구성

	주요기능
		1. Monitoring
			metric 수집, logging, Tracing 영역에서의 데이터 수집 및 분석
			
			AWS의 Cloud watch, Azure의 Monitor, GCP의 Stackdriver가 Public cloud에서 Monitoring을 담당하는 요소이며, OSS로는 Prometheus등이 있습니다.
			Monitoring은 각 대상에서 수집한 Metric을 통해 대상의 리소스 사용률이나, 트래픽 등을 대시보드로 볼 수 있게 해줍니다.

		2.Logging
			Log는 실행중인 프로세스에서 발생하는 이벤트를 말하며, Logging은 이러한 Log들을 수집해서 보여주는 것을 말합니다.
			마이크로 서비스 아키텍쳐는 Monolithic 어플리케이션보다 실행하는 프로세스의 수가 훨씬 많기 때문에 장애가 발생시에 root cause를 파악하기 힘듭니다.
			
			AWS에서는 Amazon Elastic Search 등이 Logging을 담당하는 요소이며, OSS로는 EFK(Elastic Search - FluentD - Kibana) 가 대표적입니다.
			각 서비스에 심어진 Agent가 해당 서비스의 Log정보들을 수집하고, Log Server(Aggregator)를 통해 취합됩니다. 최종적으로 Kibana와 같은 툴을 통해 시각화된 데이터를 관리자에게 보여줍니다.

			고려사항
				로그를 서비스 내 저장소에 저장할 경우, 관리가 어려워지므로 ( container managed service의 경우 컨테이너가 종료되거나 재시작 될 경우 사라질 수 있습니다.) 로그를 어플리케이션 서버 내부에 저장하지 않고 외부 저장소에 저장하도록 합니다.
				MSA 에서 가상머신과 컨테이너는 application과의 결합이 "약한 결합"입니다. 즉, 배포에 사용되는 컨테이너는 배포할 때마다 달라질 수 있습니다. 따라서 디스크 저장 상태에 의존하는 것이 아니라, 별도의 Logging도구를 활용하는 것이 효율적입니다.
				로그파일은 ACL을 통하여 로그 관리자만 로그를 확인 할 수 있도록 하여야 합니다.
		3. Tracing
			Tracing은 마이크로 서비스 아키텍처에서 발생한 Event를 발생한 순서대로 나열하여 추적할 수 있게 해주는 기능입니다.
			AWS에서의 Amazon X-Ray 등이 Tracing을 담당하는 요소이며, OSS로는 Zipkin Jaeger등이 대표적입니다.

6. CI/CD Automation
CI/CD는 어플리케이션 개발 단계를 자동화하여, 어플리케이션을 보다 짧은 주기로 고객에게 제공
지속적인 통합(Continuous Integration), 지속적인 전달(Continuous Delivery), 지속적인 배포(Continuous Deployment)가 CI/CD의 기본 개념



## 9. Appendix

#### reference site

* What are microservices?  
https://microservices.io/  

+ Building Microservices: Using an API Gateway  
https://www.nginx.com/blog/building-microservices-using-an-api-gateway/  

+  마이크로서비스 Microservices (2) API 게이트웨이  
https://futurecreator.github.io/2018/09/14/microservices-with-api-gateway/  

+  MSA 제대로 이해하기 -(4)Service Mesh  
https://velog.io/@tedigom/MSA-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-4Service-Mesh-f8k317qn1b  

+ Service Mesh 란?  
https://medium.com/dtevangelist/service-mesh-%EB%9E%80-8dfafb56fc07  

+ [MSA] Internal LoadBalancer Service Mesh  
https://waspro.tistory.com/434  
