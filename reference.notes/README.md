## Reference Notes
AA/TA, Java/Framework, Design/Prototyping/Optimization

### EA(Enterprise Architecture)

1. BA(Business Architecture)

2. AA(Application Architecture)
    * Development Environment
      + [Install and setup Java](AA/JDK/install.n.setup.md) ([install script](AA/JDK/install.n.setup.script.md)) : JDK(Java™ Platform, Development Kit) 설치
      + [Eclipse IDE Setup](eclipse.ide.setup.md) : J2EE 개발을 위한 통합개발환경 도구 Eclipse IDE 설치
    * Language
      + JAVA
        - [Collections.sort with multiple fields](AA/Java/collections.sort.sample.md)
        - [JAXB on Java 9 and beyond](AA/Java/jaxb.on.java.9.n.beyond.md)
      + Groovy
      + Javascript
    * Build
      + Maven
      + Gradle
    * Framework
      + Spring
        - [Overriding Dependency Versions with Spring Boot](AA/Framework/springboot.transitive.dependency.md)

3. TA(technical architecture)
    * Operating System
      + [Account/Group management](TA/system/management.account.n.group.md) : 시스템 계정/그룹 관리
      + [Ditectory management](TA/system/management.directory.md) : 시스템 디렉토리 정의/관리
    * Middle-ware
      + [Nginx : Install and setup](TA/nginx/install.n.setup.md) : Nginx Web Server 설치
        - [Optimize : performance tunning](TA/nginx/optimize.performance.tunning.md) : 성능 최적화
        - [Install PHP](TA/nginx/install.n.setup.php.md) : PHP 연계 구성
        - [Configuration Sample](TA/nginx/configuration.sample.md) : 설정 예
      + [Apache Tomcat : Install and setup](TA/apache.tomcat/install.n.setup.md) ([install script](TA/apache.tomcat/install.n.setup.script.md)) : Tomcat Servlet Container - Multi Instances
      + [Node.js : Install and setup](TA/node.js/install.n.setup.md) : Node.js 설치
      + [Hadoop ecosystem : Introduction](TA/hadoop/introduction.md) : Hadoop이란
    * Cloud Computing
      + [Docker : Install and setup](TA/cloud/docker/install.n.setup.md) ([install script](TA/cloud/docker/install.n.setup.script.md)) : Docker(independent container platform) 설치
        - [Administration : management and dockerize](TA/cloud/docker/administration.management.md) : 도커 관리
        - [Container orchestration and clustering](TA/cloud/docker/orchestration.n.clustering.md) : 분산 클러스터링
        - [Docker Change Port Mapping for an Existing Container](TA/cloud/docker/change.port.mapping.md) : 도커 포트 변경
      + [Kubernetes(k8s) : Concept and theorem](TA/cloud/kubernetes/concept.theorem.md) : 쿠버네티스 개요
        - [Install and setup](TA/cloud/kubernetes/install.n.setup.md) ([install script](TA/cloud/kubernetes/install.n.setup.script.md)) : Kubernetes(Docker orchestration and clustering) 설치
        - [Web UI (Dashboard) : Install and setup](TA/cloud/kubernetes/install.n.setup.dashboard.md) : 웹 관리 모듈 설치
        - [Administration : management](TA/cloud/kubernetes/administration.management.md) : 쿠버네티스 관리

4. DA(technical architecture)

5. QA(Quality Assurance)
    * Testing and Tuning
      + [Performance testing and tuning](QA/performance.testing.and.tuning.md) : 성능 테스트 및 환경 구성
        - [Install and setup nGrinder](QA/install.n.setup.ngrinder.md) : 성능 및 스트레이스 테스트를 위한 nGrinder 설치 ([Groovy Script](QA/ngrinder.groovy.script.md))
        - [Install and setup Scouter](QA/install.n.setup.scouter.md) : APM(Application Performance Management) Scouter 설치
