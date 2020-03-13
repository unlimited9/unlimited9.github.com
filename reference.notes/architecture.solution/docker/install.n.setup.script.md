## Install and setup docker 
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

mkdir -p /data/docker

cd /apps/install  
curl -O http://mirror.centos.org/centos/7/extras/x86_64/Packages/container-selinux-2.99-1.el7_6.noarch.rpm  
sudo rpm -Uvh container-selinux-2.99-1.el7_6.noarch.rpm 

sudo yum install -y policycoreutils-python

curl -fsSL https://get.docker.com/ | sudo sh  
sudo usermod -aG docker app

sudo systemctl enable docker  
sudo systemctl start docker
