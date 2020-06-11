# Upgrading kubeadm clusters

`message`  
> this version of kubeadm only supports deploying clusters with the control plane version >= 1.17.0. Current version: v1.16.3


## Determine which version to upgrade to

#### Find the latest stable 1.18 version:
`Ubuntu, Debian or HypriotOS`  
```
apt update
apt-cache madison kubeadm
# find the latest 1.18 version in the list
# it should look like 1.18.x-00, where x is the latest patch
```
`CentOS, RHEL or Fedora`  
```
yum list --showduplicates kubeadm --disableexcludes=kubernetes
# find the latest 1.18 version in the list
# it should look like 1.18.x-0, where x is the latest patch
```

## Upgrading control plane nodes

#### On your first control plane node, upgrade kubeadm:
`Ubuntu, Debian or HypriotOS`  
```
# replace x in 1.18.x-00 with the latest patch version
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm=1.18.x-00 && \
apt-mark hold kubeadm
-
# since apt-get version 1.1 you can also use the following method
apt-get update && \
apt-get install -y --allow-change-held-packages kubeadm=1.18.x-00
```
`CentOS, RHEL or Fedora`  
```
# replace x in 1.18.x-0 with the latest patch version
yum install -y kubeadm-1.18.x-0 --disableexcludes=kubernetes
```

#### Verify that the download works and has the expected version:
```
kubeadm version
```

#### Drain the control plane node:
```
# replace <cp-node-name> with the name of your control plane node
kubectl drain <cp-node-name> --ignore-daemonsets
```

#### On the control plane node, run:
```
sudo kubeadm upgrade plan
```

#### Choose a version to upgrade to, and run the appropriate command. For example:
```
# replace x with the patch version you picked for this upgrade
sudo kubeadm upgrade apply v1.18.x
```

#### Manually upgrade your CNI provider plugin.

#### Uncordon the control plane node:
```
# replace <cp-node-name> with the name of your control plane node
kubectl uncordon <cp-node-name>
```

## Upgrade additional control plane nodes

#### Same as the first control plane node but use:
```
sudo kubeadm upgrade node
```
instead of:
```
sudo kubeadm upgrade apply
```
> Also sudo kubeadm upgrade plan is not needed.

## Upgrade kubelet and kubectl

#### Upgrade the kubelet and kubectl on all control plane nodes:
`Ubuntu, Debian or HypriotOS`  
```
# replace x in 1.18.x-00 with the latest patch version
apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=1.18.x-00 kubectl=1.18.x-00 && \
apt-mark hold kubelet kubectl
-
# since apt-get version 1.1 you can also use the following method
apt-get update && \
apt-get install -y --allow-change-held-packages kubelet=1.18.x-00 kubectl=1.18.x-00
```
`CentOS, RHEL or Fedora`  
```
# replace x in 1.18.x-0 with the latest patch version
yum install -y kubelet-1.18.x-0 kubectl-1.18.x-0 --disableexcludes=kubernetes
```
>$ sudo yum install -y kubelet kubectl --disableexcludes=kubernetes  

#### Restart the kubelet
```
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

## Upgrade worker nodes

#### Upgrade kubeadm on all worker nodes:
`Ubuntu, Debian or HypriotOS`  
```
# replace x in 1.18.x-00 with the latest patch version
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm=1.18.x-00 && \
apt-mark hold kubeadm
-
# since apt-get version 1.1 you can also use the following method
apt-get update && \
apt-get install -y --allow-change-held-packages kubeadm=1.18.x-00
```
`CentOS, RHEL or Fedora`  
```
# replace x in 1.18.x-0 with the latest patch version
yum install -y kubeadm-1.18.x-0 --disableexcludes=kubernetes
```

#### Drain the control plane node:
```
# replace <cp-node-name> with the name of your control plane node
kubectl drain <cp-node-name> --ignore-daemonsets
```

#### Upgrade the kubelet configuration
```
sudo kubeadm upgrade node
```

#### Upgrade kubelet and kubectl
`Ubuntu, Debian or HypriotOS`  
```
# replace x in 1.18.x-00 with the latest patch version
apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=1.18.x-00 kubectl=1.18.x-00 && \
apt-mark hold kubelet kubectl
-
# since apt-get version 1.1 you can also use the following method
apt-get update && \
apt-get install -y --allow-change-held-packages kubelet=1.18.x-00 kubectl=1.18.x-00
```
`CentOS, RHEL or Fedora`  
```
# replace x in 1.18.x-0 with the latest patch version
yum install -y kubelet-1.18.x-0 kubectl-1.18.x-0 --disableexcludes=kubernetes
```
>$ sudo yum install -y kubelet kubectl --disableexcludes=kubernetes  

#### Restart the kubelet
```
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

#### Uncordon the node
```
# replace <cp-node-name> with the name of your control plane node
kubectl uncordon <cp-node-name>
```

#### Verify the status of the cluster
```
kubectl get nodes
```

## 9. Appendix

#### reference site

* Upgrading kubeadm clusters  
https://kubernetes.io/ko/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
