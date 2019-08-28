## Reference Notes
AA/TA, Java/Framework, Design/Prototyping/Optimization

### EA(Enterprise Architecture)

1. BA(Business Architecture)

2. AA(Application Architecture)
    * Development Environment
      + [Install and setup Java](AA/JDK/install.n.setup.md) ([install script](AA/JDK/install.n.setup.script.md)) : JDK(Java™ Platform, Development Kit) 설치
      + [Eclipse IDE Setup](eclipse.ide.setup.md) : J2EE 개발을 위한 통합개발환경 도구 Eclipse IDE 설치
      + [Apache Tomcat Setup](AA/apache.tomcat/install.n.setup.md) ([install script](AA/apache.tomcat/install.n.setup.script.md)) : Tomcat Servlet Container - Multi Instances
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
    * Middle-ware
      + [Install and setup Nginx](TA/nginx/install.n.setup.md) : Nginx Web Server 설치
        - [Optimize : performance tunning](TA/nginx/optimize.performance.tunning.md) : 성능 최적화
        - [Install PHP](TA/nginx/install.n.setup.php.md) : PHP 연계 구성
        - [Configuration Sample](TA/nginx/configuration.sample.md) : 설정 예
      
4. DA(technical architecture)

5. QA(Quality Assurance)
    * Testing and Tuning
      + [Performance testing and tuning](QA/performance.testing.and.tuning.md) : 성능 테스트 및 환경 구성
        - [Install and setup nGrinder](QA/install.n.setup.ngrinder.md) : 성능 및 스트레이스 테스트를 위한 nGrinder 설치 ([Groovy Script](QA/ngrinder.groovy.script.md))
        - [Install and setup Scouter](QA/install.n.setup.scouter.md) : APM(Application Performance Management) Scouter 설치
