--
> 여기부터 master/worker node 모두 수행

## set account and directory
```
su -

groupadd -g 3000 app
useradd -d /apps -g 3000 -m -u 3000 -s /bin/bash app
passwd app

usermod -aG wheel app

mkdir -p /data
chown -R app:app /data

su - app

mkdir -p /apps/install
cd /apps/install
```

## docker 설치
```
sudo yum install -y policycoreutils-python

curl -O http://mirror.centos.org/centos/7/extras/x86_64/Packages/container-selinux-2.107-1.el7_6.noarch.rpm
sudo rpm -Uvh container-selinux-2.107-1.el7_6.noarch.rpm

curl -fsSL https://get.docker.com/ | sudo sh

sudo usermod -aG docker app

cat <<EOF > /apps/install/docker/daemon.json
{
    "graph": "/data/docker"
}
EOF
sudo cp /apps/install/docker/daemon.json /etc/docker/daemon.json

sudo systemctl enable docker
sudo systemctl start docker

sudo sysctl net.bridge.bridge-nf-call-iptables=1
cat <<EOF > /apps/install/docker/docker.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo cp /apps/install/docker/docker.conf /etc/sysctl.d/docker.conf
sudo sysctl --system
```
> `기동 후 변경`  
> sudo mv /var/lib/docker /data/docker  
> sudo systemctl restart docker

## kubernetes 설치
```
cat <<EOF > /apps/install/kubernetes/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF
sudo cp /apps/install/kubernetes/kubernetes.repo /etc/yum.repos.d/kubernetes.repo

sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl enable kubelet && systemctl start kubelet
```

## system : swap 제거
```
sudo sed -r -i '/ swap / s/^(.*)$/#\1/g' /etc/fstab
sudo swapoff -a
```

## loadbalancer : haproxy 설치
[haproxy](../haproxy/install.n.setup.md)

## etcd 설치
[etcd cluster](../etcd/install.n.setup.md)

> 여기까지 master/worker node 모두 수행
--
> 여기부터 master node 수행

## master node 설정
> Stacked control plane and etcd nodes  
> **External etcd nodes**

$ mkdir -p /apps/kubernetes

$ vi /apps/kubernetes/config.yaml
```
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: stable
apiServerCertSANs:
- 10.251.0.194
controlPlaneEndpoint: "10.251.0.194:6443"
etcd:
  external:
    endpoints:
      - http://10.251.0.191:2379
      - http://10.251.0.192:2379
      - http://10.251.0.193:2379
#      - https://10.251.0.191:2379
#      - https://10.251.0.192:2379
#      - https://10.251.0.193:2379
#    caFile: /apps/kubernetes/etcd/ca.pem
#    certFile: /apps/kubernetes/etcd/kubernetes.pem
#    keyFile: /apps/kubernetes/etcd/kubernetes-key.pem
networking:
  dnsDomain: mobon.platform
  podSubnet: 10.244.0.0/16
#  serviceSubnet: 10.96.0.0/12
apiServerExtraArgs:
  apiserver-count: "3"
```

$ sudo kubeadm init --config=/apps/kubernetes/config.yaml
```
...
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes by copying certificate authorities 
and service account keys on each node and then running the following as root:

  kubeadm join 10.251.0.194:6443 --token jsfxu6.0kj84vtirx0vezb7 \
    --discovery-token-ca-cert-hash sha256:ea609f534e6527f8d5ffb5b5cf2488fa79f9c231401386577872037a3338dc3e \
    --control-plane 	  

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.251.0.194:6443 --token jsfxu6.0kj84vtirx0vezb7 \
    --discovery-token-ca-cert-hash sha256:ea609f534e6527f8d5ffb5b5cf2488fa79f9c231401386577872037a3338dc3e 
```

#### other master cluster nodes 설정

`copy the certificates from MPK-Cluster-09`  
[app@MPK-Cluster-09 pki]$ cp -R /etc/kubernetes/pki /apps/kubernetes  
[app@MPK-Cluster-09 pki]$ rm -fr /apps/kubernetes/pki/apiserver.*  

`copy the certificates to MPK-Cluster-10 and kubeadm init`  
[app@MPK-Cluster-10 pki]$ sudo cp -R /apps/kubernetes/pki /etc/kubernetes/  
[app@MPK-Cluster-10 pki]$ sudo kubeadm init --config=/apps/kubernetes/config.yaml  
>kubeadm join 10.251.0.194:6443 --token 1sxpyh.0v06p5ik5osjw5dn \  
>    --discovery-token-ca-cert-hash sha256:ea609f534e6527f8d5ffb5b5cf2488fa79f9c231401386577872037a3338dc3e 
>
>`copy the certificates to MPK-Cluster-11 and kubeadm init`   
>[app@MPK-Cluster-11 pki]$ sudo cp -R /apps/kubernetes/pki /etc/kubernetes/  
>[app@MPK-Cluster-11 pki]$ sudo kubeadm init --config=/apps/kubernetes/config.yaml  
>kubeadm join 10.251.0.194:6443 --token 6kbq9e.qirblr0o52gplk5l \  
>    --discovery-token-ca-cert-hash sha256:ea609f534e6527f8d5ffb5b5cf2488fa79f9c231401386577872037a3338dc3e 

#### root 이외 계정에서 사용 시 아래와 같이 환경 설정을 추가한다.
$ mkdir -p $HOME/.kube  
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config

#### add pod network
> 클러스터링된 서버중 한대에만 하면 되는 듯....  
$ export KUBECONFIG=$HOME/.kube/config  
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

#### scheduling에서 master node의 pods 제외
$ kubectl taint nodes --all node-role.kubernetes.io/master-  
>$ kubectl taint nodes --all node-role.kubernetes.io/master-node/k8s-master untainted  

> 여기까지 master node 수행
--
> 여기부터 worker node 수행

## worker nodes
$ sudo kubeadm join 10.251.0.194:6443 --token 6kbq9e.qirblr0o52gplk5l \  
    --discovery-token-ca-cert-hash sha256:ea609f534e6527f8d5ffb5b5cf2488fa79f9c231401386577872037a3338dc3e 

#### token 확인(master node에서)
$ kubeadm token list  

$ openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'  

## 9. Appendix

#### reference site

* 고가용성 토폴로지 선택  
https://kubernetes.io/ko/docs/setup/production-environment/tools/kubeadm/ha-topology/

* Installing kubeadm  
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#before-you-begin

* Set up a High Availability etcd cluster with kubeadm  
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/setup-ha-etcd-with-kubeadm/

+ Docker with Kubernetes #9 - Master Node의 HA 구성  
https://crystalcube.co.kr/203

+ Docker with Kubernetes #10 - ETCD의 HA 구성 - Master Node와 ETCD를 HA(High Availability) 구성하기  
https://crystalcube.co.kr/204

+ Install and configure a multi-master Kubernetes cluster with kubeadm  
https://blog.inkubate.io/install-and-configure-a-multi-master-kubernetes-cluster-with-kubeadm/

+ HA Kubernetes with Kubeadm  
https://medium.com/@bambash/ha-kubernetes-cluster-via-kubeadm-b2133360b198

