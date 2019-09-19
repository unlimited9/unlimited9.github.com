## [Reference Notes](../)
AA/TA, Java/Framework, Design/Prototyping/Optimization
 
#### EA(Enterprise Architecture) / 3. TA(technical architecture)

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
* Cloud Computing
  + [Docker : Install and setup](TA/cloud/docker/install.n.setup.md) ([install script](TA/cloud/docker/install.n.setup.script.md)) : Docker(independent container platform) 설치
    - [Administration : management and dockerize](TA/cloud/docker/administration.management.md) : 도커 관리
    - [Create image and container](TA/cloud/docker/create.image.n.container.md) : 도커 이미지/컨테이너 생성
    - [Container orchestration and clustering](TA/cloud/docker/orchestration.n.clustering.md) : 분산 클러스터링
    - [Docker Change Port Mapping for an Existing Container](TA/cloud/docker/change.port.mapping.md) : 도커 포트 변경
    - [Docker private registry](TA/cloud/docker/private.registry.md) : 도커 사설 레지스트리 구성
  + [Kubernetes(k8s) : Concept and theorem](TA/cloud/kubernetes/concept.theorem.md) : 쿠버네티스 개요
    - [Install and setup](TA/cloud/kubernetes/install.n.setup.md) ([install script](TA/cloud/kubernetes/install.n.setup.script.md)) : Kubernetes(Docker orchestration and clustering) 설치
    - [Web UI (Dashboard) : Install and setup](TA/cloud/kubernetes/install.n.setup.dashboard.md) : 웹 관리 모듈 설치
    - [Administration : management](TA/cloud/kubernetes/administration.management.md) : 쿠버네티스 관리
    - [Kubernetes deployment](TA/cloud/kubernetes/how.to.deployment.md) : 쿠버네티스 서비스 디플로이(tutorial)
