## mobon.service.conversion

#### mobon.service.conversion
$ vi application/mobon.service.conversion.yaml
```
apiVersion: v1
kind: Namespace
metadata:
  name: mobon
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mobon-service-conversion
  namespace: mobon
  labels:
    app: mobon-service-conversion
spec:
  selector:
    matchLabels:
      app: mobon-service-conversion
  replicas: 3
  template:
    metadata:
      labels:
        app: mobon-service-conversion
    spec:
#      nodeSelector:
#        env: conversion
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
        - name: mobon-service-conversion
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
              containerPort: 7080
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
              path: /conversion/healthCheck
              port: 7080
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
              value: "$(JAVA_OPTS) -Dobj_name=conversion-$(_POD_IP)"
            - name: JAVA_OPTS
              value: "$(JAVA_OPTS) -Xms512m -Xmx512m -XX:MaxMetaspaceSize=512m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8"
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
              mkdir -p /pgms/mobon/conversion.service;
              cp -R /repository/git/mobon/conversion.git/* /pgms/mobon/conversion.service;
              echo 'org.gradle.jvmargs='$JAVA_OPTS >> /pgms/mobon/conversion.service/gradle.properties;
              gradle --build-file /pgms/mobon/conversion.service/build.gradle :mobon.service.conversion:bootRun -Dspring.profiles.active=prod -PjvmArgs="$(JAVA_OPTS)";
#              gradle --build-file /pgms/mobon/conversion.service/build.gradle :mobon.service.conversion:bootJar;
#              echo "########################################";
#              echo "run springboot application with scouter agent.java...";
#              echo "########################################";
#              java $JAVA_OPTS -Dspring.profiles.active=prod -jar /pgms/mobon/conversion.service/mobon.service.conversion/build/libs/mobon.service.conversion-1.0.jar;
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
              value: http://172.20.0.7:9000/enliple/mobon/service/conversion.git
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
  name: mobon-service-conversion-svc
  namespace: mobon
  labels:
    app: mobon-service-conversion
spec:
  selector:
    app: mobon-service-conversion
  type: ClusterIP
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: 7080

```
