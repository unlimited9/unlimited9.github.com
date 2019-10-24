# How to Kubernetes Deployment

## packaging

#### packaging : get/create docker image (dockerizing)
ref. [create docker image and container](../docker/create.image.n.container.md)

## create kubernetes resouces

#### caution
* gitrepo volume 사용 시 kubernetes node에 git이 설치되어 있지 않다면 아래와 같은 에러 발생
  > `error : message`  
  Warning  FailedMount  28s (x10 over 4m38s)  kubelet, tf-01     MountVolume.SetUp failed for volume "app-git-repository" : failed to exec 'git clone -- http://172.20.0.7:9000/enliple/mobon/platform/gateway.git .': : executable file not found in $PATH  
  > `solution`  
  $ yum install git

* private docker image 사용 시 아래와 같은 처리 필요
  > `error : message`  
  Failed to pull image "mobon/java.app.ext:latest": rpc error: code = Unknown desc = Error response from daemon: pull access denied for mobon/java.app.ext, repository does not exist or may require 'docker login
  > `solution`  
  $ yum install git
#### ConfigMap
ref. [kubernetes ConfigMap and Secret](configMap.n.secret.md)

#### Deployment
`create kubernetes resource file : Deployment`  
$ vi /apps/kubernetes/resources/mobon.gateway.aggregator.deployment.yaml 
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mobon-platform-gateway-aggregator-deployment
  labels:
    app: mobon-platform-gateway-aggregator
spec:
  selector:
    matchLabels:
      app: mobon-platform-gateway-aggregator
  replicas: 3
#  minReadySeconds: 5
  template:
    metadata:
      labels:
        app: mobon-platform-gateway-aggregator
    spec:
      containers:
        - name: mobon-platform-gateway-aggregator
          image: docker-registry.mobon.net:5000/mobon/java.app.ext:latest
          imagePullPolicy: Always
          volumeMounts:
            - name: app-git-repository
              mountPath: /repository/git/mobon.platform
            - name: app-scouter-repository
              mountPath: /apps/scouter/2.7.0/agent.host/conf/scouter.conf
              subPath: scouter-host.conf
            - name: app-scouter-repository
              mountPath: /apps/scouter/2.7.0/agent.java/conf/scouter.conf
              subPath: scouter-java.conf
          ports:
            - name: http
              containerPort: 8080
#          livenessProbe:
#            httpGet:
#              path: /default/dspt/status/health
#              port: 10254
#              scheme: HTTP
#            initialDelaySeconds: 10
#            periodSeconds: 10
#            timeoutSeconds: 1
#            successThreshold: 1
#            failureThreshold: 3
          readinessProbe:
            httpGet:
              path: /default/dspt/status/health
              port: 8080
              scheme: HTTP
#            initialDelaySeconds: 10
#            periodSeconds: 10
#            timeoutSeconds: 1
#            successThreshold: 1
#            failureThreshold: 3
          env:
            - name: _POD_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
            - name: SCOUTER_DIR
              value: "/apps/scouter/2.7.0"
            - name: JAVA_OPTS
              value: "-javaagent:$(SCOUTER_DIR)/agent.java/scouter.agent.jar"
            - name: JAVA_OPTS
              value: "$(JAVA_OPTS) -Dscouter.config=$(SCOUTER_DIR)/agent.java/conf/scouter.conf"
            - name: JAVA_OPTS
              value: "$(JAVA_OPTS) -Dobj_name=product-$(_POD_IP)"
          command: ["/bin/bash", "-c"]
          args:
            - source /etc/profile;
              echo "########################################";
              echo "start scouter agent host...";
              echo "########################################";
              cd /apps/scouter/2.7.0/agent.host;
              ./host.sh;
              echo "########################################";
              echo "gradle springboot application build and packaging...";
              echo "########################################";
              mkdir # /pgms/mobon.platform.gateway;
              cp -R /repository/git/mobon.platform/gateway.git/aggregation.service /pgms/mobon.platform.gateway/aggregation.service;
              gradle --build-file /pgms/mobon.platform.gateway/aggregation.service/build.gradle :framework.boot.application:bootJar;
              echo "########################################";
              echo "run springboot application with scouter agent.java...";
              echo "########################################";
              java $JAVA_OPTS -jar /pgms/mobon.platform.gateway/aggregation.service/framework.boot.application/build/libs/framework.boot.application-1.0.jar;
#              tail -f /dev/null;
#          lifecycle:
#            postStart:
#              exec:
#                command: ["/bin/bash", "-c", "echo $JAVA_OPTS"]
#            preStop:
#              exec:
#                command: ["/bin/sh","-c",""]
      initContainers:
        - name: git-sync
          image: k8s.gcr.io/git-sync:v3.1.2
          imagePullPolicy: Always
          volumeMounts:
            - name: app-git-repository
              mountPath: /repository/git/mobon.platform
#            - name: git-secret
#              mountPath: /etc/git-secret
          env:
            - name: GIT_SYNC_REPO
              value: http://172.20.0.7:9000/enliple/mobon/platform/gateway.git
            - name: GIT_SYNC_BRANCH
              value: master
            - name: GIT_SYNC_ROOT
              value: /repository/git/mobon.platform
            - name: GIT_SYNC_DEST
              value: ""
            - name: GIT_SYNC_PERMISSIONS
              value: "0777"
            - name: GIT_SYNC_ONE_TIME
              value: "true"
#            - name: GIT_SYNC_SSH
#              value: "true"
            - name: GIT_SYNC_USERNAME
              value: git_id
            - name: GIT_SYNC_PASSWORD
              value: git_passwd
          securityContext:
            runAsUser: 0
      imagePullSecrets:
       - name: regcred
      volumes:
      - name: app-git-repository
        emptyDir: {}
#      - name: git-secret
#        secret:
#          defaultMode: 256
#          secretName: git-cred # your-ssh-key
      - name: app-scouter-repository
        configMap:
          name: scouter-config
          defaultMode: 420
```
>          - name: GIT_SYNC_USERNAME
>            valueFrom:
>              secretKeyRef:
>                name: git-creds
>                key: username
>          - name: GIT_SYNC_PASSWORD
>            valueFrom:
>              secretKeyRef:
>                name: git-creds
>                key: password

>`livenessProbe, readinessProbe options`  
initialDelaySecond : 컨테이너가 실행되고 첫 Probe체크를 할 때까지의 delay (default: 0)  
periodSeconds : 체크 주기 (default 10초, min 1초)  
timeoutSeconds : Probe 체크 timeout (default: 1초, min 1초)  
successThreshold : Probe 체크 실패 시 Threshold만큼 다시 체크에 성공해야 success로 판단 (default: 1)  
failureThreshold : Probe 체크 실패 시 Threshold만큼 재 시도 하고 체크 포기 (default: 3)  
readness의 경우 트래픽 대상에서 제외, liveness의 경우 컨테이너 재시작  

`deployment 생성`  
$ kubectl create -f /apps/kubernetes/resources/mobon.gateway.aggregator.deployment.yaml

> secret을 사용할 경우 아래와 같이 secret을 생성하고 위 주석 제거 및 GIT_SYNC_USERNAME, GIT_SYNC_PASSWORD 주석 처리해서 사용
>#### Using SSH with git-sync
<details>
<summary>more ...</summary>
<div markdown="1">
  
>https://github.com/kubernetes/git-sync/blob/master/docs/ssh.md
>
>$ ssh-keyscan github.com > /tmp/known_hosts
>
>$ kubectl create secret generic git-creds \
>    --from-file=ssh=$HOME/.ssh/git_rsa \
>    --from-file=known_hosts=/tmp/known_hosts
>
>$ kubectl get secret git-creds

</div>
</details>

`pod 확인`  
$ kubectl get pod
```
NAME                                                           READY   STATUS    RESTARTS   AGE
mobon-platform-gateway-aggregator-deployment-844b4b7bc-4zs79   1/1     Running   0          5m36s
mobon-platform-gateway-aggregator-deployment-844b4b7bc-d5gjx   1/1     Running   0          5m36s
mobon-platform-gateway-aggregator-deployment-844b4b7bc-q8xn8   1/1     Running   0          5m36s
```
$ kubectl describe pod mobon-platform-gateway-aggregator-deployment-844b4b7bc-4zs79

`pod 접속 확인`  
$ kubectl exec -it mobon-platform-gateway-aggregator-deployment-844b4b7bc-4zs79 /bin/bash

#### Updating a Deployment
: 객체 설정 업데이트

`rollout status`  
$ kubectl rollout status deploy mobon-platform-gateway-aggregator-deployment
> $ kubectl rollout status deployment.v1.apps/mobon-platform-gateway-aggregator-deployment
```
deployment "mobon-platform-gateway-aggregator-deployment" successfully rolled out
```

`deployment configuration update`  
: 리소스의 설정 정보를 kubectl 설치 서버 에디터로 수정  
$ kubectl edit deploy mobon-platform-gateway-aggregator-deployment
> $ kubectl edit deployment.v1.apps/mobon-platform-gateway-aggregator-deployment

`kubectl replace`  
: 설정 파일 수정 및 생성 후 설정을 업데이트  
> $ kubectl replace -f /apps/kubernetes/resources/mobon.gateway.aggregator.deployment.yaml

`kubectl patch`  
: 파일 업데이트 없이 기존 리소스의 설정 정보 중 여러개의 필드를 수정  
> $ kubectl patch deployment mobon-platform-gateway-aggregator-deployment --patch 'pec:\n template:\n  spec:\n containers:\n - name: mobon-platform-gateway-aggregator-deployment\n image: mobon-platform-gateway-aggregator=docker-registry.mobon.net:5000/mobon/java.app.ext:v2'

`rollout history`  
: 기존에 배포된 이력 확인  
$ kubectl rollout history deploy/mobon-platform-gateway-aggregator-deployment  
> $ kubectl rollout history deployment.v1.apps/mobon-platform-gateway-aggregator-deployment

`rollout undo`  
: 이전 버전으로 롤백  
$ kubectl rollout undo deploy mobon-platform-gateway-aggregator-deployment  
> $ kubectl rollout undo deployment.v1.apps/mobon-platform-gateway-aggregator-deployment  
> $ kubectl rollout undo deployment.v1.apps/mobon-platform-gateway-aggregator-deployment --to-revision=2

`rollout restart`  
$ kubectl rollout restart deploy/mobon-platform-gateway-aggregator-deployment

`deployment scale(pod 수) 변경`  
$ kubectl scale deploy mobon-platform-gateway-aggregator-deployment --replicas=10
> $ kubectl scale deployment.v1.apps/mobon-platform-gateway-aggregator-deployment --replicas=10

`deployment image 변경`  
> $ kubectl set image deployment mobon-platform-gateway-aggregator-deployment mobon-platform-gateway-aggregator=docker-registry.mobon.net:5000/mobon/java.app.ext:v2

>#### ReplicationController
<details>
<summary>more ...</summary>
<div markdown="1">

>`create kubernetes resource file : ReplicationController`  
>$ vi /apps/kubernetes/resources/mobon.gateway.rc.yaml 
>```
>apiVersion: v1
>kind: ReplicationController
>metadata:
>  name: mobon-platform-gateway-aggregator-rc
>spec:
>  replicas: 3
>  selector:
>    app: mobon-platform-gateway-aggregator
>  template:
>    metadata:
>      name: mobon-platform-gateway-aggregator-pod
>      labels:
>        app: mobon-platform-gateway-aggregator
>    spec:
>      containers:
>      - name: mobon-platform-gateway-aggregator
>        image: docker-registry.mobon.net:5000/mobon/java.app.ext:latest
>        imagePullPolicy: Always
>        volumeMounts:
>          - name: app-git-repository
>            mountPath: /repository/git/mobon.platform.gateway.git
>            readOnly: true
>        ports:
>          - containerPort: 8080
>        command: ["/bin/bash", "-c"]
>        args:
>          - source /etc/profile;
>            mkdir /pgms/mobon.platform.gateway;
>            cp -R /repository/git/mobon.platform.gateway.git /pgms/mobon.platform.gateway/sources;
>            gradle --build-file /pgms/mobon.platform.gateway/sources/aggregation.service/build.gradle :framework.boot.application:bootRun;
>#            tail -f /dev/null;
>      imagePullSecrets:
>        - name: regcred
>      volumes:
>      - name: app-git-repository
>        gitRepo:
>          repository: http://mobon_admin:mobonproject2019!@172.20.0.7:9000/enliple/mobon/platform/gateway.git
>          revision: master
>          directory: .
>```
>
>> gitRepo volume deprecated
>
>`rc 생성`  
>$ kubectl create -f /apps/kubernetes/resources/mobon.gateway.rc.yaml

`pod 확인`  
$ kubectl get pod
```
NAME                                                           READY   STATUS    RESTARTS   AGE
mobon-platform-gateway-aggregator-deployment-844b4b7bc-4zs79   1/1     Running   0          5m36s
mobon-platform-gateway-aggregator-deployment-844b4b7bc-d5gjx   1/1     Running   0          5m36s
mobon-platform-gateway-aggregator-deployment-844b4b7bc-q8xn8   1/1     Running   0          5m36s
```
$ kubectl describe pod mobon-platform-gateway-aggregator-deployment-844b4b7bc-4zs79

`pod 접속 확인`  
$ kubectl exec -it mobon-platform-gateway-aggregator-deployment-844b4b7bc-4zs79 /bin/bash

`rc scale(pod 수) 변경`  
$ kubectl scale --replicas=6 rc/mobon.platform.gateway.aggregator.rc 

</div>
</details>

#### Service

`Get external IP address of Kubernetes nodes`  
$ kubectl get nodes --selector=kubernetes.io/role!=master -o jsonpath={.items[*].status.addresses[?\(@.type==\"InternalIP\"\)].address}

`create kubernetes resource file : Service`  
$ vi /apps/kubernetes/resources/mobon.gateway.aggregator.svc.yaml 
```
apiVersion: v1
kind: Service
metadata:
  name: mobon-platform-gateway-aggregator-svc
spec:
  selector:
    app: mobon-platform-gateway-aggregator
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: 8080
  type: LoadBalancer
  externalIPs:
    - 172.20.0.31
```

`svc 생성`  
$ kubectl create -f /apps/kubernetes/resources/mobon.gateway.aggregator.svc.yaml

`svc 확인`  
$ kubectl get svc

`접속 확인`  
$ curl ip:80

## check kubernetes resources

#### debugging
$ kubectl get rc  
$ kubectl get pods  
$ kubectl describe pod mobon-platform-gateway-aggregator-deployment-844b4b7bc-4zs79  

$ kubectl logs -f -lapp=mobon-platform-gateway-aggregator  
>```
>error: you are attempting to follow 12 log streams, but maximum allowed concurency is 5, use --max-log-requests to increase the limit
>```
>$ kubectl logs -f -lapp=mobon-platform-gateway-aggregator --max-log-requests 20  

>$ kubectl logs mobon-platform-gateway-aggregator-deployment-844b4b7bc-4zs79  
>$ kubectl logs mobon-platform-gateway-aggregator-deployment-844b4b7bc-4zs79 -c git-sync

$ journalctl -u kubelet

## delete kubernetes resources

#### service delete
$ kubectl delete svc --all

#### deployment delete
$ kubectl delete deployment mobon-platform-gateway-aggregator-deployment

#### replication controller delete
$ kubectl delete rc --all

#### pod delete
$ kubectl delete pod --all

> controller rc나 deployment를 지우지 않고 pod를 삭제하면 controller가 pod를 다시 생성한다.
> controller를 삭제하면 관련 pod가 같이 삭제된다.

## Considerations

#### Caution 



## 9. Appendix

#### reference site

* 쿠버네티스 #6 - 실제 서비스 배포해보기  
https://bcho.tistory.com/1261?category=731548

* 컨테이너를 위한 커맨드와 인자 정의하기  
https://kubernetes.io/ko/docs/tasks/inject-data-application/define-command-argument-container/

* Deployments
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment

+ Guide to Spring Cloud Kubernetes  
https://www.baeldung.com/spring-cloud-kubernetes

+ Configuration management: a Spring Boot use-case with Kubernetes  
https://www.exoscale.com/syslog/configuration-management-kubernetes-spring-boot/

- git-sync  
https://ddii.dev/kubernetes/git-sync/#

- Using SSH with git-sync  
https://github.com/kubernetes/git-sync/blob/master/docs/ssh.md

- 쿠버네티스 컨트롤러 : 디플로이먼트(Deployments)  
https://arisu1000.tistory.com/27833

- kubernetes를 이용한 서비스 무중단 배포  
https://tech.kakao.com/2018/12/24/kubernetes-deploy/

- Kubernetes 실습 스크립트  
https://github.com/TheOpenCloudEngine/uEngine-cloud/wiki/Kubernetes-%EC%8B%A4%EC%8A%B5-%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8
