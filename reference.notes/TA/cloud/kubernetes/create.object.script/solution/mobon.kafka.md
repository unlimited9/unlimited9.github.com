## mobon.kafka

#### add node label
$ kubectl label nodes mpk-cluster-06 kafka=  
$ kubectl label nodes mpk-cluster-07 kafka=  
$ kubectl label nodes mpk-cluster-08 kafka=  

#### mobon.kafka
$ vi solution/mobon.kafka.yaml
```
apiVersion: v1
kind: Namespace
metadata:
  name: mobon
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mobon-data-kafka
  namespace: mobon
  labels:
    app: mobon-data-kafka
spec:
  serviceName: mobon-data-kafka-svc
  selector:
    matchLabels:
      app: mobon-data-kafka
#  podManagementPolicy: Parallel
  replicas: 3
#  minReadySeconds: 5
  template:
    metadata:
      labels:
        app: mobon-data-kafka
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
              - key: kafka
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
                    - mobon-data-kafka
              topologyKey: kubernetes.io/hostname
      containers:
        - name: mobon-data-kafka
          image: docker-registry.mobon.net:5000/mobon/kafka.2:latest
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
              
              sed -i -e 's/^broker.id=0$/broker.id=$ordinal/' /apps/kafka/2.13-2.4.0/config/server.properties
              sed -i -e 's/^#listeners=PLAINTEXT:\/\/:9092$/listeners=PLAINTEXT:\/\/:7642/' /apps/kafka/2.13-2.4.0/config/server.properties
              sed -i -e 's/^#advertised.listeners=PLAINTEXT:\/\/your.host.name:9092$/advertised.listeners=PLAINTEXT:\/\/${HOSTNAME}:7642/' /apps/kafka/2.13-2.4.0/config/server.properties
              sed -i -e 's/^zookeeper.connect=localhost:2181$/zookeeper.connect=10.10.10.25:7214,10.10.10.26:7214,10.10.10.27:7214/' /apps/kafka/2.13-2.4.0/config/server.properties

              /apps/kafka/2.13-2.4.0/bin/kafka-server-start.sh /apps/kafka/2.13-2.4.0/config/server.properties
#              tail -f /dev/null
          volumeMounts:
#            - name: app-kafka-config
#              mountPath: /app/kafka/2.13-2.4.0/config
          resources:
            limits:
              cpu: 1000m
            requests:
              cpu: 100m
          ports:
            - name: client
              containerPort: 7642
      imagePullSecrets:
        - name: docker-reg-cred
      volumes:
#        - name: app-kafka-config
#          configMap:
#            name: kafka-config 
#            defaultMode: 420
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
  name: mobon-data-kafka-svc
  namespace: mobon
  labels:
    app: mobon-data-kafka
spec:
  selector:
    app: mobon-data-kafka
  ports:
    - name: client
      port: 7642
      targetPort: 7642
  clusterIP: None

```

## 9. Appendix

#### reference site

* 분산 시스템 코디네이터 kafka 실행하기  
https://kubernetes.io/ko/docs/tutorials/stateful-application/kafka/
