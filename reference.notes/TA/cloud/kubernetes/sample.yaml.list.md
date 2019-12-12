## namespace

#### mobon.namespace
$ vi mobon.namespace.yaml
```
apiVersion: v1
kind: Namespace
metadata:
  name: mobon
```

> 이후 계속 중복 정의한 Namespace 관련 부분은 계속 있다고 에러가 나지만 그냥 냅뒀다

## configuration

#### mobon.scouter.config
$ vi mobon.scouter.config.yaml
```
apiVersion: v1
kind: Namespace
metadata:
  name: mobon
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: scouter-config
  namespace: mobon
data:
  scouter-host.conf: |
    net_collector_ip=172.20.0.108
  scouter-java.conf: |
    net_collector_ip=172.20.108
    trace_http_client_ip_header_key=X-Forwarded-For
    profile_http_parameter_enabled=true
    profile_http_header_enabled=true
    xlog_lower_bound_time_ms=100

```

#### mobon.telegraf.config
$ vi mobon.telegraf.config.yaml
```
apiVersion: v1
kind: Namespace
metadata:
  name: mobon
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: telegraf-config
  namespace: mobon
data:
  telegraf.conf: |
    [global_tags]
      role = "ap-server"
      scouter_obj_type_prefix = "mobon"
    
    [agent]
      interval = "5s"
      round_interval = true
      metric_batch_size = 1000
      metric_buffer_limit = 10000
      collection_jitter = "0s"
      flush_interval = "5s"
      flush_jitter = "0s"
      precision = ""
      debug = false
      quiet = false
      logfile = ""
      hostname = "ap-server"
      omit_hostname = false
    
    #[[outputs.influxdb]]
    #  urls = ["http://10.0.0.127:8086"]
    #  database = "telegraf"
    #  precision = "s"
    #  retention_policy = "default"
    #  write_consistency = "any"
    #  timeout = "5s"
    #  username = "telegraf"
    #  password = "telegraf"
    
    [[outputs.http]]
      url = "http://172.20.0.108:6100/telegraf/metric"
      timeout = "5s"
      method = "POST"
      data_format = "influx"
    
    [[inputs.statsd]]
      # Address and port to host UDP listener on
      service_address = “:8125”
      # Delete gauges every interval (default=false)
      delete_gauges = true
      # Delete counters every interval (default=false)
      delete_counters = true
      # Delete sets every interval (default=false)
      delete_sets = false
      # Delete timings & histograms every interval (default=true)
      delete_timings = true
      # Percentiles to calculate for timing & histogram stats
      percentiles = [90]
      # convert measurement names, “.” to “_” and “-” to “__”
      convert_names = false
      templates = [
        "* measurement.field"
      ]
      # Number of UDP messages allowed to queue up, once filled,
      # the statsd server will start dropping packets
      allowed_pending_messages = 10000
      # Number of timing/histogram values to track per-measurement in the
      # calculation of percentiles. Raising this limit increases the accuracy
      # of percentiles but also increases the memory usage and cpu time.
      percentile_limit = 1000
      # UDP packet size for the server to listen for. This will depend on the size
      # of the packets that the client is sending, which is usually 1500 bytes.
      udp_packet_size = 1500
    
    [[inputs.cpu]]
      percpu = true
      totalcpu = true
      collect_cpu_time = false
    
    [[inputs.disk]]
      ignore_fs = ["tmpfs", "devtmpfs"]
    
    [[inputs.kernel]]
    
    [[inputs.mem]]
    
    [[inputs.net]]
    
    [[inputs.netstat]]
    
    [[inputs.system]]
    
    [[inputs.diskio]]
    
    [[inputs.processes]]
    
```

## application

#### mobon.gateway.api.common
$ vi mobon.gateway.api.common.yaml
```
apiVersion: v1
kind: Namespace
metadata:
  name: mobon
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mobon-gateway-api-common-deployment
  namespace: mobon
  labels:
    app: mobon-gateway-api-common
spec:
  selector:
    matchLabels:
      app: mobon-gateway-api-common
  replicas: 3
  template:
    metadata:
      labels:
        app: mobon-gateway-api-common
    spec:
#      nodeSelector:
#        env: product
#        servicetype: apps
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
                - apps
      containers:
        - name: mobon-gateway-api-common
          image: docker-registry.mobon.net:5000/mobon/java.app.ext:latest
          imagePullPolicy: Always
          volumeMounts:
            - name: app-git-repository
              mountPath: /repository/git/mobon
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
              path: /api/status/health
#              path: /default/dspt/status/health
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
              mkdir -p /pgms/mobon/gateway/api.git;
              cp -R /repository/git/mobon/gateway/api.git/* /pgms/mobon/gateway/api.git;
              gradle --build-file /pgms/mobon/gateway/api.git/build.gradle :gateway.api.common:bootJar;
              echo "########################################";
              echo "run springboot application with scouter agent.java...";
              echo "########################################";
              java $JAVA_OPTS -jar /pgms/mobon/gateway/api.git/gateway.api.common/build/libs/gateway.api.common-1.0.jar;
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
              mountPath: /repository/git/mobon
#            - name: git-secret
#              mountPath: /etc/git-secret
          env:
            - name: GIT_SYNC_REPO
              value: http://172.20.0.7:9000/enliple/mobon/platform/gateway/api.git
            - name: GIT_SYNC_BRANCH
              value: master
            - name: GIT_SYNC_ROOT
              value: /repository/git/mobon/gateway
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
       - name: docker-reg-cred
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
---
apiVersion: v1
kind: Service
metadata:
  name: mobon-gateway-api-common-svc
  namespace: mobon
  labels:
    app: mobon-gateway-api-common
spec:
  selector:
    app: mobon-gateway-api-common
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: 8080
  type: LoadBalancer
  externalIPs:
  - 10.251.0.183

```

## mobon.gateway.service.aggregation

#### mobon.gateway.service.aggregation
$ vi mobon.gateway.service.aggregation.yaml 
```
apiVersion: v1
kind: Namespace
metadata:
  name: mobon
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mobon-gateway-service-aggregation-deployment
  namespace: mobon
  labels:
    app: mobon-gateway-service-aggregation
spec:
  selector:
    matchLabels:
      app: mobon-gateway-service-aggregation
  replicas: 3
  template:
    metadata:
      labels:
        app: mobon-gateway-service-aggregation
    spec:
#      nodeSelector:
#        env: product
#        servicetype: apps
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
                - apps
      containers:
        - name: mobon-gateway-service-aggregation
          image: docker-registry.mobon.net:5000/mobon/java.app.ext:latest
          imagePullPolicy: Always
          volumeMounts:
            - name: app-git-repository
              mountPath: /repository/git/mobon
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
              path: /tk/status/health
#              path: /default/dspt/status/health
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
              mkdir -p /pgms/mobon/gateway/service.git;
              cp -R /repository/git/mobon/gateway/service.git/* /pgms/mobon/gateway/service.git;
              gradle --build-file /pgms/mobon/gateway/service.git/build.gradle :gateway.service.aggregation:bootJar;
              echo "########################################";
              echo "run springboot application with scouter agent.java...";
              echo "########################################";
              java $JAVA_OPTS -jar /pgms/mobon/gateway/service.git/gateway.service.aggregation/build/libs/gateway.service.aggregation-1.0.jar;
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
              mountPath: /repository/git/mobon
#            - name: git-secret
#              mountPath: /etc/git-secret
          env:
            - name: GIT_SYNC_REPO
              value: http://172.20.0.7:9000/enliple/mobon/platform/gateway/service.git
            - name: GIT_SYNC_BRANCH
              value: master
            - name: GIT_SYNC_ROOT
              value: /repository/git/mobon/gateway
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
       - name: docker-reg-cred
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
---
apiVersion: v1
kind: Service
metadata:
  name: mobon-gateway-service-aggregation-svc
  namespace: mobon
  labels:
    app: mobon-gateway-service-aggregation
spec:
  selector:
    app: mobon-gateway-service-aggregation
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: 8080
  type: LoadBalancer
  externalIPs:
  - 10.251.0.184

```

## mobon.service.product

#### mobon.service.product
$ vi mobon.service.product.yaml
```
apiVersion: v1
kind: Namespace
metadata:
  name: mobon
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mobon-service-product-deployment
  namespace: mobon
  labels:
    app: mobon-service-product
spec:
  selector:
    matchLabels:
      app: mobon-service-product
  replicas: 3
  template:
    metadata:
      labels:
        app: mobon-service-product
    spec:
#      nodeSelector:
#        env: product
#        servicetype: apps
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
                - apps
      containers:
        - name: mobon-service-product
          image: docker-registry.mobon.net:5000/mobon/java.app.ext:latest
          imagePullPolicy: Always
          volumeMounts:
            - name: app-git-repository
              mountPath: /repository/git/mobon
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
              path: /default/snoop.jsp
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
              mkdir -p /pgms/mobon/product.service;
              cp -R /repository/git/mobon/product.git/* /pgms/mobon/product.service;
              gradle --build-file /pgms/mobon/product.service/build.gradle :mobon.service.product:bootJar;
              echo "########################################";
              echo "run springboot application with scouter agent.java...";
              echo "########################################";
              java $JAVA_OPTS -jar /pgms/mobon/product.service/mobon.service.product/build/libs/mobon.service.product-1.0.jar;
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
              mountPath: /repository/git/mobon
#            - name: git-secret
#              mountPath: /etc/git-secret
          env:
            - name: GIT_SYNC_REPO
              value: http://172.20.0.7:9000/enliple/mobon/service/product.git
            - name: GIT_SYNC_BRANCH
              value: master
            - name: GIT_SYNC_ROOT
              value: /repository/git/mobon
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
       - name: docker-reg-cred
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
---
apiVersion: v1
kind: Service
metadata:
  name: mobon-service-product-svc
  namespace: mobon
  labels:
    app: mobon-service-product
spec:
  selector:
    app: mobon-service-product
  type: ClusterIP
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: 8080

```

> ping mobon-service-product-svc.default.svc.mobon.stage  
> curl http://mobon-service-product-svc/default/dspt/product/conf/developer/KR

## mobon.elasticsearch

#### mobon.elasticsearch.master
$ mobon.elasticsearch.master.yaml
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
          command: ["/bin/bash", "-c", /apps/elasticsearch/elasticsearch start; tail -f /dev/null;]
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
$ mobon.elasticsearch.data.yaml
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
          command: ["/bin/bash", "-c", /apps/elasticsearch/elasticsearch start; tail -f /dev/null;]
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
$ mobon.elasticsearch.client.yaml
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
          command: ["/bin/bash", "-c", /apps/elasticsearch/elasticsearch start; tail -f /dev/null;]
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





