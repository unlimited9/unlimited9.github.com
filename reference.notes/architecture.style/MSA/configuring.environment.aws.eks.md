# configuring.environment : AWS EKS

## AWS EKS console 

#### create container ubuntu:latest

docker run -d -it --name aws.management.console --restart=unless-stopped ubuntu  

docker exec -it aws.management.console /bin/bash  

<<<<<<<<<<<<<<<<<<<<  
`install packages`  
apt update -y  
apt-get install -y sudo net-tools iproute2 vim wget curl unzip  

`create group/user : app/app`  
groupadd -g 3000 app  
useradd -d /apps -g 3000 -m -u 3000 -s /bin/bash app  
passwd app

`create directory`  
mkdir -p /apps/install /pgms /data /logs  
chown -R app.app /apps /pgms /data /logs  

sudo adduser app sudo  
  
su - app  
mkdir /pgms/ggcore  

cd /apps/install  

`AWS CLI 설치`  
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"  
unzip awscliv2.zip  
sudo ./aws/install  
```
sudo: setrlimit(RLIMIT_CORE): Operation not permitted
You can now run: /usr/local/bin/aws --version
```
> echo "Set disable_coredump false" >> /etc/sudo.conf  

<<<<<<<<<<<<<<<<<<<<



Service mesh : AWS App Mesh



## appendix

#### reference.site

* Amazon EKS 클러스터 생성  
https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/create-cluster.html  

* Amazon EKS 시작하기
https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/getting-started.html

+ EKS Cluster 구축 - 2. VPC, Subnet, IAM, EKS Cluster 생성  
https://aws-diary.tistory.com/44  

+ [AWS] EKS 세팅하기  
https://kscory.com/dev/aws/eks-setup  
