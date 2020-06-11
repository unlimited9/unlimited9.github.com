## kubernetes cluster node 

#### creating required operating system group and user
[Create operating system group and user](../../system/management.account.n.group.md)

#### install dependency packages
[Docker Install and setup](../docker/install.n.setup.md)  
$ curl -fsSL https://get.docker.com/ | sudo sh  
$ sudo usermod -aG docker $USER  
$ sudo systemctl start docker && systemctl restart docker  

[Kubernetes Install and setup](./install.n.setup.md)  
$ sudo vi /etc/yum.repos.d/kubernetes.repo  
$ sudo yum update  
$ sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes  
$ sudo systemctl enable kubelet && systemctl start kubelet  

$ sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab  
$ sudo swapoff -a  

#### add pki
$ cp [certificate and key files] /etc/kubernetes/pki  

#### add node to kubernetes cluster  
$ sudo kubeadm join 192.168.100.12:6443 --token fpcq1q.nb2okgy5x2r6xlxz --discovery-token-ca-cert-hash sha256:4574c39be1691106f94bf18ff5d4e9c2c45f390ea53d3786dac4880320778203  
> add master node : --control-plane  

#### delete node to kubernetes cluster  
$ kubectl drain $NODE_NAME --delete-local-data --force --ignore-daemonsets  
$ kubectl delete node <node name>  

## 9. Appendix

#### reference site




