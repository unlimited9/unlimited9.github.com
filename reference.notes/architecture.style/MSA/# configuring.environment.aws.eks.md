# configuring.environment : AWS EKS

## Network

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
10.0.0.0/16  

`set vpc`  
select vpc host, Actions/Edit DNS Hostnames > DNS hostname enable check  

#### create subnet  
subnet-ggid-dev-2b
10.0.2.0/24

subnet-ggid-dev-2d
10.0.4.0/24




## appendix

#### reference.site

+ 만들면서 배우는 아마존 버추얼 프라이빗 클라우드(Amazon VPC)  
https://www.44bits.io/ko/post/understanding_aws_vpc  
