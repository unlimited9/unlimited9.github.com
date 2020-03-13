# How to Kubernetes Deployment(StatefulSet)

## create kubernetes resouces

#### StatefulSet
`create kubernetes resource file : StatefulSet`  
$ vi /apps/kubernetes/resources/mobon.platform.mongodb.statefulset.yaml 
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
#  minReadySeconds: 5
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
              mkdir # /pgms/mobon.platform.gateway;
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
              value: "true"
#            - name: GIT_SYNC_SSH
#              value: "true"
            - name: GIT_SYNC_USERNAME
              value: git_id
            - name: GIT_SYNC_PASSWORD
              value: git_passwd
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


```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mobon-platform-audience-mongodb-statefulset
  labels:
    app: mobon-platform-audience-mongodb
spec:
  selector:
    matchLabels:
      app: mobon-platform-audience-mongodb
  serviceName: "mobon-platform-audience-mongodb"
#  podManagementPolicy: Parallel
  replicas: 3
#  minReadySeconds: 5
  template:
    metadata:
      labels:
        app: mobon-platform-audience-mongodb
    spec:
      terminationGracePeriodSeconds: 10
      containers:
        - name: mobon-platform-audience-mongodb
          image: k8s.gcr.io/nginx-slim:0.8
          ports:
            - containerPort: 80
              name: web
          volumeMounts:
            - name: www
              mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
    - metadata:
      name: www
      spec:
        accessModes: [ "ReadWriteOnce" ]
        storageClassName: "standard"
        resources:
          requests:
            storage: 1Gi

```
  selector:
    matchLabels:
      app: mobon-platform-gateway-aggregator
  replicas: 3
#  minReadySeconds: 5
  template:
