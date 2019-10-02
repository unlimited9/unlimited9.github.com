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

#### Deployment
`create kubernetes resource file : Deployment`  
$ vi /apps/kubernetes/resources/mobon.gateway.deployment.yaml 
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
            - containerPort: 8080
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
              mkdir /pgms/mobon.platform.gateway;
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
              value: mobon_admin
            - name: GIT_SYNC_PASSWORD
              value: mobonproject2019!
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

`deployment 생성`  
$ kubectl create -f /apps/kubernetes/resources/mobon.gateway.deployment.yaml

> secret을 사용할 경우 아래와 같이 secret을 생성하고 위 주석 제거 및 GIT_SYNC_USERNAME, GIT_SYNC_PASSWORD 주석 처리해서 사용

#### Using SSH with git-sync
<details>
<summary>more ...</summary>
<div markdown="1">
  
https://github.com/kubernetes/git-sync/blob/master/docs/ssh.md

$ ssh-keyscan github.com > /tmp/known_hosts

$ kubectl create secret generic git-creds \
    --from-file=ssh=$HOME/.ssh/git_rsa \
    --from-file=known_hosts=/tmp/known_hosts

$ kubectl get secret git-creds

</div>
</details>


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

</div>
<details>


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

#### Service
`create kubernetes resource file : Service`  
$ vi /apps/kubernetes/resources/mobon.gateway.svc.yaml 
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
$ kubectl create -f /apps/kubernetes/resources/mobon.gateway.svc.yaml

`svc 확인`  
$ kubectl get svc

`접속 확인`  
$ curl ip:80

## check kubernetes resources

#### debugging
$ kubectl get rc  
$ kubectl get pods  
$ kubectl describe pod mobon-platform-gateway-aggregator-deployment-844b4b7bc-4zs79  
$ kubectl logs mobon-platform-gateway-aggregator-deployment-844b4b7bc-4zs79  
$ kubectl logs mobon-platform-gateway-aggregator-deployment-844b4b7bc-4zs79 -c git-sync  
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

> rc를 지우지 않고 pod를 삭제하면 rc가 pod를 다시 생성한다.

## 9. Appendix

#### reference site

* 쿠버네티스 #6 - 실제 서비스 배포해보기  
https://bcho.tistory.com/1261?category=731548

* 컨테이너를 위한 커맨드와 인자 정의하기  
https://kubernetes.io/ko/docs/tasks/inject-data-application/define-command-argument-container/


+ Guide to Spring Cloud Kubernetes  
https://www.baeldung.com/spring-cloud-kubernetes

+ Configuration management: a Spring Boot use-case with Kubernetes  
https://www.exoscale.com/syslog/configuration-management-kubernetes-spring-boot/

- git-sync  
https://ddii.dev/kubernetes/git-sync/#

- Using SSH with git-sync  
https://github.com/kubernetes/git-sync/blob/master/docs/ssh.md
