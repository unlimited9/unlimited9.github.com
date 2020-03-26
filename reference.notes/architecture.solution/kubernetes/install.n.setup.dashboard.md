## Web UI (Dashboard) 설치

Master node에서 실행

$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-rc2/aio/deploy/recommended.yaml  
> $ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml

#### kubernetes-dashboard pod
> v2 부터는 namespace가 kube-system에서 kubernetes-dashboard로 변경  

$ kubectl get pods --all-namespaces
```
NAMESPACE              NAME                                       READY   STATUS    RESTARTS   AGE
...
kubernetes-dashboard   kubernetes-dashboard-5f7b999d65-67xz4      1/1     Running   0          23s
...
```


NodePort, Proxy, API Server, Ingress 등으로 서비스에 접속이 가능하지만 API Server를 통해 접근하는것이 가장 합리적이라 생각됨

#### NodePort
>`접속을 위한 바인딩 정보 확인`  
>$ kubectl -n kubernetes-dashboard get service kubernetes-dashboard
>```
>NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
>kubernetes-dashboard   ClusterIP   10.102.183.54   <none>        443/TCP   3m49s
>```

`Dashboard 서비스 설정 변경`  
$ kubectl -n kubernetes-dashboard edit service kubernetes-dashboard
```
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2019-04-25T09:33:59Z"
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
  resourceVersion: "1878"
  selfLink: /api/v1/namespaces/kube-system/services/kubernetes-dashboard
  uid: 414099b7-673d-11e9-9202-ecebb898c114
spec:
  clusterIP: 10.98.252.101
  ports:
  - port: 443
    protocol: TCP
    targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
  sessionAffinity: None
#  type: ClusterIP
  type: NodePort
status:
  loadBalancer: {}
```
>요부분만 변경  
>```
>#  type: ClusterIP
>type: NodePort
>```

`접속을 위한 바인딩 정보 확인`  
$ kubectl -n kubernetes-dashboard get service kubernetes-dashboard
```
NAME                   TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
kubernetes-dashboard   NodePort   10.98.252.101   <none>        443:30444/TCP   22h
```

`Access URL`  
https://10.251.0.191:30444/

`Kubernetes  Dashboard 접속`  
1. kubeconfig  
2. **Token**

`Create service account`  
$ kubectl -n kubernetes-dashboard create serviceaccount admin-user  
> -n : namespace 지정을 안하면 namespace를 'default'로 생성한다.  
>$ kubectl create serviceaccount admin-user  
>or  
>```
>$ cat  <<EOF | kubectl create -f -
>apiVersion: v1
>kind: ServiceAccount
>metadata:
>  name: admin-user
>  namespace: kubernetes-dashboard
>EOF
>```
serviceaccount/admin-user created

`Create cluster role binding`  
$ kubectl create clusterrolebinding admin-user --clusterrole=cluster-admin --serviceaccount=kubernetes-dashboard:admin-user  
>$ kubectl create clusterrolebinding admin-user --clusterrole=cluster-admin --serviceaccount=default:admin-user  
>or  
>```
>$ cat <<EOF | kubectl create -f -
>apiVersion: rbac.authorization.k8s.io/v1
>kind: ClusterRoleBinding
>metadata:
>  name: admin-user
>roleRef:
>  apiGroup: rbac.authorization.k8s.io
>  kind: ClusterRole
>  name: cluster-admin
>subjects:
>- kind: ServiceAccount
>  name: admin-user
>  namespace: kubernetes-dashboard
>EOF
>```
>return  
clusterrolebinding.rbac.authorization.k8s.io/admin-user created

`사용자 계정 토큰으로 대시보드 로그인`  
>namespace를 지정안하면 'default' namespace를 사용한다.

$ kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')  
>$ kubectl describe secret $(kubectl get secret | grep admin-user | awk '{print $1}')  
>or  
$ kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get serviceaccount admin-user -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" | base64 --decode
```
Name:         admin-user-token-4fxnw
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin-user
              kubernetes.io/service-account.uid: f50e282a-67f9-11e9-9202-ecebb898c114

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLTRmeG53Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJmNTBlMjgyYS02N2Y5LTExZTktOTIwMi1lY2ViYjg5OGMxMTQiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06YWRtaW4tdXNlciJ9.TBAStTlp_d7h0lG0bF_yg4YIl5Fg8EjQzgZxN6MzFiUkSWWeKNJGvx9lFOkD7kY6YltRaab1IRlzil28nnmJrgaNBgQyqgHMI_bDDPvX1gAXsI8uy9nz5_Ieiupnri7lYY4BDxMFUMLrF2PMENDuZXR0YQFc4QmKzk61xUygNTpah3H9VM314L4DPex0aQGzkyKYqTp7rtRruH-zgtnySV5myDxXM8fzL6qPkyiJCHNSO-piz-wTiwEpFrf0skWnpERGIjbRshe1lxsb-CSY-_ZhrYHYF04C6lrSB2BKl79Ai4xJBqQ_4DZvwxF-9ATZS3z3HIWtD9DYr_Z2qB4_fQ
```

#### Proxy
접속IP를 127.0.0.1에서 외부에서도 접속할 수 있도록 변경하면 아래 토큰으로 인증이 안된다.  
토큰 생성을 위와 같이 생성해서 하면 될 수도 있을 것 같다.  
>
`외부 접속 프록시 데몬 실행`  
$ kubectl proxy --port=8001 --address=127.0.0.1 --accept-hosts='^*$' &  
> $ kubectl proxy --port=8001 --address=172.20.104 --accept-hosts='^*$' &  

`Access URL`  
>usage : https://도메인/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/  
>http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/

`Kubernetes  Dashboard 접속`  
1. kubeconfig  
2. **Token**

`토큰을 대시보드에 입력`  
$ kubectl -n kube-system describe $(kubectl -n kube-system get secret -n kube-system -o name | grep namespace) | grep token
```
Name:         namespace-controller-token-2dc64
Type:  kubernetes.io/service-account-token
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJuYW1lc3BhY2UtY29udHJvbGxlci10b2tlbi0yZGM2NCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJuYW1lc3BhY2UtY29udHJvbGxlciIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjZlZGZmNzI5LTY2NWItMTFlOS1iODA4LTNjOTcwZTkyNjRhMSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTpuYW1lc3BhY2UtY29udHJvbGxlciJ9.TLaAwEopJDuSAVpj1fGWoAp6Kfxectz9RuXA5Sk4ZkL2bFy5lqO0Yb63ONSLM7Eni4xVDdT6OlV6yqhmBwSQd852xRAI06FB-867eogFg_0LmRpvCt8JUMljjP4DChPdJoa_2bEIBEioNqb0bEgVMqSPFNaCY_ZvUt3-Qt29Gd62-owO6pYiZGf8u5Ct6yjC_JnbbagljL85bJIDnjz1TJC6Hx6zY36ETUjIId5O1S90r92tIf3aeahQ_-QbUU-eam_4W4T-sc_MuSnIHEXT-eRKFP000XIdsYwzz0OLAWhnQatYodBvZDaguF_yENtyLI6dftJ-o5xIkZF4ZMR7yg
```

>```
>Name:         namespace-controller-token-m56dg
>Type:  kubernetes.io/service-account-token
>token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJuYW1lc3BhY2UtY29udHJvbGxlci10b2tlbi1tNTZkZyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJuYW1lc3BhY2UtY29udHJvbGxlciIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjdjNjc1YzJhLTY3M2EtMTFlOS05MjAyLWVjZWJiODk4YzExNCIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTpuYW1lc3BhY2UtY29udHJvbGxlciJ9.N2RLp8zMLPGspdI8AOA6j_WzpLpFfPxz4cOHiELX62TegUAOTUjHRzox6FhZjwdDLYBuLtPDIaMQXg1BgyuLN1L7zxsEDmp6XoK4SH-caZCwjQUzJejoS_NLAbqoxeQg4qANRajBMUrff2QIJfCm4nxp1dHUIUFRvnChDdB0n-lBZjUM3HhNrovgdzYFhNNRIVBlUSUs1a_JT0L1-9R5UldEFCuAW1RW4IflZYdyZJQMDvH1JB_-bnVysZaqBd0SWg0HOqUOCpDLVGOdrI6GZMpQDFcFCCepnePr9AX08CO-GwwS4oYXw4MzFlYDyzP-6nTWcTSkDvA5R_ZXd2ye9g
>```

#### API Server

$ grep 'client-certificate-data' ~/.kube/config | head -n 1 | awk '{print $2}' | base64 -d >> kubecfg.crt  
$ cat kubecfg.crt  

$ grep 'client-key-data' ~/.kube/config | head -n 1 | awk '{print $2}' | base64 -d >> kubecfg.key  
$ cat kubecfg.key  

$ openssl pkcs12 -export -clcerts -inkey kubecfg.key -in kubecfg.crt -out kubecfg.p12 -name "kubernetes-admin"  

$ kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')  
> make kubecfg.token file  

`add certificate to client browser`  
> 사용자 PC에서 작업  
> download files  
/etc/kubernetes/pki/ca.crt  
./kubecfg.p12  
./kubecfg.token

$ certutil -addstore "Root" ca.crt  
$ certutil -p password -user -importPFX kubecfg.p12  

`Access URL`  
https://[master_ip]:6443/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login

`Kubernetes  Dashboard 접속`  
1. kubeconfig  
2. **Token**

$ kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```
Name:         admin-user-token-mdtck
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin-user
              kubernetes.io/service-account.uid: 534d3928-08f8-46bd-b6b6-e1223bd00121

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  20 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6Il9mMkJRVkJwem0wTWN5ZEg4WmFKcEZiSHI3Q2dMMTZGNFJqVWJmb1hjbGsifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLW1kdGNrIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI1MzRkMzkyOC0wOGY4LTQ2YmQtYjZiNi1lMTIyM2JkMDAxMjEiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.tqXDrDpfCmGNsb6SQg7rZ8yOqMN3S0nwejQk3a2oGi2BCcEQyEfs_BdnMTSvzFuTx2flPfWlEmDf2zudAUFeKECEfZwsePvPh518bb2e8YrgXGf6kmXdR2p5e5C1YOgR74My-fV_HDtLvCog68crsCrwUlpq_M_t0hwydHiCw6Gqy3ZBKQZPiiKYPS-4QtGEIxN4yHs5Sr_Jy_6sxOEluvunF24hIIw2_Go-V2LvKrh3WUXWYt565fKs4072gFRfYRMJYJohUsMpeoYghpFVGZxSYqBjIjDM3LyEJSAE09-nF-IACTYmz7yJXaUMoGYsjetES7afgPXYkUWPg906OA
```

## 9. Appendix

#### reference site

* kubernetes / dashboard
https://github.com/kubernetes/dashboard/wiki

* Access Clusters Using the Kubernetes API  
https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/

* [Container Management] Kubernetes Dashboard Install & Setting  
https://waspro.tistory.com/516  

