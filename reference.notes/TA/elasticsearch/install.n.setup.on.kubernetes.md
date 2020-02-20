# Highly Available and Scalable Elasticsearch on Kubernetes

```
apiVersion: v1
kind: Namespace
metadata:
  name: mobon.2.0
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
  namespace: mobon.2.0
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
---
apiVersion: elasticsearch.k8s.elastic.co/v1alpha1
kind: Elasticsearch
metadata:
  name: mobon.elasticsearch
  namespace: mobon.2.0
  labels:
    component: elasticsearch
    role: master
spec:
  version: 7.1.0
  nodes:
  # 3 dedicated master nodes
  - nodeCount: 3
    config:
      node.master: true
      node.data: false
      node.ingest: false
      cluster.remote.connect: false
    podTemplate:
      spec:
        containers:
        - name: elasticsearch
          resources:
            limits:
              memory: 16Gi
              cpu: 2
        affinity:
          podAntiAffinity:
            preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchLabels:
                    elasticsearch.k8s.elastic.co/cluster-name: quickstart
                topologyKey: kubernetes.io/hostname
    volumeClaimTemplates:
    - metadata:
        name: elasticsearch-data
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 1Ti
        storageClassName: local-storage
  # 3 ingest-data nodes
  - nodeCount: 3
    config:
      node.master: false
      node.data: true
      node.ingest: true
    podTemplate:
      spec:
        affinity:
          podAntiAffinity:
            preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchLabels:
                    elasticsearch.k8s.elastic.co/cluster-name: quickstart
                topologyKey: kubernetes.io/hostname
    volumeClaimTemplates:
    - metadata:
        name: elasticsearch-data
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 1Ti
        storageClassName: local-storage
```






```
apiVersion: v1
kind: Namespace
metadata:
  name: elasticsearch
---



cat <<EOF | kubectl apply -f -
apiVersion: elasticsearch.k8s.elastic.co/v1alpha1
kind: Elasticsearch
metadata:
  name: quickstart
spec:
  version: 7.2.0
  nodes:
  - nodeCount: 1
    config:
      node.master: true
      node.data: true
      node.ingest: true
EOF
```



#### Deployment and Headless for Master Nodes.
```
apiVersion: v1
kind: Namespace
metadata:
  name: elasticsearch
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: es-master
  namespace: elasticsearch
  labels:
    component: elasticsearch
    role: master
spec:
  replicas: 3
  template:
    metadata:
      labels:
        component: elasticsearch
        role: master
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: role
                  operator: In
                  values:
                  - master
              topologyKey: kubernetes.io/hostname
      initContainers:
      - name: init-sysctl
        image: busybox:1.27.2
        command:
        - sysctl
        - -w
        - vm.max_map_count=262144
        securityContext:
          privileged: true
      containers:
      - name: es-master
        image: quay.io/pires/docker-elasticsearch-kubernetes:6.2.4
        env:
        - name: NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: CLUSTER_NAME
          value: my-es
        - name: NUMBER_OF_MASTERS
          value: "2"
        - name: NODE_MASTER
          value: "true"
        - name: NODE_INGEST
          value: "false"
        - name: NODE_DATA
          value: "false"
        - name: HTTP_ENABLE
          value: "false"
        - name: ES_JAVA_OPTS
          value: -Xms256m -Xmx256m
        - name: PROCESSORS
          valueFrom:
            resourceFieldRef:
              resource: limits.cpu
        resources:
          limits:
            cpu: 2
        ports:
        - containerPort: 9300
          name: transport
        volumeMounts:
        - name: storage
          mountPath: /data
      volumes:
          - emptyDir:
              medium: ""
            name: "storage"
---
apiVersion: v1
kind: Service
metadata:
  name: elasticsearch-discovery
  namespace: elasticsearch
  labels:
    component: elasticsearch
    role: master
spec:
  selector:
    component: elasticsearch
    role: master
  ports:
  - name: transport
    port: 9300
    protocol: TCP
  clusterIP: None
```
$ kubectl apply -f es-master.yml  
$ kubectl -n elasticsearch get all
$ kubectl -n elasticsearch logs -f po/es-master-594b58b86c-9jkj2 | grep ClusterApplierService

#### Stateful Set and Headless Service for Data Nodes
```
apiVersion: v1
kind: Namespace
metadata:
  name: elasticsearch
---
apiVersion: storage.k8s.io/v1beta1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
  fsType: xfs
allowVolumeExpansion: true
---
apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
  name: es-data
  namespace: elasticsearch
  labels:
    component: elasticsearch
    role: data
spec:
  serviceName: elasticsearch-data
  replicas: 3
  template:
    metadata:
      labels:
        component: elasticsearch
        role: data
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: role
                  operator: In
                  values:
                  - data
              topologyKey: kubernetes.io/hostname
      initContainers:
      - name: init-sysctl
        image: busybox:1.27.2
        command:
        - sysctl
        - -w
        - vm.max_map_count=262144
        securityContext:
          privileged: true
      containers:
      - name: es-data
        image: quay.io/pires/docker-elasticsearch-kubernetes:6.2.4
        env:
        - name: NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: CLUSTER_NAME
          value: my-es
        - name: NODE_MASTER
          value: "false"
        - name: NODE_INGEST
          value: "false"
        - name: HTTP_ENABLE
          value: "false"
        - name: ES_JAVA_OPTS
          value: -Xms256m -Xmx256m
        - name: PROCESSORS
          valueFrom:
            resourceFieldRef:
              resource: limits.cpu
        resources:
          limits:
            cpu: 2
        ports:
        - containerPort: 9300
          name: transport
        volumeMounts:
        - name: storage
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: storage
      annotations:
        volume.beta.kubernetes.io/storage-class: "fast"
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: fast
      resources:
        requests:
          storage: 10Gi
---
apiVersion: v1
kind: Service
metadata:
  name: elasticsearch-data
  namespace: elasticsearch
  labels:
    component: elasticsearch
    role: data
spec:
  ports:
  - port: 9300
    name: transport
  clusterIP: None
  selector:
    component: elasticsearch
    role: data
```
$ kubectl apply -f es-data.yml  
$ kubectl -n elasticsearch get pods -l role=data

#### Deployment and External Service for the Client Nodes
```
apiVersion: v1
kind: Namespace
metadata:
  name: elasticsearch
---
apiVersion: apps/v1Deployment and External Service for the Client Nodes
beta1
kind: Deployment
metadata:
  name: es-client
  namespace: elasticsearch
  labels:
    component: elasticsearch
    role: client
spec:
  replicas: 2
  template:
    metadata:
      labels:
        component: elasticsearch
        role: client
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: role
                  operator: In
                  values:
                  - client
              topologyKey: kubernetes.io/hostname
      initContainers:
      - name: init-sysctl
        image: busybox:1.27.2
        command:
        - sysctl
        - -w
        - vm.max_map_count=262144
        securityContext:
          privileged: true
      containers:
      - name: es-client
        image: quay.io/pires/docker-elasticsearch-kubernetes:6.2.4
        env:
        - name: NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: CLUSTER_NAME
          value: my-es
        - name: NODE_MASTER
          value: "false"
        - name: NODE_DATA
          value: "false"
        - name: HTTP_ENABLE
          value: "true"
        - name: ES_JAVA_OPTS
          value: -Xms256m -Xmx256m
        - name: NETWORK_HOST
          value: _site_,_lo_
        - name: PROCESSORS
          valueFrom:
            resourceFieldRef:
              resource: limits.cpu
        resources:
          limits:
            cpu: 1
        ports:
        - containerPort: 9200
          name: http
        - containerPort: 9300
          name: transport
        volumeMounts:
        - name: storage
          mountPath: /data
      volumes:
          - emptyDir:
              medium: ""
            name: storage
---
apiVersion: v1
kind: Service
metadata:
  name: elasticsearch
  namespace: elasticsearch
  annotations: 
    cloud.google.com/load-balancer-type: Internal
  labels:
    component: elasticsearch
    role: client
spec:
  selector:
    component: elasticsearch
    role: client
  ports:
  - name: http
    port: 9200
  type: LoadBalancer
```
$ kubectl apply -f es-client.yml   
$ kubectl -n elasticsearch get pods -l role=client

$ kubectl -n elasticsearch get all

#Check logs of es-master leader podroot
$ kubectl -n elasticsearch logs po/es-master-594b58b86c-bj7g7 | grep ClusterApplierService

## 9. Appendix

### reference site

#### Elastic Cloud on Kubernetes [master] » Advanced Elasticsearch node scheduling
https://www.elastic.co/guide/en/cloud-on-k8s/master/k8s-advanced-node-scheduling.html

#### Highly Available and Scalable Elasticsearch on Kubernetes
https://medium.com/faun/https-medium-com-thakur-vaibhav23-ha-es-k8s-7e655c1b7b61
