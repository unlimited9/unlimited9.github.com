## mobon.gateway.scouter

#### mobon.platform.scouter.config
$ vi mobon.platform.scouter.config.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: scouter-config
  namespace: default
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

## mobon.gateway.telegraf

#### mobon.platform.telegraf.config
$ vi mobon.platform.telegraf.config.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: telegraf-config
  namespace: default
data:
  telegraf.conf: |
    [global_tags]
      role = "ap-server"
      scouter_obj_type_prefix = "mobon.platform"
    
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

## mobon.gateway.api.common

#### mobon.gateway.api.common.deployment
$ vi mobon.gateway.api.common.deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mobon-platform-gateway-api-common-deployment
  labels:
    app: mobon-platform-gateway-api-common
spec:
  selector:
    matchLabels:
      app: mobon-platform-gateway-api-common
  replicas: 3
  template:
    metadata:
      labels:
        app: mobon-platform-gateway-api-common
    spec:
      containers:
        - name: mobon-platform-gateway-api-common
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
              mkdir -p /pgms/mobon.platform/gateway/api.git;
              cp -R /repository/git/mobon.platform/gateway/api.git/* /pgms/mobon.platform/gateway/api.git;
              gradle --build-file /pgms/mobon.platform/gateway/api.git/build.gradle :gateway.api.common:bootJar;
              echo "########################################";
              echo "run springboot application with scouter agent.java...";
              echo "########################################";
              java $JAVA_OPTS -jar /pgms/mobon.platform/gateway/api.git/gateway.api.common/build/libs/gateway.api.common-1.0.jar;
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
              value: http://172.20.0.7:9000/enliple/mobon/platform/gateway/api.git
            - name: GIT_SYNC_BRANCH
              value: master
            - name: GIT_SYNC_ROOT
              value: /repository/git/mobon.platform/gateway
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

#### mobon.gateway.api.common.svc
$ vi mobon.gateway.api.common.svc.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: mobon-platform-gateway-api-common-svc
spec:
  selector:
    app: mobon-platform-gateway-api-common
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: 8080
  type: LoadBalancer
  externalIPs:
  - 10.251.0.181
  - 119.205.238.81

```

## mobon.gateway.service.aggregation

#### mobon.gateway.service.aggregation.deployment
$ vi mobon.gateway.service.aggregation.deployment.yaml 
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
              path: /tracker/status/health
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
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mobon-platform-gateway-service-aggregation-deployment
  labels:
    app: mobon-platform-gateway-service-aggregation
spec:
  selector:
    matchLabels:
      app: mobon-platform-gateway-service-aggregation
  replicas: 3
  template:
    metadata:
      labels:
        app: mobon-platform-gateway-service-aggregation
    spec:
      containers:
        - name: mobon-platform-gateway-service-aggregation
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
              path: /default/dspt/status/health
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
              mkdir -p /pgms/mobon.platform/gateway/service.git;
              cp -R /repository/git/mobon.platform/gateway/service.git/* /pgms/mobon.platform/gateway/service.git;
              gradle --build-file /pgms/mobon.platform/gateway/service.git/build.gradle :gateway.service.aggregation:bootJar;
              echo "########################################";
              echo "run springboot application with scouter agent.java...";
              echo "########################################";
              java $JAVA_OPTS -jar /pgms/mobon.platform/gateway/service.git/gateway.service.aggregation/build/libs/gateway.service.aggregation-1.0.jar;
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
              value: http://172.20.0.7:9000/enliple/mobon/platform/gateway/service.git
            - name: GIT_SYNC_BRANCH
              value: master
            - name: GIT_SYNC_ROOT
              value: /repository/git/mobon.platform/gateway
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

#### mobon.gateway.service.aggregation.svc
$ vi mobon.gateway.service.aggregation.svc.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: mobon-platform-gateway-service-aggregation-svc
spec:
  selector:
    app: mobon-platform-gateway-service-aggregation
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: 8080
  type: LoadBalancer
  externalIPs:
  - 10.251.0.182
  - 119.205.238.82

```

## mobon.service.product

#### mobon.service.product.deployment
$ vi mobon.service.product.deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mobon-platform-service-product-deployment
  labels:
    app: mobon-platform-service-product
spec:
  selector:
    matchLabels:
      app: mobon-platform-service-product
  replicas: 3
  template:
    metadata:
      labels:
        app: mobon-platform-service-product
    spec:
      containers:
        - name: mobon-platform-service-product
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
              mkdir -p /pgms/mobon.platform/product.service;
              cp -R /repository/git/mobon.platform/product.git/* /pgms/mobon.platform/product.service;
              gradle --build-file /pgms/mobon.platform/product.service/build.gradle :framework.springboot:bootJar;
              echo "########################################";
              echo "run springboot application with scouter agent.java...";
              echo "########################################";
              java $JAVA_OPTS -jar /pgms/mobon.platform/product.service/framework.springboot/build/libs/framework.springboot-1.0.jar;
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
              value: http://172.20.0.7:9000/enliple/mobon/service/product.git
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

#### mobon.service.product.svc
$ vi mobon.service.product.svc.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: mobon-platform-service-product-svc
spec:
  selector:
    app: mobon-platform-service-product
  type: ClusterIP
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: 8080

```

> ping mobon-platform-service-product-svc.default.svc.mobon.stage  
> curl http://mobon-platform-service-product-svc/default/dspt/product/conf/developer/KR
