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

## master node 설정
```
sudo kubeadm init --pod-network-cidr 10.244.0.0/16 --service-dns-domain "mobon.platform"

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

export KUBECONFIG=$HOME/.kube/config

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```
> `kubeadm init 결과`  
> kubeadm join 10.251.0.191:6443 --token xqsmni.g076nsl4qq9t4r0o \
    --discovery-token-ca-cert-hash sha256:b603abc7e73847bd13ca13ba97abeb1f92b00bf1a6ec0ef2c8ad35724f2c82cb 

## master node ha 구성
```
kubelet --version

export KUBERNETES_VER=v1.16.3
export LOAD_BALANCER_DNS=10.251.0.182
export LOAD_BALANCER_PORT=6443
export CP1_HOSTNAME=MPK-Cluster-09
export CP1_IP=10.251.0.191
export CP2_HOSTNAME=MPK-Cluster-10
export CP2_IP=10.251.0.192
export CP3_HOSTNAME=MPK-Cluster-11
export CP3_IP=10.251.0.193




```




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

