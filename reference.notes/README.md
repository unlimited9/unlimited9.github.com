## Reference Notes
AA/TA, Java/Framework, Design/Prototyping/Optimization

#### EA(Enterprise Architecture)

1. BA(Business Architecture)

2. AA(Application Architecture)
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
        - [Spring Boot : Management and Operations](AA/Framework/springboot.management.n.operations.md)
        - [Overriding Dependency Versions with Spring Boot](AA/Framework/springboot.transitive.dependency.md)

3. TA(technical architecture)
    * Operating System
      + [Account/Group management](TA/system/management.account.n.group.md) : 시스템 계정/그룹 관리
      + [Ditectory management](TA/system/management.directory.md) : 시스템 디렉토리 정의/관리
      + [Openssl : self-signed certificate](TA/system/openssl.self.signed.certificate.md) : SSL 인증서 생성
    * Middle-ware
      + [Nginx : Install and setup](TA/nginx/install.n.setup.md) : Nginx Web Server 설치
        - [Optimize : performance tunning](TA/nginx/optimize.performance.tunning.md) : 성능 최적화
        - [Install PHP](TA/nginx/install.n.setup.php.md) : PHP 연계 구성
        - [Configuration Sample](TA/nginx/configuration.sample.md) : 설정 예
      + [Apache Tomcat : Install and setup](TA/apache.tomcat/install.n.setup.md) ([install script](TA/apache.tomcat/install.n.setup.script.md)) : Tomcat Servlet Container - Multi Instances
        - [Operation and Maintenance](TA/apache.tomcat/operation.n.maintenance.md) : 운영 관리
      + [Node.js : Install and setup](TA/node.js/install.n.setup.md) : Node.js 설치
        - [Dockerizing](TA/node.js/dockerizing.md) : 도커 이미지 만들기
      + [Hadoop ecosystem : Introduction](TA/hadoop/introduction.md) : Hadoop이란
      + [Apache Kafka : Install and setup](TA/apache.kafka/install.n.setup.md) : Kafka Message Queue 서버 설치
        - [Kafka Manager : Monitoring](TA/apache.kafka/install.n.setup.kafka.manager.md) : Kafka 모니터링 
      + [Haproxy : Install and setup](TA/haproxy/install.n.setup.md) : Haproxy TCP/HTTP Load Balancer 설치
    * Cloud Computing
      + [Docker : Install and setup](TA/cloud/docker/install.n.setup.md) ([install script](TA/cloud/docker/install.n.setup.script.md)) : Docker(independent container platform) 설치
        - [Administration : management and dockerize](TA/cloud/docker/administration.management.md) : 도커 관리
        - [Create image and container](architecture.solution/docker/create.image.n.container.md) : 도커 이미지/컨테이너 생성
        - [Container orchestration and clustering](TA/cloud/docker/orchestration.n.clustering.md) : 분산 클러스터링
        - [Docker Change Port Mapping for an Existing Container](TA/cloud/docker/change.port.mapping.md) : 도커 포트 변경
        - [Docker private registry](architecture.solution/docker/private.registry.md) : 도커 사설 레지스트리 구성
      + [Kubernetes(k8s) : Concept and theorem](architecture.solution/kubernetes/concept.theorem.md) : 쿠버네티스 개요
        - [Install and setup](architecture.solution/kubernetes/install.n.setup.md) ([install script](architecture.solution/kubernetes/install.n.setup.script.md)) : Kubernetes(Docker orchestration and clustering) 설치
        - [Install and setup HA, multi-master](architecture.solution/kubernetes/master.node.cluster.ha.md) : 작성중... Marster HA 구성
        - [Web UI (Dashboard) : Install and setup](architecture.solution/kubernetes/install.n.setup.dashboard.md) : 웹 관리 모듈 설치
        - [Administration : management](architecture.solution/kubernetes/administration.management.md) : 쿠버네티스 관리
        - [Kubernetes deployment](architecture.solution/kubernetes/how.to.deployment.md) : 쿠버네티스 서비스 디플로이(tutorial)
        - [Kubernetes ConfigMap and Secret](architecture.solution/kubernetes/configMap.n.secret.md) : 어플리케이션 설정 분리
    * Hardware Sizing : 규모산정  
      + [hardware sizing](TA/system/hardware.sizing.md) : 정보시스템 하드웨어 규모산정 지침
    * Development Environment  
      + [gitlab ci/cd](TA/gitlab/gitlab.ci.cd.md) : Gitlab Runner
     
4. DA(technical architecture)

5. QA(Quality Assurance)
    * Testing and Tuning
      + [Performance testing and tuning](QA/performance.testing.and.tuning.md) : 성능 테스트 및 환경 구성
        - [Install and setup nGrinder](QA/install.n.setup.ngrinder.md) : 성능 및 스트레이스 테스트를 위한 nGrinder 설치 ([Groovy Script](QA/ngrinder.groovy.script.md))
        - [Install and setup Scouter](QA/install.n.setup.scouter.md) : APM(Application Performance Management) Scouter 설치
    * Monitoring
      + [Install and setup Telegraf](QA/install.n.setup.telegraf.md) : 시스템 모니터링 및 지표 수집 에이전트

#### Architecture style(pattern, model)

1. [MSA(Microservices Architecture)](architecture.style/MSA/concept.md) : 서비스 분산, 독립적인 배포 및 확장
