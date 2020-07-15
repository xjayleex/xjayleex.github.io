---
layout: post
comment: false
title: AWS - Compute
date: 2020-03-17
category: "AWS"
tags: [AWS]
author: Jaehyun Lee
---
### Understand EC2



#### **EC2 Feature**
---
- EC2 Instance  가상 컴퓨팅 환경
- AMI(Amazon Machine Image)   서버에 필요한 OS와 추가적인 소프트웨어들이 패키지 형태로 구성된 상태로 제공되는 템플릿으로 인스턴스를 쉽게 생성 가능
- Instance Type  여러가지 구성으로 CPU, Memory, Storage, Network 제공
- Key Pair  AWS는 Public Key를 저장하고 유저는 Private Key를 안전한 장소에 보관하는 방식으로 인스턴스 로그인 정보를 보호
- EBS(Amazon Elastic Volume)을 이용해 영구적 데이터 저장
- Region & Availability Zones  인스턴스와 Amazone EBS 볼륨 등의 리소스를 다른 물리적 장소에서 액세스 가능
- Secure Group을 통해 인스턴스에 연결할 수 있는 프로토콜, 포트, 소스 IP 범위를 지정하는 Firewall 기능
- EIP(Elastic IP)  동적 클라우드 컴퓨팅을 위한 고정 IPv4 주소
- Tag  사용자가 생성하여 EC2 리소스에 할당할 수 있는 Metadata
- VPC(Virtual Priavte Clouds)  AWS 클라우드에서는 논리적으로 격리되어 있지만, 원할 때 마다 사용자 네트워크와 연결할 수 있는 가상 네트워크

#### **Accessing EC2**
---
##### Amazon EC2 Console
- Web 기반 사용자 인터페이스  

##### AWS CLI
- Window, Mac, Linux 지원  

##### AWS Query
- "Action"이라는 쿼리 변수로, GET, POST Request로 HTTP, HTTPS Request 가능  

##### AWS SDK Libraries
- HTTP나 HTTPS Request를 직접 보내는 대신, 각 언어가 제공하는 고유의 API를 사용하도록 라이브러리 제공  
- 이 라이브러리를 통해서 HTTP/HTTPS Request에 암호화된 사인, Request Retrying, Error Response 핸들링 등의 작업을 자동화할 수 있는 기능 제공

#### **AWS EC2 - AMI**
---
- AMI는 인스턴스를 시작하는 데 필요한 정보를 제공
- 동일한 구성의 인스턴스가 여러 개 필요할 때에 한 AMI에서 여러 인스턴스 구동 가능
- VPC 내에서, 다양한 AMI를 사용해 인스턴스들을 구동 가능

##### AMI는 다음을 포함
- 1개 이상의 EBS Snapshot 또는, 인스턴스 저장 지원 AMI의 경우, 인스턴스의 루트 볼륨에 대한 템플릿(e.g. OS, Applcation Server, Apps)
- AMI를 사용해 인스턴스를 시작할 수 있는 AWS 계정을 제어하는 시작 권한
- 시작될 때 인스턴스에 연결할 볼륨을 지정하는 블록 디바이스 매핑

#### **AMI Types**
---
다음 유형을 기준으로 사용할 AMI 선택 가능
##### - Region & Availability Zone
##### - Operating System
##### - Computer Architecture
##### - 시작 권한(Lauch PErmissions)
AMI 소유자는 시작 권한을 지정하여 가용성 결정
- Public  모든 AWS 계정에 시작 권한 부여
- Explicit  특정 AWS 계정들에 시작 권한 부여
- Private/Implicit  Creator Account Only  
##### - Root Device Storage
- 모든 AMI는 Amazon EBS나 Instance Store를 루트 디바이스 스토리지로 사용
EBS Backed
- EBS 볼륨은 EC2 인스턴스의 생명주기로부터 독립적으로 유지된다.
- EBS backed 인스턴스는 볼륨 삭제없이 중지될 수 있다.
- EBS 인스턴스는 Delete On Termination 플래그가 활성화되어 있지 않은 이상, 볼륨 데이터를 유지한다. (Default는 True로 설정되어 있음)
- 인스턴스를 사용하기 전에 스냅샷에서 인스턴스를 부팅하는 데 필요한 부분만 검색하면 되므로 EBS Backed 인스턴스는 Instance Store Backed 인스턴스보다 훨씬 빠르게 부팅된다.
- AMI 생성에 CreateImage API  단일 명령/호출을 사용해 AMI 생성이 쉽다.
Instance Store backed
- 
