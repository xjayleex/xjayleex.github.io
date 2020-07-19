---
title: AWS HA & Fault Tolerance Architecture
date: 2020-07-20
category: "AWS"
tags: [AWS]
author: Jaehyun Lee
---
### **AWS High Availability & Fault Tolerance Architecture**
---
- S3, SimpleDB, SQS 및 ELB와 같은 대부분의 고급 서비스는 HA와 Fault Tolerance를 염두에 두고 구축되었음.
- EC2 및 EBS와 같은 기본 인프라를 제공하는 서비스는 AZ, EIP 및 스냅샷과 같은 HA,FT 기능을 제공하여 이를 제대로 사용해 이점을 가지도록 한다.
![Image](/assets/images/aws/AWS-High-Availability-and-Fault-Tolerance-1024x749.png){:    style=" width    : 80%; margin: 0 auto; display: block;"}

#### **Region and Availability Zone**
---
- AWS는 여러 Region 및 AZ에서 제공되므로 Redundant한 배포 역역에 쉽게 액세스 할 수 있음.
- AZ는 다른 AZ 장애로부터 격리되도록 설계된 지리적인 로케이션
- Region 및 AZ는 애플리케이션을 분산하여 FT를 높이고, Multi Site 솔루션 구축 방법을 제공.
- AZ는 동일 Region의 다른 AZ에 저렴하고 latency가 짧은 네트워크 연결 제공.
- 여러 AZ에 EC2 인스턴스를 배치하면, 단일 데이터센터에서의 애플리케이션 failure로 부터 보호.
- 하나의 Zone이 fail 하더라도 다른 Zone의 애플리케이션을 계속 실행할 수 있도록 동일 Region이나 다른 Region의 둘 이상의 AZ에서 independent 한 애플리케이션 스택을 실행하는 것이 중요.

#### **AMIs**
---
- AMI를 이용해 다른 인스턴스 유형의 서버 리소스를 생성하고, 새 인스턴스 또는 실패한 인스턴스 교체를 할 수 있다.

#### **Auto Scaling**
---
- Auto Scaling을 통해 정의된 규칙에 따라 EC2 용량을 자동으로 늘리거나 줄임.
- 또한 Auto Scaling을 통해 증가하는 로드에 대한 대응으로 더 많은 인스턴스 추가 가능. 해당 인스턴스가 더 이상 필요하지 않게되면 자동으로 종료.
- Auto Scaling을 사용하면 자동으로 교체 인스턴스가 뜰 것이라는 것을 알고 서버 인스턴스를 마음대로 종료 가능.
- Auto Scaling은 AWS Region 내의 여러 AZ에서 작동 할 수 있음.

#### **Elastic Load Balancing - ELB**
---
- ELB를 통해 시스템 Availability를 높이고 들어오는 트래픽을 여러 EC2 인스턴스에 걸쳐 애플리케이션으로 분산.
- ELB를 사용하면 DNS 호스트 네임이 생성되고, 호스트 네임으로 전송된  모든 요청이 EC2 Instance Pool으로 위임됨.
- ELB는 Host의 상태 확인, 여러 AZ에 걸쳐있는 EC2 인스턴스로 트래픽 분배, 로드밸런싱 Rotation에서 EC2 Host의 동적인 추가와 제거를 지원.
- ELB는 EC2 인스턴스 Pool 내에서 비정상 인스턴스를 감지하고 Auto Scaling을 사용해 비정상 인스턴스가 완벽히 복원 될 때까지 트래픽을 정상 인스턴스로 라우팅.
- Auto Scaling과 ELB는 이상정인 조합으로, ELB는 주소 지정을 위한 단일 DNS 네임을 제공하고, Auto Scaling은 요청을 수락 할 수 있는  정상적인 EC2 인스턴스 수가 항상 적절한지 확인.
- ELB를 사용해 한 Regon의 여러 AZ에 있는 인스턴스간에 균형을 맞출 수 있다.
