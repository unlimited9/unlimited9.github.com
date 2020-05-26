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


## 9. Appendix

#### reference site

* Kubernetes Metrics Server  
https://github.com/kubernetes-sigs/metrics-server  

+ 쿠버네티스 모니터링 : metrics-server (kubernetes monitoring : metrics-server)  
https://arisu1000.tistory.com/27856  
