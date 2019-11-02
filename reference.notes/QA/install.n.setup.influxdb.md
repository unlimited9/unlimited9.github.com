influxdb 실행 

서버관 구 : Grafana, Influxdb, Telegraf

#### kubernetes-dashboard pod
$ kubectl get pods --all-namespaces
```
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
...
kube-system   kubernetes-dashboard-5f7b999d65-67xz4      1/1     Running   0          23s
...
```

> `접속을 위한 바인딩 정보 확인`
>$ kubectl -n kube-system get service kubernetes-dashboard
>```
>NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
>kubernetes-dashboard   ClusterIP   10.102.183.54   <none>        443/TCP   3m49s
>```

#### NodePort

`Dashboard 서비스 설정 변경`
$ kubectl -n kube-system edit service kubernetes-dashboard
```
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be



influxdb download : https://portal.influxdata.com/downloads

$ wget https://dl.influxdata.com/influxdb/releases/influxdb_1.5.1_amd64.deb



$ sudo dpkg -i influxdb_1.5.1_amd64.deb



라이브러리 설치

$ sudo sed -i -e 's/us.archive.ubuntu.com/archive.ubuntu.com/g' /etc/apt/sources.list



$ sudo apt-get update



$ sudo apt-get install curl



$ apt-get install apt-transport-https ( https 접근에 문제가 있을경우에 )



인스톨 : https://docs.influxdata.com/influxdb/v1.5/introduction/installation/

$ curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add -



$ source /etc/lsb-release



$ echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/influxdb.list



실행

$ sudo apt-get update && sudo apt-get install influxdb

$ sudo service influxdb start

$ influxd

$ influx



출처: https://serpiko.tistory.com/698 [삽SAP(Study And Programming) 질 블로그. by허정진]

출처: https://serpiko.tistory.com/698 [삽SAP(Study And Programming) 질 블로그. by허정진]
