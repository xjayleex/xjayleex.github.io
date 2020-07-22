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
- ELB를 사용해 한 Region의 여러 AZ에 있는 인스턴스간에 균형을 맞출 수 있다.

#### **Ealstic IPs**
---
- EIP는 Region 내 인스턴스간에 프로그래머블한 방식으로 매핑할 수 있는 Public 고정 IP
- EIP는 특정 인스턴스나 인스턴스의 생명주기가 아니라, AWS 계정과 연결됨.
- EIP는 Master DB, 중앙 파일 서버, EC2 Hosted 로드 밸런서와 같은 일관된 엔드포인트가 필요한 서비스와 인스턴스에 사용될 수 있다.
- EIP 주소는 실행중인 인스턴스나 교체 인스턴스로 주소를 빠르게 리매핑하여 AZ나 호스트 장애를 해결하는 데 사용 할 수 있다.

#### **Reserved Instance**
---
- 예약 인스턴스는 컴퓨팅 용량을 항상 저렴한 비용으로 예약 및 보장한다.

#### **Elastic Block Store(EBS)**
---
- EBS는 인스턴스 생명 주기와 독립적으로 유지되며, on-Instance 스토리지보다 훨씬 내구성이 뛰어는 영구적 off-Instance 스토리지 볼륨을 제공.
- EBS 볼륨은 데이터를 중복 저장하고, 단일 AZ 내에서 자동으로 복제된다.
- EC2 인스턴스가 Fail하고 교체되어야 하는 경우와 같은 장애 조치 시나리오에서 EBS 볼륨을 새 EC2 인스턴스에 연결 할 수 있다.
- 중요 데이터는 적절한 백업, 복제 또는 데이터를 재생성 할 수 있는 기능 없이 Instance 스토리지에만 저장해서는 안된다.

#### **EBS Snapshot**
---
- EBS 볼륨은 매우 안정적이지만 장애 발생 가능성을 더욱 줄이고 내구성을 높이기 위해 S3 볼륨에 데이터를 저장하고, 특정 시점에 스냅샷을 생성한 다음 여러 AZ로 복제 할 수 있음.
- 스냅샷을 이용해 새 EBS 볼륨 생성. 이 스냅샷은 스냅샷을 만드는 시점의 원래 볼륨의 정확한 Replica. 스냅샷은 AZ에 영향을 주는 문제뿐만 아니라 디스크 오류 또는 Host 수준의 문제를 효과적으로 처리 할 수 있는 방법 제공.
- 스냅샷은 이전 스냅샷 이후의 변경 사항만 백업하므로 최신 스냅샷을 유지할 것.
- 스냅샷은 Region과 연결되어 있고, EBS 볼륨은 단일 AZ와 연결되어 있음.

#### **Relational Database Service - RDS**
---
- 데이터베이스의 Synchronous Standby Replica가 다른 AZ에 프로비저닝되는 RDS Multi-AZ 배포는 데이터베이스 가용성을 높이고, 예기치 못한 중단으로부터 데이터베이스르 보호.
- 장애시, Standby가 Transparent하게 Primary로 승격되어, DB 작업 처리
- Default로 동작하는 Automated Backup은 데이터베이스 인스턴스에 대한 특정 시점으로의 백업 지원.
- RDS는 데이터베이스 및 트랜잭션 로그를 백업하고, 사용자 지정 Retention 기간 동안 저장.
- Automated Backup 외에도 명시적으로 삭제 할 떄까지 유지되는 manual Backup.
- RDS Read Replica는 데이터베이스의 읽기 전용 Replica를 제공하고 읽기 작업량이 많은 워크로드를 위해 단일 DB 배포 용량을 넘어서 확장 할 수 있는 기능 제공.
- RDS Read Replica는 고가용성 솔루션이 아닌, 확장성 솔루

#### **Simple Storage Service - S3**
---
- S3는 내구성이 뛰어난 동시에 FT를 제공하며, Redundant Object Storage 제공.
- S3는 S3 Region의 여러 시설에서 여러 장치에 중복으로 Object 저장.
- S3는 이미지, 비디오 및 기타 Static한 미디어와 같이 변경에 빈번하지 않은 객체를 위한 스토리지
- S3는 CloudFront 서비스와 상호 작용해 이와 같은 리소스의 Edge Caching 및 Streaming 지원

#### **CloudFront**
---
- CloudFront를 사용하면 엣지 로케이션의 글로벌 네트워크를 사용하여 동적, 정적 및 스트리밍 컨텐츠를 포함한 웹 서비스를 제공 할 수 있음.
- 컨텐츠 요청이 요청으로부터 가장 가까운 위치로 자동 라우팅.
- CloudFront는 S3 및 EC2와 같은 AWS 서비스와 함꼐 작동하도록 최적화됨.
- CloudFront는 AWS 이외의 origin 서버와도 Transparent하게 동작.

#### **Simple Queue Service - SQS**
---
- SQS는 Fault-Tolerant한 애플리케이션의 중추 역할을 할 수 있는 매우 안정적인 분산 메시징 시스템
- SQS는 모든 메시지를 적어도 한번 이상 전달하도록 설계됨.
- 큐로 전송되는 메시지는 최대 4일 동안(14일까지 연장 가능), 또는 애플리케이션에서 읽거나 삭제 할 떄까지 보존.
- SQS는 Visibility timeout이라는 configure 가능한 시간 간격을 사용해, 한 번에 하나의 요청만 처리함(다른 요청들은 폴링해서 처리)
- 큐에 쌓인 메시지 수가 증가하기 시작하거나 메시지를 처리하는 데 걸리는 평균 시간이 너무 길면 추가 EC2 인스턴스를 추가하여 워커를 상향 조정 할 수 있음.

#### **Route 53**
---
- Route 53은 가용성과 확장성이 뛰어난 DNS 웹서비스.
- Domain에 대한 쿼리는 가장 가까운 DNS 서버로 자동 라우팅.
- Route 53은 zone apex record는 물론 Domain Name(e.g. www.example.com)에 대한 요청을 ELB로 resolve.
