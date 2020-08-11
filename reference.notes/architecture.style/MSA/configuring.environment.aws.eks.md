# configuring.environment : AWS EKS

## AWS Network 기본 구성  

[AWS Network 구성](configuring.environment.aws.vpc.md)

## AWS EKS CLI 

<details>
<summary>docker container : ubuntu:latest</summary>
<div markdown="1">

#### create container ubuntu:latest
docker run -d -it --name aws.management.console --restart=unless-stopped ubuntu  
docker exec -it aws.management.console /bin/bash  

<<<<<<<<<<<<<<<<<<<<  
#### install packages  
apt update -y  
apt-get install -y sudo net-tools iproute2 vim wget curl unzip  

#### create group/user : app/app  
groupadd -g 3000 app  
useradd -d /apps -g 3000 -m -u 3000 -s /bin/bash app  
passwd app

#### create directory  
mkdir -p /apps/install /pgms /data /logs  
chown -R app.app /apps /pgms /data /logs  

sudo adduser app sudo  
  
su - app  
mkdir /pgms/ggcore  

cd /apps/install  

#### AWS CLI 설치  
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"  
unzip awscliv2.zip  
sudo ./aws/install  
```
sudo: setrlimit(RLIMIT_CORE): Operation not permitted
You can now run: /usr/local/bin/aws --version
```
> echo "Set disable_coredump false" >> /etc/sudo.conf  

`AWS CLI 자격 증명 구성`  
$ aws configure  
AWS Access Key ID [None]:  
AWS Secret Access Key [None]:  
Default region name [None]: ap-northeast-2  
Default output format [None]: json  

#### eksctl 설치  
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version

#### kubectl 설치 및 구성  
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.17.7/2020-07-08/bin/linux/amd64/kubectl  
chmod +x ./kubectl  
sudo mv ./kubectl /usr/local/bin  
kubectl version --short --client  

aws eks -ap-northeast-2 update-kubeconfig --name ggid-dev  

#### aws-iam-authenticator 설치  
curl -o aws-iam-authenticator https://amazon-eks.s3.us-west-2.amazonaws.com/1.17.7/2020-07-08/bin/linux/amd64/aws-iam-authenticator  
chmod +x ./aws-iam-authenticator  
sudo mv ./aws-iam-authenticator /usr/local/bin/  
aws-iam-authenticator help  

aws-iam-authenticator token -i ggid-dev | python3 -m json.tool  

>#### Creating, displaying, and deleting Amazon EC2 key pairs  
>`Create a key pair`  
>aws ec2 create-key-pair --key-name ggid-dev-key-pair --query 'KeyMaterial' --output text > ggid-dev-key-pair.pem  
>> on windows  
>> PS C:\>aws ec2 create-key-pair --key-name ggid-dev-key-pair --query 'KeyMaterial' --output text | out-file -encoding ascii -filepath ggid-dev-key-pair.pem  
>
>chmod 400 ggid-dev-key-pair.pem  
>
>`Display your key pair`  
>aws ec2 describe-key-pairs --key-name ggid-dev-key-pair  
>
>`Delete your key pair`  
>aws ec2 delete-key-pair --key-name ggid-dev-key-pair  
>
>`Retrieving the public key for your key pair`  
>ssh-keygen -y -f ggid-dev-key-pair.pem > ggid-dev-public-key.pub  
>>-bash: ssh-keygen: command not found  
>>sudo apt-get install openssh-client  
>
>#### Create your Amazon EKS cluster and compute
>eksctl create cluster \
>--name ggid-dev \
>--version 1.17 \
>--region ap-northeast-2 \
>--nodegroup-name ggid-dev-workers \
>--node-type t3.medium \
>--nodes 3 \
>--nodes-min 1 \
>--nodes-max 4 \
>--ssh-access \
>--ssh-public-key ggid-dev-public-key.pub \
>--managed
>

<<<<<<<<<<<<<<<<<<<<

</div>
</details>

## AWS Management Console

#### EKS 권한 생성  

`ggid-dev-eks-cluster-role`  
```
cat > eks-cluster-role.json << EOF
{
	"Version": "2020-08-10",
	"Statement": [
		{
			"Effect": "Allow",
			"Principal": {
				"Service": "eks.amazonaws.com"
			},
			"Action": "sts:AssumeRole"
		}
	]
}
EOF
aws iam create-role --role-name ggid-dev-eks-cluster-role --assume-role-policy-document file://eks-cluster-role.json
aws iam attach-role-policy --role-name ggid-dev-eks-cluster-role --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy
aws iam attach-role-policy --role-name ggid-dev-eks-cluster-role --policy-arn arn:aws:iam::aws:policy/AmazonEKSServicePolicy
```

`ggid-dev-eks-worker-role`  
```
cat > eks-worker-role.json << EOF
{
	"Version": "2020-08-10",
	"Statement": [
		{
			"Effect": "Allow",
			"Principal": {
				"Service": "ec2.amazonaws.com"
			},
			"Action": "sts:AssumeRole"
		}
	]
}
EOF
aws iam create-role --role-name ggid-dev-eks-worker-role --assume-role-policy-document file://eks-worker-role.json
aws iam attach-role-policy --role-name ggid-dev-eks-worker-role --policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy
aws iam attach-role-policy --role-name ggid-dev-eks-worker-role --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly
aws iam attach-role-policy --role-name ggid-dev-eks-worker-role --policy-arn arn:aws:iam::aws:policy/CloudWatchLogsFullAccess
aws iam attach-role-policy --role-name ggid-dev-eks-worker-role --policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy
aws iam attach-role-policy --role-name ggid-dev-eks-worker-role --policy-arn arn:aws:iam::aws:policy/AmazonRoute53FullAccess
```

#### EKS Cluster 생성

`Cluster ARN`  
arn:aws:eks:ap-northeast-2:809599471177:cluster/ggid-dev  

`Cluster IAM Role ARN`  
arn:aws:iam::809599471177:role/ggid-dev-eks-cluster-role  


`EKS Cluster 생성`  
ggid-dev


#### Node Group 추가

`1단계: 노드 그룹 구성`  
1. Name : ggid-dev-worker-node  
2. Node IAM Role : ggid-dev-eks-worker-role  
3. Subnet : subnet-0785faca258774d83, subnet-000329fe214537363  
4. 노드에 대한 원격 액세스 허용: 활성화됨  
5. SSH 키 페어 : eksctl-ggid-dev-nodegroup-ggid-dev-workers-b3:44:2c:73:8b:31:78:7c:e4:4a:d0:aa:ae:55:59:23
6. 원격 액세스 권한 허용 대상 : sg-0b162e7f6b62da13c
7. Kubernetes 레이블 (0)
8. 태그 (1) : kubernetes.io/cluster/ggid-dev : owned

`2단계: 컴퓨팅 구성 설정`  
노드 컴퓨팅 구성  
1. AMI 유형 : Amazon Linux 2 (AL2_x86_64)  
2. 인스턴스 유형 : t3.medium  
3. 디스크 크기 : 20  

`3단계: 조정 구성 설정`  
그룹 크기  
1. 최소 크기 : 1 노드  
2. 최대 크기 : 4 노드  
3. 원하는 크기 : 2 노드  

#### ref. cloudformation 사용
`Amazon EKS Linux 작업자 노드 시작`  
https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/launch-workers.html  

> https://amazon-eks.s3.us-west-2.amazonaws.com/cloudformation/2020-06-10/amazon-eks-nodegroup.yaml
> https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2019-09-27/amazon-eks-nodegroup.yaml

`AWS CloudFormation`  
https://console.aws.amazon.com/cloudformation/  


Stack Name : ggid-dev-worker-node  
#### 파라미터
`EKS Cluster`  
EKS Cluster : ggid-dev  
ClusterControlPlaneSecurityGroup : select  

`Worker Node Configuration`  
NodeGroupName : ggid-dev-nodegroup  
KeyName : select  

`Worker Network Configuration`  
VpcId : select  
Subnets : select  




## appendix

#### reference.site

* Amazon EKS 클러스터 생성  
https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/create-cluster.html  

* Amazon EKS 시작하기  
https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/getting-started.html

+ [AWS] EKS 세팅하기  
https://kscory.com/dev/aws/eks-setup  
