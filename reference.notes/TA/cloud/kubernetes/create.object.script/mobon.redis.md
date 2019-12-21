## mobon.redis

#### mobon.redis
$ mobon.redis.yaml
```
apiVersion: v1
kind: Namespace
metadata:
  name: mobon
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mobon-data-redis
  namespace: mobon
  labels:
    app: mobon-data-redis
spec:
  serviceName: mobon-data-redis
  selector:
    matchLabels:
      app: mobon-data-redis
#  podManagementPolicy: Parallel
  replicas: 6
#  minReadySeconds: 5
  template:
    metadata:
      labels:
        app: mobon-data-redis
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
              - key: redis
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
                    - mobon-data-redis
                - key: role
                  operator: In
                  values:
                    - master
              topologyKey: kubernetes.io/hostname
      containers:
        - name: mobon-data-redis
          image: docker-registry.mobon.net:5000/mobon/redis.5:latest
          imagePullPolicy: Always
          command: ["/bin/bash", "-c", "/apps/redis/5.0.7/bin/redis-server /apps/redis/5.0.7/config/redis.conf"]
          volumeMounts:
#            - name: app-redis-config
#              mountPath: /app/redis/5.0.7/config
            - name: app-redis-data
              mountPath: /data/redis
          resources:
            limits:
              cpu: 1000m
            requests:
              cpu: 100m
          ports:
            - name: client
              containerPort: 2816
            - name: gossip
              containerPort: 12816
      initContainers:
        - name: overcommit-memory
          image: busybox
          command: ["sysctl", "-w", "vm.overcommit_memory=1"]
          securityContext:
            privileged: true
        - name: memory-swappiness
          image: busybox
          command: ["sysctl", "-w", "vm.swappiness=1"]
          securityContext:
            privileged: true
        - name: tcp-backlog
          image: busybox
          command: ["sysctl", "-w", "net.core.somaxconn=65535"]
          securityContext:
            privileged: true
        - name: transparent-huge-pages
          image: busybox
          command: ["sysctl", "-w", "echo never > /sys/kernel/mm/transparent_hugepage/enabled"]
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
#        - name: app-redis-config
#          configMap:
#            name: redis-config 
#            defaultMode: 420
        - name : app-redis-data
          hostPath:
            path: /data/redis
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
  name: mobon-data-redis-svc
  namespace: mobon
  labels:
    app: mobon-data-redis
    role: client
spec:
  selector:
    app: mobon-data-redis
  ports:
    - name: client
      port: 2816
      targetPort: 2816
    - name: gossip
      port: 12816
      targetPort: 12816
  type: LoadBalancer
  externalIPs:
  - 10.251.0.187
```




