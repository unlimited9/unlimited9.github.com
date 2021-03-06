# Ultralight, security-first service mesh for Kubernetes
Linkerd adds critical security, observability, and reliability features to your Kubernetes stack—no code change required.

##

#### Step 0: Setup
kubectl version --short  

#### Step 1: Install the CLI
curl -sL https://run.linkerd.io/install | sh  
vi ~/.bashrc  
```
...
# linkerd2 cli path
export PATH=$PATH:$HOME/.linkerd2/bin
```
source ~/.bashrc

linkerd version

#### Step 2: Validate your Kubernetes cluster
linkerd check --pre  

#### Step 3: Install Linkerd onto the cluster
linkerd install | kubectl apply -f -  
linkerd check  
kubectl -n linkerd get deploy  

#### Step 4: Explore Linkerd  
linkerd dashboard &  
```
Linkerd dashboard available at:
http://localhost:50750
Grafana dashboard available at:
http://localhost:50750/grafana
Opening Linkerd dashboard in the default browser
Failed to open Linkerd dashboard automatically
Visit http://localhost:50750 in your browser to view the dashboard
```
로컬 시스템에서 linkerd-web pod로 포트 포워딩 해주는 것으로 공통으로 접근하기 위해서는 ingress를 설정해 줘야한다.  

linkerd -n linkerd top deploy/linkerd-web  

<details>
<summary>on windows/powershell</summary>
<div markdown="1">

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

</div>
</details>

#### install the demo app
curl -sL https://run.linkerd.io/emojivoto.yml | kubectl apply -f -  
kubectl -n emojivoto port-forward svc/web-svc 8080:80  
kubectl get -n emojivoto deploy -o yaml | linkerd inject - | kubectl apply -f -  
linkerd -n emojivoto check --proxy  

#### watch it run
linkerd -n emojivoto stat deploy  
linkerd -n emojivoto top deploy  
linkerd -n emojivoto tap deploy/web  

#### remove linkerd
linkerd install --ignore-cluster | kubectl delete -f -

## appendix

#### reference.site

* Linkerd  
https://linkerd.io/  

* Linkerd : Getting Started  
https://linkerd.io/2/getting-started/  
