## install ECK(elastic cloud on kubernetes)

#### Deploy ECK in your Kubernetes cluster  
kubectl apply -f https://download.elastic.co/downloads/eck/1.2.0/all-in-one.yaml  

kubectl -n elastic-system logs -f statefulset.apps/elastic-operator  

#### Deploy an Elasticsearch cluster  
```
cat <<EOF | kubectl apply -f -
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: quickstart
spec:
  version: 7.8.1
  nodeSets:
  - name: default
    count: 1
    config:
      node.master: true
      node.data: true
      node.ingest: true
      node.store.allow_mmap: false
EOF
```
<details>
<summary>create yaml file</summary>
<div markdown="1">

```
cat <<EOF > elasticsearch.yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: quickstart
spec:
  version: 7.8.1
  nodeSets:
  - name: default
    count: 1
    config:
      node.master: true
      node.data: true
      node.ingest: true
      node.store.allow_mmap: false
EOF
```
kubectl apply -f elasticsearch.yaml  

</div>
</details>

>linkerd inject :  
>‼ no supported objects found  
>elasticsearch "quickstart" skipped  

`Monitor cluster health and creation progressedit`  
kubectl get elasticsearch  
kubectl get pods --selector='elasticsearch.k8s.elastic.co/cluster-name=quickstart'  
kubectl logs -f quickstart-es-default-0  

`Request Elasticsearch accessedit`  
kubectl get service quickstart-es-http  

PASSWORD=$(kubectl get secret quickstart-es-elastic-user -o go-template='{{.data.elastic | base64decode}}')  
>7G5tHn3J781Qc8x4d26rotkK  

curl -u "elastic:$PASSWORD" -k "https://quickstart-es-http:9200"  

kubectl port-forward service/quickstart-es-http 9200  

curl -u "elastic:$PASSWORD" -k "https://localhost:9200"  

#### Deploy a Kibana instance  
```
cat <<EOF | kubectl apply -f -
apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
  name: quickstart
spec:
  version: 7.8.1
  count: 1
  elasticsearchRef:
    name: quickstart
EOF
```
<details>
<summary>create yaml file</summary>
<div markdown="1">

```
cat <<EOF > kibana.yaml
apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
  name: quickstart
spec:
  version: 7.8.1
  count: 1
  elasticsearchRef:
    name: quickstart
EOF
```
kubectl apply -f kibana.yaml  

</div>
</details>

>linkerd inject :  
>‼ no supported objects found  
>kibana "quickstart" skipped  

kubectl get kibana  

kubectl get pod --selector='kibana.k8s.elastic.co/name=quickstart'  

kubectl get service quickstart-kb-http  

kubectl port-forward service/quickstart-kb-http 5601  

kubectl get secret quickstart-es-elastic-user -o=jsonpath='{.data.elastic}' | base64 --decode; echo  
>7G5tHn3J781Qc8x4d26rotkK  

#### Upgrade your deployment  
```
cat <<EOF | kubectl apply -f -
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: quickstart
spec:
  version: 7.8.1
  nodeSets:
  - name: default
    count: 3
    config:
      node.master: true
      node.data: true
      node.ingest: true
      node.store.allow_mmap: false
EOF
```

#### Use persistent storage  


#### Check out the samples  

## appendix

#### reference.site

* Elastic Cloud on Kubernetes [1.2] » Quickstart    
https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-quickstart.html  
