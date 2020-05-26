# kubernetes metrics-server

#### node / pod monitoring  

$ kubectl top pod
```
Error from server (NotFound): the server could not find the requested resource (get services http:heapster:)
```

#### Deployment
$ kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.6/components.yaml  

$ kubectl top node  
```
error: metrics not available yet
```

#### configuration
- --kubelet-preferred-address-types  
The priority of node address types used when determining an address for connecting to a particular node (default [Hostname,InternalDNS,InternalIP,ExternalDNS,ExternalIP])
- --kubelet-insecure-tls  
Do not verify the CA of serving certificates presented by Kubelets. For testing purposes only.
- --requestheader-client-ca-file  
Specify a root certificate bundle for verifying client certificates on incoming requests.


$ kubectl edit deployment metrics-server -n kube-system
```
...
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-insecure-tls
        - --kubelet-preferred-address-types=InternalIP
        image: k8s.gcr.io/metrics-server-amd64:v0.3.6
        imagePullPolicy: IfNotPresent
        name: metrics-server
...
```

#### the resource consumption : node
$ kubectl top node  
```
mpk-cluster-01   4163m        13%    14679Mi         23%       
mpk-cluster-02   1611m        5%     12617Mi         19%       
mpk-cluster-03   2441m        7%     17008Mi         26%       
mpk-cluster-04   4367m        13%    15047Mi         31%       
mpk-cluster-05   4425m        13%    14476Mi         22%       
mpk-cluster-06   792m         2%     14476Mi         22%       
mpk-cluster-07   1042m        3%     14848Mi         23%       
mpk-cluster-08   770m         2%     14686Mi         23%       
mpk-cluster-09   269m         1%     6935Mi          10%       
mpk-cluster-10   279m         1%     7050Mi          11%       
mpk-cluster-11   292m         1%     6838Mi          10%       
```

#### the resource consumption : pod
$ kubectl top pod
```
[app@MPK-Cluster-09 ~]$ kubectl top pod
NAME                                                  CPU(cores)   MEMORY(bytes)   
dmp-inflow-manager-5658467786-899fx                   366m         1010Mi          
dmp-inflow-manager-5658467786-8cxpq                   462m         982Mi           
dmp-inflow-manager-5658467786-9ffxs                   338m         1008Mi          
dmp-inflow-manager-5658467786-ff68d                   400m         1008Mi          
dmp-inflow-manager-5658467786-j6j2p                   361m         1020Mi          
mobon-batch-processor-5b99864958-rjdvl                11m          703Mi           
mobon-batch-processor-5b99864958-tq6nd                11m          696Mi           
mobon-batch-processor-5b99864958-trgj9                10m          629Mi           
mobon-data-elasticsearch-client-0                     279m         1824Mi          
mobon-data-elasticsearch-client-1                     27m          1723Mi          
mobon-data-elasticsearch-client-2                     88m          1736Mi          
mobon-data-elasticsearch-data-0                       441m         18670Mi         
mobon-data-elasticsearch-data-1                       440m         18533Mi         
mobon-data-elasticsearch-data-2                       478m         17197Mi         
mobon-data-elasticsearch-master-0                     10m          1005Mi          
mobon-data-elasticsearch-master-1                     6m           1035Mi          
mobon-data-elasticsearch-master-2                     6m           988Mi           
mobon-data-kafka-0                                    12m          1330Mi          
mobon-data-kafka-1                                    10m          1338Mi          
mobon-data-kafka-2                                    12m          1367Mi          
mobon-data-redis-master-0                             3m           16Mi            
mobon-data-redis-master-1                             4m           18Mi            
mobon-data-redis-master-2                             3m           18Mi            
mobon-data-redis-slave-0                              4m           69Mi            
mobon-data-redis-slave-1                              3m           62Mi            
mobon-data-redis-slave-2                              4m           63Mi            
mobon-data-zookeeper-0                                2m           359Mi           
mobon-data-zookeeper-1                                3m           372Mi           
mobon-data-zookeeper-2                                3m           347Mi           
mobon-gateway-api-common-6c5fbccf77-78vrr             107m         976Mi           
mobon-gateway-api-common-6c5fbccf77-cln5f             104m         977Mi           
mobon-gateway-api-common-6c5fbccf77-kk8ch             151m         973Mi           
mobon-gateway-api-common-6c5fbccf77-q76kl             101m         980Mi           
mobon-gateway-api-common-6c5fbccf77-tr65j             95m          971Mi           
mobon-gateway-api-common-6c5fbccf77-wsvwl             154m         968Mi           
mobon-gateway-service-aggregation-7f6888b477-4md4f    353m         1773Mi          
mobon-gateway-service-aggregation-7f6888b477-kb26z    361m         1632Mi          
mobon-gateway-service-aggregation-7f6888b477-klsmj    291m         1729Mi          
mobon-gateway-service-aggregation-7f6888b477-qmpx4    348m         1691Mi          
mobon-gateway-service-aggregation-7f6888b477-r2kmm    264m         1710Mi          
mobon-gateway-service-aggregation-7f6888b477-t8shf    259m         1722Mi          
mobon-kafka-manager-67b7889896-th8rm                  0m           6Mi             
mobon-security-authorization-oauth-7b99db6b88-ddhj5   7m           633Mi           
mobon-security-authorization-oauth-7b99db6b88-lrlss   9m           609Mi           
mobon-security-authorization-oauth-7b99db6b88-tvq9t   9m           637Mi           
mobon-service-adid-8596f7d7dc-gkw87                   15m          960Mi           
mobon-service-adid-8596f7d7dc-v9nsd                   21m          963Mi           
mobon-service-adid-8596f7d7dc-wccdf                   22m          960Mi           
mobon-service-conversion-596bfb8655-4zdnc             140m         1056Mi          
mobon-service-conversion-596bfb8655-mc72j             106m         930Mi           
mobon-service-conversion-596bfb8655-xcfmz             158m         1062Mi          
mobon-service-inclination-68c9f8bd77-8mrn4            2445m        1204Mi          
mobon-service-inclination-68c9f8bd77-g7ppf            2474m        1162Mi          
mobon-service-inclination-68c9f8bd77-pg8kp            2739m        1257Mi          
mobon-service-keyword-7fdcd487bc-h7zwz                9m           1395Mi          
mobon-service-keyword-7fdcd487bc-r572q                8m           1241Mi          
mobon-service-keyword-7fdcd487bc-stqhb                8m           1402Mi          
mobon-service-product-79cdbc6565-4n4qj                216m         868Mi           
mobon-service-product-79cdbc6565-9flx2                34m          780Mi           
mobon-service-product-79cdbc6565-c9xmj                463m         1046Mi          
mobon-service-product-79cdbc6565-cffwx                34m          712Mi           
mobon-service-product-79cdbc6565-cz5pq                33m          736Mi           
mobon-service-product-79cdbc6565-fvdbc                497m         1039Mi          
mobon-service-product-79cdbc6565-hl62c                31m          791Mi           
mobon-service-product-79cdbc6565-k6669                205m         852Mi           
mobon-service-product-79cdbc6565-pswrw                342m         1011Mi          
mobon-service-product-79cdbc6565-s2v5s                76m          848Mi           
mobon-service-product-79cdbc6565-w6fdn                243m         808Mi           
mobon-service-product-79cdbc6565-wkx9z                417m         1019Mi          
```

## 9. Appendix

#### reference site

* Kubernetes Metrics Server  
https://github.com/kubernetes-sigs/metrics-server  

+ 쿠버네티스 모니터링 : metrics-server (kubernetes monitoring : metrics-server)  
https://arisu1000.tistory.com/27856  
