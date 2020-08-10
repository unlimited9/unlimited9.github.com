# configuring.environment : AWS VPC

## AWS VPC

#### 기본 VPC의 구성 요소들  
`VPC`  

`Subnet`  

`Route Table`  

`Internet Gateway`  
라우팅 테이블에 연결해서 인터넷을 사용  
인터넷을 사용하고자 하는 리소스는 Public IP가 있어야 한다.  

`DHCP options set`  

`Network ACL / Security Group`  
가상 방화벽  
`Network ACL` : 서브넷 앞단에서 트래픽을 제어하는 역할을 합니다
`Security Group` : 인스턴스의 앞단에서 트래픽을 제어

#### create vpc
`사설망 대역` : 10.0.0.0/8, 172.16.0.0/12,  192.168.0.0/16  

vpc-ggid-dev  
10.251.0.0/16

`set vpc`  
select vpc host, Actions/Edit DNS Hostnames > DNS hostname enable check  

#### create subnet  

1. Name : subnet-ggid-dev-2a  
VPC : vpc-ggid-dev  
IPv4 CIDR : 10.251.0.0/24  
AZ : ap-northeast-2a  

2. Name : subnet-ggid-dev-2b  
VPC : vpc-ggid-dev  
IPv4 CIDR : 10.251.32.0/24  
AZ : ap-northeast-2b  

3. Name : subnet-ggid-dev-2c  
VPC : vpc-ggid-dev  
IPv4 CIDR : 10.251.640/24  
AZ : ap-northeast-2c  

4. Name : subnet-ggid-dev-2d  
VPC : vpc-ggid-dev  
IPv4 CIDR : 10.251.96.0/24  
AZ : ap-northeast-2d  

#### create internet gateway
igw-ggid-dev  

`Attach to VPC`  
Status : detached  
Actions/Attach to VPC > select VPC, Attach click  
VPC : vpc-ggid-dev  

#### create route table

1. Name : rtb-ggid-dev-public  
VPC : vpc-ggid-dev  
IPv4 CIDR : 10.251.0.0/24  
Subnet Connect : subnet-ggid-dev-2a, subnet-ggid-dev-2b  
`select Routes Tab > Edit Routes > Add Route`  
Destination : 0.0.0.0/0  
Target : Internet Gateway > igw-ggid-dev  

2. Name : rtb-ggid-dev-private  
VPC : vpc-ggid-dev  
IPv4 CIDR : 10.251.0.0/24  
Subnet Connect : subnet-ggid-dev-2c, subnet-ggid-dev-2d  

>select VPC > Routes Tab > Edit Routes > Add Route  
>Destination : 0.0.0.0/0
>Target : Internet Gateway > igw-ggid-dev  

#### EIP

#### NAT Gateway

#### ref. cloudformation 사용  
`Amazon EKS 클러스터용 VPC 생성`  
https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/create-public-private-vpc.html  

>https://amazon-eks.s3.us-west-2.amazonaws.com/cloudformation/2020-06-10/amazon-eks-vpc-private-subnets.yaml  
>https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2019-09-27/amazon-eks-vpc-sample.yaml  

`AWS CloudFormation`  
https://console.aws.amazon.com/cloudformation/  


## appendix

#### reference.site

+ 아마존 웹 서비스(AWS, Amazon Web Serivces)란?  
https://www.44bits.io/ko/keyword/amazon-web-service  

+ 만들면서 배우는 아마존 버추얼 프라이빗 클라우드(Amazon VPC)  
https://www.44bits.io/ko/post/understanding_aws_vpc  


+ AWS Network 기본 구성 구축 - 2 (Subnet, NAT, Route Table)  
https://aws-diary.tistory.com/8?category=753069  

+ [AWS] EKS 세팅하기  
https://kscory.com/dev/aws/eks-setup  



