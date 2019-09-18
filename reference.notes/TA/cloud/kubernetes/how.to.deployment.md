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
  Failed to pull image "mobon/java.app.env:latest": rpc error: code = Unknown desc = Error response from daemon: pull access denied for mobon/java.app.env, repository does not exist or may require 'docker login
  > `solution`  
  $ yum install git

#### ReplicationController
`create kubernetes object file : ReplicationController`  
$ vi /apps/kubernetes/resources/mobon.gateway.rc.yaml 
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: mobon-platform-gateway-aggregator-rc
spec:
  replicas: 3
  selector:
    app: mobon-platform-gateway-aggregator
  template:
    metadata:
      name: mobon-platform-gateway-aggregator-pod
      labels:
        app: mobon-platform-gateway-aggregator
    spec:
      containers:
      - name: mobon-platform-gateway-aggregator
        image: mobon/java.app.env:latest
        imagePullPolicy: Always
        volumeMounts:
          - name: app-git-repository
            mountPath: /pgms/mobon.platform.gateway/repository/git
            readOnly: true
        ports:
          - containerPort: 8080
        command: ["/bin/bash", "-c"]
        args:
          - ls -al /pgms/mobon.platform.gateway/repository/git
          - gradle --build-file /pgms/mobon.platform.gateway/repository/git/aggregation.service/build.gradle :framework.boot.application:build
      volumes:
      - name: app-git-repository
        gitRepo:
          repository: http://172.20.0.7:9000/enliple/mobon/platform/gateway.git
          revision: master
          directory: .
```

`rc 생성`  
$ kubectl create -f /apps/kubernetes/resources/mobon.gateway.rc.yaml

`pod 확인`  
$ kubectl get pod
```
NAME                                         READY   STATUS              RESTARTS   AGE
mobon-platform-gateway-aggregator-rc-fxkwx   0/1     ContainerCreating   0          8s
mobon-platform-gateway-aggregator-rc-wdxpp   0/1     ContainerCreating   0          8s
mobon-platform-gateway-aggregator-rc-x8qdz   0/1     ContainerCreating   0          8s
```
$ kubectl describe pod mobon-platform-gateway-aggregator-rc-fxkwx

`pod 접속 확인`  
$ kubectl exec -it mobon-platform-gateway-aggregator-rc-fxkwx /bin/bash

`rc scale(pod 수) 변경`  
$ kubectl scale --replicas=6 rc/mobon.platform.gateway.aggregator.rc 

#### Service
`create kubernetes object file : Service`  
$ vi /apps/kubernetes/resources/mobon.gateway.svc.yaml 
```
apiVersion: v1
kind: Service
metadata:
  name: mobon.gateway.svc
spec:
  selector:
    app: mobon.gateway
  ports:
    - port: 80
      protocol: TCP
      targetPort: 8080
  type: LoadBalancer
```

`svc 생성`  
$ kubectl create -f /apps/kubernetes/resources/mobon.gateway.svc.yaml

`svc 확인`  
$ kubectl get svc

`접속 확인`  
$ curl ip:80

## delete kubernetes resources

#### service delete
$ kubectl delete svc --all

#### replication controller delete
$ kubectl delete rc --all

#### pod delete
$ kubectl delete pod --all

> rc를 지우지 않고 pod를 삭제하면 rc가 pod를 다시 생성한다.

## 9. Appendix

#### reference site

* 쿠버네티스 #6 - 실제 서비스 배포해보기  
https://bcho.tistory.com/1261?category=731548
