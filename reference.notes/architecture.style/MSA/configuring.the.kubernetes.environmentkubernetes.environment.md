# Configuring the kubernetes environment

#### install kubernetes/docker  
`on linux`  
  - [Kubernetes(Docker orchestration and clustering) 설치](/reference.notes/architecture.solution/kubernetes/install.n.setup.md)  
  - [Kubernetes masster node HA, multi-master 구성](/reference.notes/architecture.solution/kubernetes/master.node.cluster.ha.md)  
  - [Docker(independent container platform) 설치](/reference.notes/architecture.solution/docker/install.n.setup.md)  

`on windows`  

#### install linkerd  
`on kubernetes cluster`  
  - [Linkerd(service mesh for kubernetes) 설치](/reference.notes/architecture.solution/linkerd/install.and.setup.md)  
  - [Linkerd(service mesh)에 서비스 추가/주입](/reference.notes/architecture.solution/linkerd/add.service.to.linkerd.service.mesh.md)  

`linkerd dashboard`  
_on linux_  
linkerd dashboard &  
_on windows_  
Start-process -NoNewWindow linkerd dashboard

#### install ECK(elastic cloud on kubernetes)
`on kubernetes cluster`  
  - [ECK 설치](/reference.notes/architecture.solution/elasticsearch/install.ECK_elastic.cloud.on.kubernetes_.md)  

`Elasticsearch cluster`  
_on linux_  
kubectl port-forward service/quickstart-es-http 9200 &  
_on windows_  
Start-process -NoNewWindow kubectl port-forward service/quickstart-es-http 9200

`Kibana`  
_on linux_  
kubectl port-forward service/quickstart-kb-http 5601  
_on windows_  
Start-process -NoNewWindow kubectl kubectl port-forward service/quickstart-kb-http 5601  

