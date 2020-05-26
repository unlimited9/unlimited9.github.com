# kubernetes metrics-server

#### node / pod monitoring  

$ kubectl top pod
```
Error from server (NotFound): the server could not find the requested resource (get services http:heapster:)
```
$ kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.6/components.yaml  

$ kubectl top node  
```
error: metrics not available yet
```

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

## 9. Appendix

#### reference site

* Kubernetes Metrics Server  
https://github.com/kubernetes-sigs/metrics-server  

+ 쿠버네티스 모니터링 : metrics-server (kubernetes monitoring : metrics-server)  
https://arisu1000.tistory.com/27856  
