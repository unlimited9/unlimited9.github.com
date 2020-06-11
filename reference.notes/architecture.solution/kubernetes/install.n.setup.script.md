# setup

groupadd -g 3000 app  

useradd -d /apps -g 3000 -m -u 3000 -s /bin/bash app  
passwd app  

mkdir -p /apps/install  
mkdir -p /pgms  
mkdir -p /data  
mkdir -p /logs  

chown -R app:app /apps  
chown -R app:app /pgms  
chown -R app:app /data  
chown -R app:app /logs  

su - app  

>sudo apt-get update  
sudo apt-get install -y docker.io  
>
>apt-get update && apt-get install -y apt-transport-https  
>curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -  
>
>cat <<EOF >/etc/apt/sources.list.d/kubernetes.list  
>deb http://apt.kubernetes.io/ kubernetes-xenial main  
>EOF  
>
>apt-get update  
>apt-get install -y kubelet kubeadm kubectl  

sudo vi /etc/yum.repos.d/kubernetes.repo  
```
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
```

sudo yum update  

sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes  

sudo systemctl  enable kubelet && systemctl start kubelet  

sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab  
sudo swapoff -a  

### master
firewall-cmd --zone=public  --permanent --add-port=6443/tcp  
firewall-cmd --zone=public  --permanent --add-port=2379-2380/tcp  
firewall-cmd --zone=public --permanent --add-port=10250/tcp  
firewall-cmd --zone=public --permanent --add-port=10251/tcp  
firewall-cmd --zone=public --permanent --add-port=10252/tcp  
firewall-cmd --reload  

sudo sysctl net.bridge.bridge-nf-call-iptables=1  

sudo kubeadm init --pod-network-cidr=10.244.0.0/16  
```
...
kubeadm join 172.20.0.31:6443 --token 93lcrd.xlrmkrick0zuzz3q \
    --discovery-token-ca-cert-hash sha256:6c9c7e0ccf33693345a89658badb95054a3b50dca47f93bcd9080ad1bb841725 
```

mkdir -p $HOME/.kube  
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  
sudo chown $(id -u):$(id -g) $HOME/.kube/config  
export KUBECONFIG=$HOME/.kube/config  

#### Pod Network 추가
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

kubectl taint nodes --all node-role.kubernetes.io/master-

### worker

#### join to kubernetes master
sudo kubeadm join 172.20.0.31:6443 --token cl9ae5.6ybdxgycoqglyjo9 --discovery-token-ca-cert-hash sha256:4ddb92af1f45ee771dc8ecfcddfa516440641ff6547490a9e6a9eb30e1ff0637 

#### check
`master node`  
kubectl get nodes  

### Web UI (Dashboard) 설치
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml  

kubectl -n kube-system edit service kubernetes-dashboard  
```
...
#  type: ClusterIP
  type: NodePort
...
```

kubectl -n kube-system get service kubernetes-dashboard  
```
NAME                   TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
kubernetes-dashboard   NodePort   10.108.126.210   <none>        443:32388/TCP   16h
```

`Access URL`  
https://172.20.0.31:32388/  

`Create service account`  
kubectl create serviceaccount admin-user  
`Create cluster role binding`  
kubectl create clusterrolebinding admin-user --clusterrole=cluster-admin --serviceaccount=default:admin-user  
`get account security token`  
kubectl describe secret $(kubectl get secret | grep admin-user | awk '{print $1}')  



