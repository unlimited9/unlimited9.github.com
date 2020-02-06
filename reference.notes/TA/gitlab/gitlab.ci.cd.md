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
  - restart

before_script:
  # set java environment path
  - source /etc/profile.d/jdk.12.sh
  #- pwd

cache:
  paths:
    - gateway.service.aggregation/build

build-to-aggregation:
  stage: build
  environment:
    name: aggregation build server
  only:
    - master
  script:
    - gradle -v
    - gradle --build-file build.gradle :gateway.service.aggregation:bootJar;
#    - gradle --build-file gateway.service.aggregation/build.gradle :build
  tags:
    - build

test-to-aggregation:
  stage: test
  environment:
    name: aggregation test server
  only:
    - master
  script:
    - echo 'test to aggregation'
  tags:
    - test

deploy-to-aggregation:
  stage: deploy
  environment:
    name: aggregation deploy server
  only:
    - master
  script:
    # source backup over ssh
    - ssh -i ~/.ssh/app2 -t -p 7722 app@10.251.0.194 "mv /pgms/mobon.platform/gateway.service.aggregation*.jar /pgms/mobon.platform/backup/gateway.service.aggregation_$(date +"%Y%m%d_%H%M%S").jar || :"
    # jar(package) file transfer to deploy server
    - scp -i ~/.ssh/app2 -P 7722 gateway.service.aggregation/build/libs/gateway.service.aggregation*.jar app@10.251.0.194:/pgms/mobon.platform
  after_script:
    - echo 'deploy completed'
#  when: manual
  tags:
    - deploy

restart-to-aggregation-development:
  stage: restart
  environment:
    name: aggregation development service restart
  only:
    - master
  when : manual
  script:
    # development source backup over ssh
    - ssh -i ~/.ssh/app -t -p 7722 app@172.20.0.103 "mv /pgms/mobon.platform/gateway.service.aggregation*.jar /pgms/mobon.platform/backup/gateway.service.aggregation_$(date +"%Y%m%d_%H%M%S").jar || :"
    # jar(package) file transfer to devlepment server
    - scp -i ~/.ssh/app -P 7722 gateway.service.aggregation/build/libs/gateway.service.aggregation*.jar app@172.20.0.103:/pgms/mobon.platform
    # restart service
    - ssh -i ~/.ssh/app -t -p 7722 app@172.20.0.103 "ps -ef | grep gateway.service.aggregation | grep -v grep | awk '{ print $2 }' | xargs kill -9  || :" 
    - ssh -i ~/.ssh/app -T -p 7722 app@172.20.0.103 "find /pgms/mobon.platform -maxdepth 1 -type f -name 'gateway.service.aggregation*.jar' | sort | tail -1 | xargs -I MPGS_JAR nohup /apps/jdk/12.0.2/bin/java -Dspring.profiles.active=dev -jar MPGS_JAR 1> /dev/null 2>&1 &"
  after_script:
    - echo 'restart completed'
  tags:
    - development.restart

```

#### java library build and deploy
$ vi .gitlab-ci.yml
```yaml
# Define stages in a pipeline
stages:
  - build
  - test
  - deploy_utils
  - deploy_da_utils
  - deploy_tracker_common

before_script:
  # set java environment path
  - source /etc/profile.d/java12.sh
  - pwd

# build to development server
build-to-development:
  stage: build
  environment:
    name: development deploy server
  only:
    - master
  script:
    - gradle -v
    - gradle --build-file mobon.utils/build.gradle :build
    - gradle --build-file mobon.da.utils/build.gradle :build
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
    - gradle -v
    - gradle --build-file mobon.utils/build.gradle :mobon.utils:test
    - gradle --build-file mobon.da.utils/build.gradle :mobon.da:test
  tags:
    - development
    
# deploy to development server
deploy_utils-to-development:
  stage: deploy_utils
  environment:
    name: development deploy server
  only:
    - master
  when: manual
  script:
    - gradle -v
    - gradle --build-file mobon.utils/build.gradle :mobon.utils:uploadArchives
  tags:
    - development

deploy_da_utils-to-development:
  stage: deploy_da_utils
  environment:
    name: development deploy server
  only:
    - master
  when: manual
  script:
    - gradle -v
    - gradle --build-file mobon.da.utils/build.gradle :mobon.da:uploadArchives
  tags:
    - development

deploy_tracker_common-to-development:
  stage: deploy_tracker_common
  environment:
    name: development deploy server
  only:
    - master
  when: manual
  script:
    - gradle -v
    - gradle --build-file mobon.tracker.common/build.gradle :mobon.tracker.common:uploadArchives
  tags:
    - development
#deploy-to-development:
#  stage: deploy
#  environment:
#    name: development deploy server
#  only:
#    - master
#  script:
#    - pwd
#    # upload to nexus
#    - curl --upload-file mobon.utils/build/libs/enliple-utils-0.1.0.jar -u admin:P@ssw0rd -v http://172.20.0.7:8081/repository/maven-releases/com/mobon/mobon.utils/0.1.0/mobon.utils-0.1.0.jar
#  after_script:
#    - echo 'deploy completed'
#  when: manual
#  tags:
#    - development
    
# upload to nexus

cache:
  paths:
    - mobon.utils/build
    - mobon.da.utils/build
    
```

>
>#### Upload Nexus
>$ curl --upload-file mobonUtils/mobon.ad.utils/build/libs/enliple-utils-0.1.0.jar -u admin:P@ssw0rd -v http://172.20.0.7:8081/repository/maven-releases/com/mobon/mobon.ad.utils/0.1.0/mobon.ad.utils-0.1.0.jar
>>$ curl -v -F r=releases -F hasPom=false -F e=jar -F g=com.mobon -F a=mobon.ad.utils -F v=0.1.0 -F p=jar -F file=@"./mobonUtils/mobon.ad.utils/build/libs/enliple-utils-0.1.0.jar" -u admin:P@ssword http://172.20.0.7:8081/repositories/maven-releases/com/mobon/mobon.ad.utils/0.1.0/enliple-utils-0.1.0.jar
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
deploy runner for gateway.service.aggregation

Please enter the gitlab-ci tags for this runner (comma separated):
build, test, deploy, development.restart

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
>[requires Java to compile](../../AA/JDK/install.n.setup.md)

#### Gradle 설치
>[requires Gradle to build](../../AA/gradle/install.n.setup.md)

## 9. Appendix

#### reference site

* GitLab Runner  
https://docs.gitlab.com/runner/

* Run GitLab Runner in a container  
https://docs.gitlab.com/runner/install/docker.html
