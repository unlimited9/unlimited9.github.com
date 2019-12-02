# administration : kubernetes

>Created 월요일 20 11월 2017  
Container orchestration and clustering

## Getting Started

#### A. Pod

`pull docker image from docker hub`  
$ docker pull mobon/apps.server
```
Using default tag: latest
Error response from daemon: manifest for mobon/apps.server:latest not found
```
$ docker pull mobon/apps.server:1.1
```
1.1: Pulling from mobon/apps.server
8ba884070f61: Pull complete 
d7d7ee9860ef: Pull complete 
Digest: sha256:0f74b9019ea2df34e1d2f0f93245f7dd087b3c46e5ed7c14d1ac2b56e5a65584
Status: Downloaded newer image for mobon/apps.server:1.1
```

`docker images`  
$ docker images
```
REPOSITORY                           TAG                 IMAGE ID            CREATED             SIZE
...
mobon/apps.server                    1.1                 bb8349385d53        4 weeks ago         370MB
...
```

>`image tag`  
>$ docker tag mobon/apps.server mobon/apps.server:latest
>
>`image push`  
>$ docker push mobon/apps.server

`deployment yaml 파일 작성`  
$ vi gs-spring-boot-docker-deployment.yaml
```
# Kubernetes API version
apiVersion: apps/v1beta2
# Deployment object
kind: Deployment
metadata:
  # Unique-key(search)
  name: gs-spring-boot-docker-deployment
  # Deployment object group
  # 복수 설정 할 수 있으며 같은 label을 가진 object들을 같은 그룹으로 식별
  labels:
    app: gs-spring-boot-docker
spec:
  # 복제해야 할 Pod의 개수(Deployment object가 ReplicaSets object를 통해)
  replicas: 1
  selector:
    # Deployment object가 관리해야할 Pod(Pod의 label 정보)
    matchLabels:
      app: gs-spring-boot-docker
  # Deployment object가 생성할 Pod 관련 설정 입니다
  template:
    metadata:
      labels:
        app: gs-spring-boot-docker
    # Deployment object가 생성할 Pod에 대한 설정
    spec:
      # Deployment object가 생성할 Pod가 관리하는 container들의 설정
      containers:
        # container name
      - name: gs-spring-boot-docker
        # container image name(tag)
        image: nara0617/gs-spring-boot-docker:1.0
        # port(여러개)
        ports:
        - containerPort: 8080
        # Always, IfNotPresent
        imagePullPolicy: Always
        resources:
          # 컨테이너 최소 리소스
          # Spring Boot 애플리케이션의 경우 메모리 값을 256M 이상 설정
          requests:
            memory: "256Mi"
            cpu: "200m"
          # 컨테이너 최대 사용 리소스
          # 애플리케이션에 따라 적절한 CPU와 메모리 값으로 설정
          limits:
            memory: "1Gi"
            cpu: "500m"
```

----------

- 생성된 Docker  Image를 확인해보면

#### B. Service

#### C. Application 관리

#### D. Rolling Update (무중단 배포)

#### E. HPA (오토스케일링)

#### F. Ingress 활용

#### G. Kubernetes 기반의 PaaS 비교

## 9. Appendix

#### reference site

+ [Kubernetes & Docker] KubernetesPod  생성 가이드  
https://waspro.tistory.com/368?category=831751

+ 쿠버네티스 Docker Hub (public/private) 사용  
https://sarc.io/index.php/cloud/1378-docker-hub-public-private

+ 쿠버네티스 #4 - 아키텍쳐  
https://bcho.tistory.com/1258?category=731548

+ 쿠버네티스를 이용해 테스팅 환경 구현해보기  
http://woowabros.github.io/experience/2018/03/13/k8s-test.html


`kubernetes master node ha`  
* 고가용성 쿠버네티스 클러스터 마스터 설정하기  
https://kubernetes.io/ko/docs/tasks/administer-cluster/highly-available-master/

+ Docker with Kubernetes #9 - Master Node의 HA 구성  
https://crystalcube.co.kr/203

