# kubernetes logging with fluentd, elasticsearch

#### create fluentd kubernetest container

[install elasticsearch](../elasticsearch/install.n.setup.0.md)  
[install kibana](../elasticsearch/install.n.setup.2.kibana.md)

#### create fluentd kubernetest container
$ vi kubernetes/resources/solution/mobon.fluentd.yaml
```
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: fluentd
  namespace: kube-system

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: fluentd
  namespace: kube-system
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - namespaces
  verbs:
  - get
  - list
  - watch

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: fluentd
roleRef:
  kind: ClusterRole
  name: fluentd
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: fluentd
  namespace: kube-system

---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
    version: v1
spec:
  selector:
    matchLabels:
      k8s-app: fluentd-logging
      version: v1
  template:
    metadata:
      labels:
        k8s-app: fluentd-logging
        version: v1
    spec:
      serviceAccount: fluentd
      serviceAccountName: fluentd
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset:v1-debian-elasticsearch
        env:
          - name:  FLUENT_ELASTICSEARCH_HOST
            value: "10.251.0.188"
          - name:  FLUENT_ELASTICSEARCH_PORT
            value: "6530"
          - name: FLUENT_ELASTICSEARCH_SCHEME
            value: "http"
          - name: FLUENT_UID
            value: "0"
          - name: FLUENTD_SYSTEMD_CONF
            value: "disable"
#          # Option to configure elasticsearch plugin with self signed certs
#          # ================================================================
#          - name: FLUENT_ELASTICSEARCH_SSL_VERIFY
#            value: "true"
#          # Option to configure elasticsearch plugin with tls
#          # ================================================================
#          - name: FLUENT_ELASTICSEARCH_SSL_VERSION
#            value: "TLSv1_2"
#          # X-Pack Authentication
#          # =====================
#          - name: FLUENT_ELASTICSEARCH_USER
#            value: "elastic"
#          - name: FLUENT_ELASTICSEARCH_PASSWORD
#            value: "changeme"
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          #path: /var/lib/docker/containers
          path: /data/docker/containers
```
$ kubectl create -f kubernetes/resources/solution/mobon.fluentd.yaml  

$ kubectl logs -f -lk8s-app=fluentd-logging --max-log-requests 100 -n kube-system  

#### create kibana index patterns
Kibana > Management > Index patterns > `Create index pattern`  

Step 1 of 2: Define index pattern  
Index pattern : logstash-*  

Step 2 of 2: Configure settings  
Time Filter field name  
select : @timestamp  

`Create index pattern`  

#### search data
Kibana > Discover  

## 9. Appendix

#### reference site

* Fluentd Daemonset for Kubernetes  
https://github.com/fluent/fluentd-kubernetes-daemonset

