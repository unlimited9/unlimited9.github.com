## Reference Notes
> 기술 적용/검토 내용 정리

#### EA, ITA/EA
기업의 비즈니스, 데이터, 기술, 보안 등 주요 요건을 체계적으로 분석하여 기업의 현재 모습을 조감하고 앞으로의 지침을 제공하기 위한 아키텍처
> EA와 ITA가 혼용되고 사용되고 있으나, EA로 통일되어 가고 있음.
 - ITA, Information Technology Architecture, 정보 기술 아키텍처
    - = EA(Enterprise Archtecture) + TRM(Technical Reference Model) + SP(Standards Profiling)
 - EA, Enterprise Archtecture, 전사적 아키텍처와
    - = BA(Business Architecture) + DA(Data Architecture) + AA(Application Architecture) + TA(technical architecture)
---
#### AA(Application Architecture)
* Development Environment
   + [Install and setup Java](architecture.application/java/install.n.setup.jdk.md) ([install script](architecture.application/java/install.n.setup.script.jdk.md)) : JDK(Java™ Platform, Development Kit) 설치
   + [Eclipse IDE Setup](eclipse.ide.setup.md) : J2EE 개발을 위한 통합개발환경 도구 Eclipse IDE 설치
* Language
   + JAVA
      - [Collections.sort with multiple fields](architecture.application/java/collections.sort.sample.md)
      - [JAXB on Java 9 and beyond](architecture.application/java/jaxb.on.java.9.n.beyond.md)
      - [Netty network application framework](architecture.application/java/netty.nio.network.framework.md)
   + Groovy
   + Javascript
* Build
   + Maven
   + Gradle
* Framework
   + Spring
      - [Spring Boot : Management and Operations](architecture.application/framework/springboot.management.n.operations.md)
      - [Overriding Dependency Versions with Spring Boot](architecture.application/framework/springboot.transitive.dependency.md)
---
#### TA(technical architecture)
* Operating System
   + [System monitoring](architecture.quality.assurance/system.monitoring.md) : 시스템 상태 조회
   + [Account/Group management](operating.system/management.account.n.group.md) : 시스템 계정/그룹 관리
   + [Ditectory management](operating.system/management.directory.md) : 시스템 디렉토리 정의/관리
   + [Openssl : self-signed certificate](operating.system/openssl.self.signed.certificate.md) : SSL 인증서 생성
* Middle-ware
   + [Nginx : Install and setup](architecture.solution/nginx/install.n.setup.md) : Nginx Web Server 설치
      - [Optimize : performance tunning](architecture.solution/nginx/optimize.performance.tunning.md) : 성능 최적화
      - [Install PHP](architecture.solution/nginx/install.n.setup.php.md) : PHP 연계 구성
      - [Configuration Sample](architecture.solution/nginx/configuration.sample.md) : 설정 예
   + [Apache Tomcat : Install and setup](architecture.solution/apache.tomcat/install.n.setup.md) ([install script](architecture.solution/apache.tomcat/install.n.setup.script.md)) : Tomcat Servlet Container - Multi Instances
      - [Operation and Maintenance](architecture.solution/apache.tomcat/operation.n.maintenance.md) : 운영 관리
   + [Node.js : Install and setup](architecture.solution/node.js/install.n.setup.md) : Node.js 설치
      - [Dockerizing](architecture.solution/node.js/dockerizing.md) : 도커 이미지 만들기
   + [Hadoop ecosystem : Introduction](architecture.solution/hadoop/introduction.md) : Hadoop이란
   + [Apache Kafka : Install and setup](architecture.solution/apache.kafka/install.n.setup.md) : Kafka Message Queue 서버 설치
      - [Kafka Manager : Monitoring](architecture.solution/apache.kafka/install.n.setup.kafka.manager.md) : Kafka 모니터링 
   + [Haproxy : Install and setup](architecture.solution/haproxy/install.n.setup.md) : Haproxy TCP/HTTP Load Balancer 설치
* Cloud Computing
   + [Docker : Install and setup](architecture.solution/docker/install.n.setup.md) ([install script](architecture.solution/docker/install.n.setup.script.md)) : Docker(independent container platform) 설치
      - [Administration : management and dockerize](architecture.solution/docker/administration.management.md) : 도커 관리
      - [Create image and container](architecture.solution/docker/create.image.n.container.md) : 도커 이미지/컨테이너 생성
      - [Container orchestration and clustering](architecture.solution/docker/orchestration.n.clustering.md) : 분산 클러스터링
      - [Docker Change Port Mapping for an Existing Container](architecture.solution/docker/change.port.mapping.md) : 도커 포트 변경
      - [Docker private registry](architecture.solution/docker/private.registry.md) : 도커 사설 레지스트리 구성
   + [Kubernetes(k8s) : Concept and theorem](architecture.solution/kubernetes/concept.theorem.md) : 쿠버네티스 개요
      - [Install and setup](architecture.solution/kubernetes/install.n.setup.md) ([install script](architecture.solution/kubernetes/install.n.setup.script.md)) : Kubernetes(Docker orchestration and clustering) 설치
      - [Install and setup HA, multi-master](architecture.solution/kubernetes/master.node.cluster.ha.md) : 작성중... Marster HA 구성
      - [Web UI (Dashboard) : Install and setup](architecture.solution/kubernetes/install.n.setup.dashboard.md) : 웹 관리 모듈 설치
      - [Administration : management](architecture.solution/kubernetes/administration.management.md) : 쿠버네티스 관리
      - [Kubernetes deployment](architecture.solution/kubernetes/how.to.deployment.md) : 쿠버네티스 서비스 디플로이(tutorial)
      - [Kubernetes ConfigMap and Secret](architecture.solution/kubernetes/configMap.n.secret.md) : 어플리케이션 설정 분리
* Hardware Sizing : 규모산정  
   + [hardware sizing](operating.system/hardware.sizing.md) : 정보시스템 하드웨어 규모산정 지침
* Development Environment  
   + [gitlab ci/cd](architecture.solution/gitlab/gitlab.ci.cd.md) : Gitlab Runner
---
#### QA(Quality Assurance)
* Testing and Tuning
   + [Performance testing and tuning](architecture.quality.assurance/performance.testing.and.tuning.md) : 성능 테스트 및 환경 구성
      - [Install and setup nGrinder](architecture.quality.assurance/install.n.setup.ngrinder.md) : 성능 및 스트레이스 테스트를 위한 nGrinder 설치 ([Groovy Script](architecture.quality.assurance/ngrinder.groovy.script.md))
      - [Install and setup Scouter](architecture.quality.assurance/install.n.setup.scouter.md) : APM(Application Performance Management) Scouter 설치
* Monitoring
   + [Install and setup Telegraf](architecture.quality.assurance/install.n.setup.telegraf.md) : 시스템 모니터링 및 지표 수집 에이전트
---
#### Architecture style(pattern, model)
* [MSA(Microservices Architecture)](architecture.style/MSA/concept.md) : 서비스 분산, 독립적인 배포 및 확장
