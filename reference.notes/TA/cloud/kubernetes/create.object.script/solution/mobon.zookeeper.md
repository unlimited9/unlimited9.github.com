## mobon.zookeeper

#### add node label
$ kubectl label nodes mpk-cluster-06 zookeeper=  
$ kubectl label nodes mpk-cluster-07 zookeeper=  
$ kubectl label nodes mpk-cluster-08 zookeeper=  

#### mobon.zookeeper
$ vi solution/mobon.zookeeper.yaml
```
apiVersion: v1
kind: Namespace
metadata:
  name: mobon
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mobon-data-zookeeper
  namespace: mobon
  labels:
    app: mobon-data-zookeeper
spec:
  serviceName: mobon-data-zookeeper-svc
  selector:
    matchLabels:
      app: mobon-data-zookeeper
#  podManagementPolicy: Parallel
  replicas: 3
#  minReadySeconds: 5
  template:
    metadata:
      labels:
        app: mobon-data-zookeeper
    spec:
#      nodeSelector:
#        env: product
#        servicetype: data
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: env
                operator: In
                values:
                  - product
              - key: servicetype
                operator: In
                values:
                  - data
              - key: zookeeper
                operator: Exists
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                    - mobon-data-zookeeper
              topologyKey: kubernetes.io/hostname
      containers:
        - name: mobon-data-zookeeper
          image: docker-registry.mobon.net:5000/mobon/zookeeper.3:latest
          imagePullPolicy: Always
          env:
            - name: NODE_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
          command: ["/bin/bash", "-c"]
          args:
            - |
              set -ex
              source /etc/profile
              # pod ordinal index.
              [[ `hostname` =~ -([0-9]+)$ ]] || exit 1
              ordinal=${BASH_REMATCH[1]}
              # ordinal=${HOSTNAME##*-}  
              echo $((1 + $ordinal)) > /data/zookeeper/myid
              /apps/zookeeper/3.5.6/bin/zkServer.sh start-foreground
#              /apps/zookeeper/3.5.6/bin/zkServer.sh print-cmd
#              tail -f /dev/null
          volumeMounts:
#            - name: app-zookeeper-config
#              mountPath: /app/zookeeper/3.5.6/config
            - name: app-zookeeper-data
              mountPath: /data/zookeeper
          resources:
            limits:
              cpu: 1000m
            requests:
              cpu: 100m
          ports:
            - name: client
              containerPort: 7214
            - name: follow-leader
              containerPort: 7224
            - name: leader-election
              containerPort: 7234
      imagePullSecrets:
        - name: docker-reg-cred
      volumes:
#        - name: app-elasticsearch-config
#          configMap:
#            name: elasticsearch-config 
#            defaultMode: 420
        - name : app-zookeeper-data
          hostPath:
            path: /data/zookeeper
            type: Directory
#  volumeClaimTemplates:
#    - metadata:
#        name: storage
#        annotations:
#          volume.beta.kubernetes.io/storage-class: "fast"
#      spec:
#        accessModes: [ "ReadWriteOnce" ]
#        storageClassName: fast
#        resources:
#          requests:
#            storage: 10Gi
---
apiVersion: v1
kind: Service
metadata:
  name: mobon-data-zookeeper-svc
  namespace: mobon
  labels:
    app: mobon-data-zookeeper
spec:
  selector:
    app: mobon-data-zookeeper
  ports:
    - name: client
      port: 7214
      targetPort: 7214
    - name: follow-leader
      port: 7224
      targetPort: 7224
    - name: leader-election
      port: 7234
      targetPort: 7234
  clusterIP: None

```

## 9. Appendix

#### reference site

* 분산 시스템 코디네이터 ZooKeeper 실행하기  
https://kubernetes.io/ko/docs/tutorials/stateful-application/zookeeper/
