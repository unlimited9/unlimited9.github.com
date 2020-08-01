# Ultralight, security-first service mesh for Kubernetes
Linkerd adds critical security, observability, and reliability features to your Kubernetes stack—no code change required.

##

#### windows docker/kubernetes setup

`download linkerd-cli`  
https://github.com/linkerd/linkerd2/releases/download/edge-20.7.5/linkerd2-cli-edge-20.7.5-windows.exe

`rename and set path`  
D:\repo\apps\linkerd\2.x\bin\linkerd.exe  

`install Linkerd onto the cluster`  
linkerd version  
linkerd check --pre  
linkerd install | kubectl apply -f -  
linkerd check  
kubectl -n linkerd get deploy  
```
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
linkerd-controller       1/1     1            1           103s
linkerd-destination      1/1     1            1           103s
linkerd-grafana          1/1     1            1           101s
linkerd-identity         1/1     1            1           103s
linkerd-prometheus       1/1     1            1           100s
linkerd-proxy-injector   1/1     1            1           102s
linkerd-sp-validator     1/1     1            1           101s
linkerd-tap              1/1     1            1           101s
linkerd-web              1/1     1            1           102s
```

#### explore linkerd
linkerd dashboard &  
linkerd -n linkerd top deploy/linkerd-web  


#### install the demo app
curl -sL https://run.linkerd.io/emojivoto.yml | kubectl apply -f -  
kubectl -n emojivoto port-forward svc/web-svc 8080:80  
kubectl get -n emojivoto deploy -o yaml | linkerd inject - | kubectl apply -f -  
linkerd -n emojivoto check --proxy  

#### watch it run
linkerd -n emojivoto stat deploy  
linkerd -n emojivoto top deploy  
linkerd -n emojivoto tap deploy/web  

## appendix

#### reference.site

* Linkerd  
https://linkerd.io/  

* Linkerd : Getting Started  
https://linkerd.io/2/getting-started/  
