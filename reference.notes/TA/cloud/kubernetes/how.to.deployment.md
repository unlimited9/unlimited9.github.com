# How to Kubernetes Deployment

## service packaging deploy

#### packaging : get/create docker image (dockerizing)
ref. [create docker image and container](../docker/create.image.n.container.md)

#### ReplicationController
`create kubernetes object file : ReplicationController`  
$ vi /apps/kubernetes/resources/mobon.gateway.rc.yaml 
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: mobon.platform.gateway.aggregator.rc
spec:
  replicas: 3
  selector:
    app: mobon.platform.gateway.aggregator
  template:
    metadata:
      name: mobon.platform.gateway.aggregator.pod
      labels:
        app: mobon.platform.gateway.aggregator
    spec:
      containers:
      - name: mobon.platform.gateway.aggregator
        image: mobon/java.app.env:latest
        imagePullPolicy: Always
        volumeMounts:
          - name: app.repository
            mountPath: /pgms/mobon.platform.gateway/repository/git
            readOnly: true
        ports:
          - containerPort: 8080
        command: ["/bin/bash", "-c"]
        args:
          - ls -al /pgms/mobon.platform.gateway/repository/git
          - gradle --build-file /pgms/mobon.platform.gateway/repository/git/aggregation.service/build.gradle :framework.boot.application:build
          - gradle --build-file /pgms/mobon.platform.gateway/repository/git/aggregation.service/build.gradle :framework.boot.application:build
    volumes:
     - name: app.repository
       gitRepo:
            repository: http://172.20.0.7:9000/enliple/mobon/platform/gateway.git
            revision: master
            directory: .
```

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
