# How to Kubernetes Deployment

## packaging

#### packaging : get/create docker image (dockerizing)
ref. [create docker image and container](../docker/create.image.n.container.md)

## create kubernetes resouces

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
```
Name:           mobon-platform-gateway-aggregator-rc-fxkwx
Namespace:      default
Priority:       0
Node:           tf-01/172.20.0.31
Start Time:     Wed, 18 Sep 2019 10:38:48 +0900
Labels:         app=mobon-platform-gateway-aggregator
Annotations:    <none>
Status:         Pending
IP:             
Controlled By:  ReplicationController/mobon-platform-gateway-aggregator-rc
Containers:
  mobon-platform-gateway-aggregator:
    Container ID:  
    Image:         mobon/java.app.env:latest
    Image ID:      
    Port:          8080/TCP
    Host Port:     0/TCP
    Command:
      /bin/bash
      -c
    Args:
      ls -al /pgms/mobon.platform.gateway/repository/git
      gradle --build-file /pgms/mobon.platform.gateway/repository/git/aggregation.service/build.gradle :framework.boot.application:build
    State:          Waiting
      Reason:       ContainerCreating
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /pgms/mobon.platform.gateway/repository/git from app-git-repository (ro)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-lmp75 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  app-git-repository:
    Type:        GitRepo (a volume that is pulled from git when the pod is created)
    Repository:  http://172.20.0.7:9000/enliple/mobon/platform/gateway.git
    Revision:    master
  default-token-lmp75:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-lmp75
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason       Age                   From               Message
  ----     ------       ----                  ----               -------
  Normal   Scheduled    4m38s                 default-scheduler  Successfully assigned default/mobon-platform-gateway-aggregator-rc-fxkwx to tf-01
  Warning  FailedMount  28s (x10 over 4m38s)  kubelet, tf-01     MountVolume.SetUp failed for volume "app-git-repository" : failed to exec 'git clone -- http://172.20.0.7:9000/enliple/mobon/platform/gateway.git .': : executable file not found in $PATH
  Warning  FailedMount  20s (x2 over 2m35s)   kubelet, tf-01     Unable to mount volumes for pod "mobon-platform-gateway-aggregator-rc-fxkwx_default(3565ddbf-28a6-4a04-91da-636d42d24964)": timeout expired waiting for volumes to attach or mount for pod "default"/"mobon-platform-gateway-aggregator-rc-fxkwx". list of unmounted volumes=[app-git-repository]. list of unattached volumes=[app-git-repository default-token-lmp75]
```

> `error : message`  
  Warning  FailedMount  28s (x10 over 4m38s)  kubelet, tf-01     MountVolume.SetUp failed for volume "app-git-repository" : failed to exec 'git clone -- http://172.20.0.7:9000/enliple/mobon/platform/gateway.git .': : executable file not found in $PATH
> `install git`
$ yum install git

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
