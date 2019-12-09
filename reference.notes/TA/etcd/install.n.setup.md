# Install and setup : etcd

>Created 목요일 30 11월 2017  

## 1. Pre-installation setup

#### A. creating required operating system group and user
[Create operating system group and user](../system/management.account.n.group.md)

#### B. creating base directory
[Create operating system drectory](../system/management.directory.md)

## 2. installation setup : app

#### change account
>$ su - app

#### creating application directory
$ mkdir -p /apps/etcd  
$ mkdir -p /data/etcd  

#### download
`https://github.com/etcd-io/etcd/releases`  
$ wget https://github.com/etcd-io/etcd/releases/download/v3.3.18/etcd-v3.3.18-linux-amd64.tar.gz

#### decompress tarball
$ tar -xvf etcd-v3.3.18-linux-amd64.tar.gz
$ mv /apps/install/etcd-v3.3.18-linux-amd64 /apps/etcd/3.3.18

## 3. post-installation setup

#### create systemd unit file
$ vi /etc/hosts
```
...
# mpk-etcd-cluster
mpk-etcd-01 10.251.0.191
mpk-etcd-02 10.251.0.192
mpk-etcd-03 10.251.0.193
...
```

ref. systemd.configuration
```
#!/usr/bin/env bash

_NAME="mpk-etcd"
_HOSTS=("10.251.0.191" "10.251.0.192" "10.251.0.193")

for IDX in "${!_HOSTS[@]}"; do
_HOST=${_HOSTS[$IDX]}
cat << EOF > etcd.service.$_HOST
[Unit]
Description=etcd
Documentation=https://github.com/etcd-io

[Service]
ExecStart=/usr/local/bin/etcd \\
  --name $_NAME-`printf %02d ${IDX+1}` \\
  --data-dir=/data/etcd \\
  --initial-cluster-state new \\
  --initial-cluster-token $_NAME-cluster-`printf %02d ${IDX+1}` \\
  --initial-cluster mpk-etcd-01=https://${_HOSTS[0]}:2380,mpk-etcd-02=https://${_HOSTS[1]}:2380,mpk-etcd-03=https://${_HOSTS[2]}:2380 \\
  --initial-advertise-peer-urls https://$_HOST:2380 \\
  --advertise-client-urls https://$_HOST:2379 \\
  --listen-peer-urls https://$_HOST:2380 \\
  --listen-client-urls https://$_HOST:2379,http://127.0.0.1:2379
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
done
```

$ sudo vi /etc/systemd/system/etcd.service
```
[Unit]
Description=etcd
Documentation=https://github.com/etcd-io

[Service]
ExecStart=/usr/local/bin/etcd \
  --name mpk-etcd-01 \
  --data-dir=/data/etcd
  --initial-cluster-state new \
  --initial-cluster-token mpk-etcd-cluster-01 \
  --initial-cluster etcd-01=https://10.251.0.191:2380,etcd-02=https://10.251.0.192:2380,etcd-03=https://10.251.0.193:2380 \
  --initial-advertise-peer-urls https://10.251.0.191:2380 \
  --advertise-client-urls https://10.251.0.191:2379 \
  --listen-peer-urls https://10.251.0.191:2380 \
  --listen-client-urls https://10.251.0.191:2379,http://127.0.0.1:2379 \
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```
>  --cert-file=/apps/etcd/pki/kubernetes.pem \
>  --key-file=/apps/etcd/pki/kubernetes-key.pem \
>  --peer-cert-file=/apps/etcd/pki/kubernetes.pem \
>  --peer-key-file=/apps/etcd/pki/kubernetes-key.pem \
>  --trusted-ca-file=/apps/etcd/pki/ca.pem \
>  --peer-trusted-ca-file=/apps/etcd/pki/ca.pem \
>  --peer-client-cert-auth \
>  --client-cert-auth \


#### reload the daemon configuration.
$ sudo systemctl daemon-reload

#### enable etcd to start at boot time.
$ sudo systemctl enable etcd

#### start etcd.
$ sudo systemctl start etcd

#### verify that the cluster is up and running.
$ ETCDCTL_API=3 etcdctl member list

## 8. trouble-shooting

## 9. Appendix

#### reference site

* etcd-io/etcd
https://github.com/etcd-io/etcd/releases

+ etcd 3.2.3 클러스터링하기  
https://knight76.tistory.com/entry/etcd-323-%ED%81%B4%EB%9F%AC%EC%8A%A4%ED%84%B0%EB%A7%81%ED%95%98%EA%B8%B0

+ Ubuntu etcd 설치  
https://sarc.io/index.php/cloud/1739-ubuntu-etcd-installation

+ Install and configure a multi-master Kubernetes cluster with kubeadm  
https://blog.inkubate.io/install-and-configure-a-multi-master-kubernetes-cluster-with-kubeadm/
