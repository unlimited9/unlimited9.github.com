# GitLab CI/CD

>Created 목요일 30 11월 2017  
Project planning and source code management to CI/CD, monitoring, and security

## GitLab CI/CD Pipeline Configuration Sample
>create .gitlab-ci.yml file repository root path

#### java build and deploy
$ vi .gitlab-ci.yml
```yaml
# Define stages in a pipeline
stages:
  - build
  - test
  - deploy

before_script:
  # set java environment path
  - source /etc/profile.d/java11.sh
  #- pwd

cache:
  paths:
    - mobon.ad.tracker/mobon.tracker.application/build

# build to development server
build-to-development:
  stage: build
  environment:
    name: development deploy server
  only:
    - master
  script:
    - gradle -v
    - gradle --build-file mobon.ad.tracker/build.gradle :mobon.tracker.application:build
  tags:
    - development
    
# test to development server
test-to-development:
  stage: test
  environment:
    name: development deploy server
  only:
    - master
  script:
    - echo 'test to development server'
  tags:
    - development
    
# deploy to development server
deploy-to-development:
  stage: deploy
  environment:
    name: development deploy server
  only:
    - master
  script:
    # war(package) file transfer to devlepment server
    - scp -i ~/.ssh/app -P 7722 mobon.ad.tracker/mobon.tracker.application/build/libs/*.war app@172.20.0.103:/pgms/tomcat/wars/mobon.tracker
    # development server backup/deploy/restart over ssh
    - ssh -i ~/.ssh/app -t -p 7722 app@172.20.0.103 "sh /pgms/tomcat/ctrl.sh mobon.tracker backup && sh /pgms/tomcat/ctrl.sh mobon.tracker deploy && sh /pgms/tomcat/ctrl.sh mobon.tracker restart"
  after_script:
    - echo 'deploy completed'
  tags:
    - development

```

#### java build and manual deploy
> when: manual

$ vi .gitlab-ci.yml
```yaml
# Define stages in a pipeline
stages:
  - build
  - test
  - deploy

before_script:
  # set java environment path
  - source /etc/profile.d/java11.sh
  #- pwd

cache:
  paths:
    - mobon.ad.utils/build

# build to development server
build-to-development:
  stage: build
  environment:
    name: development deploy server
  only:
    - master
  script:
    - gradle -v
    - gradle --build-file mobon.ad.utils/build.gradle :build
  tags:
    - development
    
# test to development server
test-to-development:
  stage: test
  environment:
    name: development deploy server
  only:
    - master
  script:
    - echo 'test to development server'
  tags:
    - development
    
# deploy to development server
deploy-to-development:
  stage: deploy
  environment:
    name: development deploy server
  only:
    - master
  script:
    # upload to nexus
    - curl --upload-file mobon.ad.utils/build/libs/enliple-utils-0.1.0.jar -u admin:P@ssw0rd -v http://172.20.0.7:8081/repository/maven-releases/com/mobon/mobon.ad.utils/0.1.0/mobon.ad.utils-0.1.0.jar
  after_script:
    - echo 'deploy completed'
  when: manual
  tags:
    - development

```

>
>#### Upload Nexus
>$ curl --upload-file mobonUtils/mobon.ad.utils/build/libs/enliple-utils-0.1.0.jar -u admin:P@ssw0rd -v http://172.20.0.7:8081/repository/maven-releases/com/mobon/mobon.ad.utils/0.1.0/mobon.ad.utils-0.1.0.jar
>>curl -v -F r=releases -F hasPom=false -F e=jar -F g=com.mobon -F a=mobon.ad.utils -F v=0.1.0 -F p=jar -F file=@"./mobonUtils/mobon.ad.utils/build/libs/enliple-utils-0.1.0.jar" -u admin:P@ssword http://172.20.0.7:8081/repositories/maven-releases/com/mobon/mobon.ad.utils/0.1.0/enliple-utils-0.1.0.jar
>#### Error
>```
>...`
> * We are completely uploaded and fine
>< HTTP/1.1 400 Repository does not allow updating assets: maven-releases
>...
>```gitlab-runner
>
>##### Solution : Nexus repository setting
>Hosted / Deployment policy:
>Controls if deployments of and updates to artifacts are allowed
  Set : Allow redeploy

## C. GitLab Runner

> 배포서버 : 172.20.0.7:7722

#### gitlab-runner  container 실행
$ docker run -d --name gitlab-runner --restart always \
-v /srv/gitlab-runner/config:/etc/gitlab-runner \
-v /var/run/docker.sock:/var/run/docker.sock \
gitlab/gitlab-runner:latest

#### get the logs
$ docker logs gitlab-runner

#### gitlab-runner container 접속
$ docker exec -it gitlab-runner /bin/bash

#### gitlab-runner 등록
$ gitlab-runner register
```
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com )
http://172.20.0.7:9000/

Please enter the gitlab-ci token for this runner:
xxxxxxxxxxxxx

Please enter the gitlab-ci description for this runner:
deploy runner for development

Please enter the gitlab-ci tags for this runner (comma separated):
development

Please enter the executor: docker-ssh, ssh, virtualbox, docker, parallels, shell, docker+machine, docker-ssh+machine, kubernetes:
shell
```
>+ Settings > CI/CD > Runners settings > Specific Runners 에서 URL과 토큰 값을 확인 할 수 있다.
>+ gitlab-ci tags에는 .gitlab-ci.yml에 설정한 필요한 stages에 대한 tags를 넣어줘야 한다.

>gitlab-runner 하나에 여러 프로젝트를 등록할 수 있다.

>확인 : List all configured runners
>$ gitlab-runner list

>등록해제
>gitlab-runner unregister --url http://172.20.0.7:9000/ --token a5549aee3ea5eee36c39b4f70c2197

#### JDK 설치
>requires Java to compile
>[requires Java to compile](../../AA/JDK/install.n.setup.md)

#### Gradle 설치
>requires Gradle to build
>ref. 30.architecture/30.technical/30.solution/y2.gradle/10.install.n setup

## 9. Appendix

#### reference site

* GitLab Runner  
https://docs.gitlab.com/runner/

* Run GitLab Runner in a container  
https://docs.gitlab.com/runner/install/docker.html
