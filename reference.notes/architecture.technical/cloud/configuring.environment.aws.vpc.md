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

## ggid-dev

<details>
<summary>detail</summary>
<div markdown="1">

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

</div>
</details>

## ggid-dev-eks

#### VPC
1. `vpc-ggid-dev-eks`  
  VPC ID : vpc-084aaffcd29bd5f24  
  상태 : available  
  기본 VPC : 아니요  
  IPv4 CIDR : 10.10.0.0/16  
  IPv6 CIDR : -  
  DNS 확인 : 활성화됨  
  네트워크 ACL : acl-0da11dd1bf2221fac  
  DNS 호스트 이름 : 활성화됨  
  DHCP 옵션 세트 : dopt-db7114b2  
  라우팅 테이블 : rtb-09c553ba98bb4bd1e  
  소유자 : 809599471177  
  Tags :  
    Name : vpc-ggid-dev-eks
    aws:cloudformation:logical-id : VPC  
    aws:cloudformation:stack-id : arn:aws:cloudformation:ap-northeast-2:809599471177:stack/ggid-dev-eks-vpc/490aeb40-daf0-11ea-99a4-02380a7bebca  
    aws:cloudformation:stack-name : ggid-dev-eks-vpc  

#### Subnet
1. `ggid-dev-eks-public-01`  
  서브넷 ID : subnet-02f02f65302ed6079  
  상태 : available  
  VPC : vpc-084aaffcd29bd5f24 | vpc-ggid-dev-eks  
  IPv4 CIDR : 10.10.0.0/18  
  사용 가능한 IPv4 주소 : 16377  
  IPv6 CIDR : -  
  가용 영역 : ap-northeast-2a (apne2-az1)  
  라우팅 테이블 : rtb-0ebbe5126cc9a4fcb | rtb-ggid-dev-eks-public  
  네트워크 ACL : acl-0da11dd1bf2221fac  
  기본 서브넷 : 아니요  
  퍼블릭 IPv4 주소 자동 할당 : 예  
  자동 할당 IPv6 주소 : 아니요  
  Outpost ID : -  
  소유자 : 809599471177  

2. `ggid-dev-eks-public-02`  
  서브넷 ID : subnet-0d71259cadcd45e93  
  상태 : available  
  VPC : vpc-084aaffcd29bd5f24 | vpc-ggid-dev-eks  
  IPv4 CIDR : 10.10.64.0/18  
  사용 가능한 IPv4 주소 : 16377  
  IPv6 CIDR : -  
  가용 영역 : ap-northeast-2b (apne2-az2)  
  라우팅 테이블 : rtb-0ebbe5126cc9a4fcb | rtb-ggid-dev-eks-public  
  네트워크 ACL : acl-0da11dd1bf2221fac  
  기본 서브넷 : 아니요  
  퍼블릭 IPv4 주소 자동 할당 : 예  
  자동 할당 IPv6 주소 : 아니요  
  Outpost ID : -  
  소유자 : 809599471177  

3. `ggid-dev-eks-private-01`  
  서브넷 ID : subnet-0464935417e60c7f5  
  상태 : available  
  VPC : vpc-084aaffcd29bd5f24 | vpc-ggid-dev-eks  
  IPv4 CIDR : 10.10.128.0/18  
  사용 가능한 IPv4 주소 : 16379  
  IPv6 CIDR : -  
  가용 영역 : ap-northeast-2a (apne2-az1)  
  라우팅 테이블 : rtb-0f85d7b226d80db85 | rtb-ggid-dev-eks-private-a  
  네트워크 ACL : acl-0da11dd1bf2221fac  
  기본 서브넷 : 아니요  
  퍼블릭 IPv4 주소 자동 할당 : 아니요  
  자동 할당 IPv6 주소 : 아니요  
  Outpost ID : -  
  소유자 : 809599471177  

4. `ggid-dev-eks-private-02`  
  서브넷 ID : subnet-033ce6ce5ee9e062f  
  상태 : available  
  VPC : vpc-084aaffcd29bd5f24 | vpc-ggid-dev-eks  
  IPv4 CIDR : 10.10.192.0/18  
  사용 가능한 IPv4 주소 : 16379  
  IPv6 CIDR : -  
  가용 영역 : ap-northeast-2b (apne2-az2)  
  라우팅 테이블 : rtb-074b4cbe07058a27c | rtb-ggid-dev-eks-private-b  
  네트워크 ACL : acl-0da11dd1bf2221fac  
  기본 서브넷 : 아니요  
  퍼블릭 IPv4 주소 자동 할당 : 아니요  
  자동 할당 IPv6 주소 : 아니요  
  Outpost ID : -  
  소유자 : 809599471177  

#### Elastic IP  
1. `eip-ggid-dev-eks-a`  
  할당된 IPv4 주소 : 13.124.180.186  
  유형 : 퍼블릭 IP  
  할당 ID : eipalloc-09a47698affed9f20  
  연결 ID : eipassoc-0623e3f3f4fb6f046  
  범위 : VPC  
  연결된 인스턴스 ID : -  
  프라이빗 IP 주소 : 10.10.14.44  
  네트워크 인터페이스 ID : eni-088ddd4cd8f806f48   
  네트워크 인터페이스 소유자 계정 ID : 809599471177  
  퍼블릭 DNS : ec2-13-124-180-186.ap-northeast-2.compute.amazonaws.com  
  NAT 게이트웨이 ID : nat-0d9107e889e4a50b6 (nat-ggid-dev-eks-a)  
  주소 풀 : Amazon  

2. `eip-ggid-dev-eks-b`  
  할당된 IPv4 주소 : 52.79.136.45  
  유형 : 퍼블릭 IP  
  할당 ID : eipalloc-0229cae41ad8704fa  
  연결 ID : eipassoc-0530660c63f62b723  
  범위 : VPC  
  연결된 인스턴스 ID : -  
  프라이빗 IP 주소 : 10.10.95.84  
  네트워크 인터페이스 ID : eni-0daa75a53a96ccf9a  
  네트워크 인터페이스 소유자 계정 ID : 809599471177  
  퍼블릭 DNS : ec2-52-79-136-45.ap-northeast-2.compute.amazonaws.com  
  NAT 게이트웨이 ID : nat-0501e6f2eca4a80e6 (nat-ggid-dev-eks-b)  
  주소 풀 : Amazon  

#### Internet Gateway
1. `igw-ggid-dev-eks`  
  인터넷 게이트웨이 ID : igw-096e40d3280de38ed  
  상태 : Attached  
  VPC ID : vpc-084aaffcd29bd5f24 | vpc-ggid-dev-eks  
  소유자 : 809599471177  

#### NAT Gateway
1. `nat-ggid-dev-eks-a`  
  NAT 게이트웨이 ID : nat-0d9107e889e4a50b6  
  프라이빗 IP 주소 : 10.10.14.44  
  생성됨 : 2020/08/10 19:01 GMT+9  
  상태 : Available  
  네트워크 인터페이스 ID : eni-088ddd4cd8f806f48  
  삭제됨 : -  
  상태 메시지 : -  
  VPC : vpc-084aaffcd29bd5f24 / vpc-ggid-dev-eks  
  탄력적 IP 주소 : 13.124.180.186  
  서브넷 : subnet-02f02f65302ed6079 / ggid-dev-eks-public-01  

2. `nat-ggid-dev-eks-b`  
  NAT 게이트웨이 ID : nat-0501e6f2eca4a80e6  
  프라이빗 IP 주소 : 10.10.95.84  
  생성됨 : 2020/08/10 19:01 GMT+9  
  상태 :  Available  
  네트워크 인터페이스 ID : eni-0daa75a53a96ccf9a  
  삭제됨 : -  
  상태 메시지 : -  
  VPC : vpc-084aaffcd29bd5f24 / vpc-ggid-dev-eks  
  탄력적 IP 주소 : 52.79.136.45  
  서브넷 : subnet-0d71259cadcd45e93 / ggid-dev-eks-public-02  

#### Routing Table  
0. ` (default)`  
  라우팅 테이블 ID : rtb-09c553ba98bb4bd1e
  기본 : 예
  명시적으로 다음과 연결 : -
  VPC : vpc-084aaffcd29bd5f24 | vpc-ggid-dev-eks
  소유자 : 809599471177
  라우팅 : 10.10.0.0/16 - local

1. `rtb-ggid-dev-eks-public`  
  라우팅 테이블 ID : rtb-0ebbe5126cc9a4fcb  
  기본 : 아니요  
  명시적으로 다음과 연결 : subnet-02f02f65302ed6079 | ggid-dev-eks-public-01, subnet-0d71259cadcd45e93 | ggid-dev-eks-public-02  
  VPC : vpc-084aaffcd29bd5f24 | vpc-ggid-dev-eks  
  소유자 : 809599471177  
  라우팅 : 10.10.0.0/16 - local, 0.0.0.0/0 - igw-096e40d3280de38ed | igw-ggid-dev-eks  

2. `rtb-ggid-dev-eks-private-a`  
  라우팅 테이블 ID : rtb-0f85d7b226d80db85  
  기본 : 아니요  
  명시적으로 다음과 연결 : subnet-0464935417e60c7f5 | ggid-dev-eks-private-01  
  VPC : vpc-084aaffcd29bd5f24 | vpc-ggid-dev-eks  
  소유자 : 809599471177  
  라우팅 : 10.10.0.0/16 - local, 0.0.0.0/0 - nat-0d9107e889e4a50b6 | nat-ggid-dev-eks-a  

3. `rtb-ggid-dev-eks-private-b`  
  라우팅 테이블 ID : rtb-074b4cbe07058a27c  
  기본 : 아니요  
  명시적으로 다음과 연결 : subnet-033ce6ce5ee9e062f | ggid-dev-eks-private-02  
  VPC : vpc-084aaffcd29bd5f24 | vpc-ggid-dev-eks  
  소유자 : 809599471177  
  라우팅 : 10.10.0.0/16 - local, 0.0.0.0/0 - nat-0501e6f2eca4a80e6 | nat-ggid-dev-eks-b  

#### Network ACL  
1. `acl-ggid-dev-eks`  
  네트워크 ACL ID : acl-0da11dd1bf2221fac  
  기본값 : 예  
  연결 대상 : subnet-0464935417e60c7f5, subnet-02f02f65302ed6079, subnet-033ce6ce5ee9e062f, subnet-0d71259cadcd45e93  
  VPC : vpc-084aaffcd29bd5f24 | vpc-ggid-dev-eks  
  소유자 : 809599471177  
  Inbound : 100 | 유형 - 모두 트래픽 | 프로토콜 - 모두 | 포트 - 모두 | 소스 범위 - 0.0.0.0/0 | ALLOW, * | 유형 - 모두 트래픽 | 프로토콜 - 모두 | 포트 - 모두 | 0.0.0.0/0 | 소스 범위 - DENY  
  Outbound : 100 | 유형 - 모두 트래픽 | 프로토콜 - 모두 | 포트 범위 - 모두 | 소스 - 0.0.0.0/0 | ALLOW, * | 유형 - 모두 트래픽 | 프로토콜 - 모두 | 포트 범위 - 모두 | 대상 - 0.0.0.0/0 | DENY  

#### Network ACL  
1. `acl-ggid-dev-eks`  
  네트워크 ACL ID : acl-0da11dd1bf2221fac  
  기본값 : 예  
  연결 대상 : subnet-0464935417e60c7f5, subnet-02f02f65302ed6079, subnet-033ce6ce5ee9e062f, subnet-0d71259cadcd45e93  
  VPC : vpc-084aaffcd29bd5f24 | vpc-ggid-dev-eks  
  소유자 : 809599471177  
  Inbound : 100 | 유형 - 모두 트래픽 | 프로토콜 - 모두 | 포트 범위 - 모두 | 소스 - 0.0.0.0/0 | ALLOW, * | 유형 - 모두 트래픽 | 프로토콜 - 모두 | 포트 범위 - 모두 | 소스 -  0.0.0.0/0 | DENY  
  Outbound : 100 | 유형 - 모두 트래픽 | 프로토콜 - 모두 | 포트 범위 - 모두 | 대상 - 0.0.0.0/0 | ALLOW, * | 유형 - 모두 트래픽 | 프로토콜 - 모두 | 포트 범위 - 모두 | 대상 - 0.0.0.0/0 | DENY  

#### Security Group

1. `sg-ggid-dev-eks-default`  
  보안 그룹 이름 : default  
  보안 그룹 ID : sg-0644bceaa3b4a6207  
  설명 : default VPC security group  
  VPC ID: vpc-084aaffcd29bd5f24  
  소유자 : 809599471177  
  인바운드 규칙 : 유형 - 모두 트래픽 | 프로토콜 - 전체 | 포트 범위 - 전체 | 소스 -	sg-0644bceaa3b4a6207 (default)  
  아웃바운드 규칙 : 유형 - 모두 트래픽 | 프로토콜 - 전체 | 포트 범위 - 전체 | 대상 - 0.0.0.0/0  

2. `sg-ggid-dev-eks-cluster`  
  보안 그룹 이름 : eks-cluster-sg-ggid-dev-855710397  
  보안 그룹 ID : sg-0b863d54d14d7a3a8  
  설명 : EKS created security group applied to ENI that is attached to EKS Control Plane master nodes, as well as any managed workloads.  
  VPC ID : vpc-084aaffcd29bd5f24  
  소유자 : 809599471177  
  인바운드 규칙 : 유형 - 모두 트래픽 | 프로토콜 - 전체 | 포트 범위 - 전체 | 소스 -	sg-0b863d54d14d7a3a8 (eks-cluster-sg-ggid-dev-855710397)  
  아웃바운드 규칙 : 유형 - 모두 트래픽 | 프로토콜 - 전체 | 포트 범위 - 전체 | 대상 - 0.0.0.0/0  

3. `sg-ggid-dev-eks-control-plane`  
  보안 그룹 이름 : ggid-dev-eks-vpc-ControlPlaneSecurityGroup-MU8NW31YBPJ3  
  보안 그룹 ID : sg-0d105b046bdc8f940  
  설명 : Cluster communication with worker nodes  
  VPC ID : vpc-084aaffcd29bd5f24  
  소유자 : 809599471177  
  인바운드 규칙 : 없음
  아웃바운드 규칙 : 유형 - 모두 트래픽 | 프로토콜 - 전체 | 포트 범위 - 전체 | 대상 - 0.0.0.0/0  




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



