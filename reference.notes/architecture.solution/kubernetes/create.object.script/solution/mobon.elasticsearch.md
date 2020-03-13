## mobon.elasticsearch

#### mobon.elasticsearch.master
$ vi solution/mobon.elasticsearch.master.yaml
```
apiVersion: v1
kind: Namespace
metadata:
  name: mobon
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mobon-data-elasticsearch-master
  namespace: mobon
  labels:
    app: mobon-data-elasticsearch
    role: master
spec:
  serviceName: mobon-data-elasticsearch
  selector:
    matchLabels:
      app: mobon-data-elasticsearch
      role: master
#  podManagementPolicy: Parallel
  replicas: 3
#  minReadySeconds: 5
  template:
    metadata:
      labels:
        app: mobon-data-elasticsearch
        role: master
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
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                    - mobon-data-elasticsearch
                - key: role
                  operator: In
                  values:
                    - master
              topologyKey: kubernetes.io/hostname
      containers:
        - name: mobon-data-elasticsearch
          image: docker-registry.mobon.net:5000/mobon/elasticsearch.7:latest
          imagePullPolicy: Always
          env:
            - name: CLUSTER_NAME
              value: es-cluster
            - name: NODE_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: NODE_MASTER
              value: "true"
            - name: NETWORK_HOST
              value: "0.0.0.0"
            - name: DISCOVERY_SERVICE
              value: mobon-data-elasticsearch-svc
            - name: MASTER_NODES
              value: "mobon-data-elasticsearch-master-0,mobon-data-elasticsearch-master-1,mobon-data-elasticsearch-master-2"
            - name: ES_JAVA_OPTS
              value: "-Xms512m -Xmx512m"
            - name: NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
            - name: PROCESSORS
              valueFrom:
                resourceFieldRef:
                  resource: limits.cpu
            - name: MEMORY_LOCK
              value: "false"
          command: ["/bin/bash", "-c", "/apps/elasticsearch/7.5.0/bin/elasticsearch"]
          volumeMounts:
#            - name: app-elasticsearch-config
#              mountPath: /app/elasticsearch/7.5.0/config/elasticsearch.yml
            - name: app-elasticsearch-data
              mountPath: /data/elasticsearch
          resources:
            limits:
              cpu: 1000m
            requests:
              cpu: 100m
          ports:
            - name: http
              containerPort: 6530
            - name: transport
              containerPort: 6540
      initContainers:
        - name: increase-vm-max-map
          image: busybox
          command: ["sysctl", "-w", "vm.max_map_count=262144"]
          securityContext:
            privileged: true
        - name: increase-fd-ulimit
          image: busybox
          command: ["sh", "-c", "ulimit -n 65536"]
          securityContext:
            privileged: true
      imagePullSecrets:
        - name: docker-reg-cred
      volumes:
#        - name: app-elasticsearch-config
#          configMap:
#            name: elasticsearch-config 
#            defaultMode: 420
        - name: app-elasticsearch-data
          emptyDir: {}
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
  name: mobon-data-elasticsearch-svc
  namespace: mobon
  labels:
    app: mobon-data-elasticsearch
spec:
  selector:
    app: mobon-data-elasticsearch
  type: ClusterIP
  ports:
    - name: http
      port: 6540
      protocol: TCP
```

#### mobon.elasticsearch.data
$ vi solution/mobon.elasticsearch.data.yaml
```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mobon-data-elasticsearch-data
  namespace: mobon
  labels:
    app: mobon-data-elasticsearch
    role: data
spec:
  serviceName: mobon-data-elasticsearch
  selector:
    matchLabels:
      app: mobon-data-elasticsearch
      role: data
  replicas: 3
  template:
    metadata:
      labels:
        app: mobon-data-elasticsearch
        role: data
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
              - key: kubernetes.io/hostname
                operator: In
                values:
                  - mpk-cluster-09
                  - mpk-cluster-10
                  - mpk-cluster-11
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                    - mobon-data-elasticsearch
                - key: role
                  operator: In
                  values:
                    - data
              topologyKey: kubernetes.io/hostname
      containers:
        - name: mobon-data-elasticsearch
          image: docker-registry.mobon.net:5000/mobon/elasticsearch.7:latest
          imagePullPolicy: Always
          env:
            - name: CLUSTER_NAME
              value: es-cluster
            - name: NODE_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: NODE_DATA
              value: "true"
            - name: NETWORK_HOST
              value: "0.0.0.0"
            - name: DISCOVERY_SERVICE
              value: mobon-data-elasticsearch-svc
            - name: MASTER_NODES
              value: "mobon-data-elasticsearch-master-0,mobon-data-elasticsearch-master-1,mobon-data-elasticsearch-master-2"
            - name: ES_JAVA_OPTS
              value: "-Xms512m -Xmx512m"
            - name: NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
            - name: PROCESSORS
              valueFrom:
                resourceFieldRef:
                  resource: limits.cpu
            - name: MEMORY_LOCK
              value: "false"
          command: ["/bin/bash", "-c", "/apps/elasticsearch/7.5.0/bin/elasticsearch"]
          volumeMounts:
#            - name: app-elasticsearch-config
#              mountPath: /app/elasticsearch/7.5.0/config/elasticsearch.yml
            - name: app-elasticsearch-data
              mountPath: /data/elasticsearch
          resources:
            limits:
              cpu: 1000m
            requests:
              cpu: 100m
          ports:
            - name: http
              containerPort: 6530
            - name: transport
              containerPort: 6540
      initContainers:
        - name: increase-vm-max-map
          image: busybox
          command: ["sysctl", "-w", "vm.max_map_count=262144"]
          securityContext:
            privileged: true
        - name: increase-fd-ulimit
          image: busybox
          command: ["sh", "-c", "ulimit -n 65536"]
          securityContext:
            privileged: true
      imagePullSecrets:
        - name: docker-reg-cred
      volumes:
#        - name: app-elasticsearch-config
#          configMap:
#            name: elasticsearch-config 
#            defaultMode: 420
        - name : app-elasticsearch-data
          hostPath:
            path: /data/elasticsearch
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
```

#### mobon.elasticsearch.client
$ vi solution/mobon.elasticsearch.client.yaml
```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mobon-data-elasticsearch-client
  namespace: mobon
  labels:
    app: mobon-data-elasticsearch
    role: client
spec:
  serviceName: mobon-data-elasticsearch
  selector:
    matchLabels:
      app: mobon-data-elasticsearch
      role: client
#  podManagementPolicy: Parallel
  replicas: 3
#  minReadySeconds: 5
  template:
    metadata:
      labels:
        app: mobon-data-elasticsearch
        role: client
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
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                    - mobon-data-elasticsearch
                - key: role
                  operator: In
                  values:
                    - client
              topologyKey: kubernetes.io/hostname
      containers:
        - name: mobon-data-elasticsearch
          image: docker-registry.mobon.net:5000/mobon/elasticsearch.7:latest
          imagePullPolicy: Always
          env:
            - name: CLUSTER_NAME
              value: es-cluster
            - name: NODE_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: NETWORK_HOST
              value: "0.0.0.0"
            - name: DISCOVERY_SERVICE
              value: mobon-data-elasticsearch-svc
            - name: MASTER_NODES
              value: "mobon-data-elasticsearch-master-0,mobon-data-elasticsearch-master-1,mobon-data-elasticsearch-master-2"
            - name: ES_JAVA_OPTS
              value: "-Xms512m -Xmx512m"
            - name: NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
            - name: PROCESSORS
              valueFrom:
                resourceFieldRef:
                  resource: limits.cpu
            - name: MEMORY_LOCK
              value: "false"
          command: ["/bin/bash", "-c", "/apps/elasticsearch/7.5.0/bin/elasticsearch"]
          volumeMounts:
#            - name: app-elasticsearch-config
#              mountPath: /app/elasticsearch/7.5.0/config/elasticsearch.yml
            - name: app-elasticsearch-data
              mountPath: /data/elasticsearch
          resources:
            limits:
              cpu: 1000m
            requests:
              cpu: 100m
          ports:
            - name: http
              containerPort: 6530
            - name: transport
              containerPort: 6540
      initContainers:
        - name: increase-vm-max-map
          image: busybox
          command: ["sysctl", "-w", "vm.max_map_count=262144"]
          securityContext:
            privileged: true
        - name: increase-fd-ulimit
          image: busybox
          command: ["sh", "-c", "ulimit -n 65536"]
          securityContext:
            privileged: true
      imagePullSecrets:
        - name: docker-reg-cred
      volumes:
#        - name: app-elasticsearch-config
#          configMap:
#            name: elasticsearch-config 
#            defaultMode: 420
        - name: app-elasticsearch-data
          emptyDir: {}
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
  name: mobon-data-elasticsearch-client-svc
  namespace: mobon
  labels:
    app: mobon-data-elasticsearch
    role: client
spec:
  selector:
    app: mobon-data-elasticsearch
    role: client
  ports:
    - name: http
      port: 6530
      protocol: TCP
  type: LoadBalancer
  externalIPs:
  - 10.251.0.188
```

> memory locking requested for elasticsearch process but memory is not locked
>```
>...
>            - name: MEMORY_LOCK
>              value: "false"
>...
>```




