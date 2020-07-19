---
layout: post
comment: false
title: AWS Disaster Recovery Whitepaper
date: 2020-07-17
category: "AWS"
tags: [AWS]
author: Jaehyun Lee
---
### **AWS Disaster Recoverty Whitepaper**

#### **Disaster Reconvery Overview**
---
- AWS DR Whitepaper는 데이터, 시스템 및 전체 비즈니스 운영에 미치는 영향을 최소화하기 위해 DR 프로세스에 활용할 수 있는 AWS 서비스 및 기능을 중점적으로 설명.
- 최소한의 투자에서 전체 규모의 가용성 및 내결함성에 이르기까지 DR 프로세스를 개선하기 위한 모범 사례를 소개하고, DR 이벤트 중 AWS 서비스를 사용해 비용을 줄이고, 비즈니스 연속성을 보장하는 방법에 대해 설명.
- DR은 재해 대비 및 복구에 관한 것으로, 회사의 비즈니스 연속성 또는 재무에 부정적인 영향을 미치는 모든 이벤트를 재난이라고 할 수 있다. AWS 모범 사례 중 하나는 항상 장애에 대비한 시스템을 설계하는 것이다.

#### **1. Region**
---
- AWS 서비스는 전 세계 여러 Region에서 이용 가능하며, 기본 Region 외에 DR 지역을 적절하게 선택할 수 있다.

#### **2. Storage**
---
##### **Amazon S3**
- 미션 크리티컬 및 기본 데이터 스토리지용으로 설계된 내구성이 뛰어난 스토리지 인프라 제공
- 한 Region 내의 여러 시설에서 여러 장치에 중복으로 Object 저장.

##### **Amazon Glacier**
- 데이터 백업 및 아카이빙을 위해 저렴한 가격으로 스토리지 제공. 내구성은 S3와 비슷.
- Object들은 드문 액세스를 위해 최적화되어 있으며, 3-5 시간 정도가 적당.

##### **Amazon EBS**
- 특정 시점의 데이터 볼륨 스냅샷을 생성하는 기능 제공.
- 스냅샷을 사용해 볼륨을 생성하고 실행중인 인스턴스에 연결할 수 있음.

##### **Amazon Storage Gateway**
- On-premise IT 환경과 AWS의 스토리지 인프라간에 원활하고 매우 강력한 통합을 제공하는 서비스.

##### **AWS Import/Export**
- 인터넷을 우회하는 전송을 위해 휴대용 저장 장치를 사용하여 대량 데이터를 AWS 안팎으로 빠르게 이동. 
- Amazon의 고속 내부 네트워크를 통해 스토리지 장치로 데이터를 직접 전송할 수도 있음

#### **3. Compute**
---
##### **Amazon EC2**
- Resizable한 컴퓨팅 용량을 제공하여 쉽게 생성 및 스케일링 가능
- Preconfigured 된 AMI를 사용하여 EC2 인스턴스 생성.
- EC2 인스턴스는 여러 Availability Zone에서 시작할 수 있으며, 다른 Availability Zone의 장애로부터 격리되도록 설계.

#### **4. Networking**
---
##### **Amazon Route 53**
- 가용성과 확장성이 뛰어난 DNS 웹 서비스
- DR 시나리오를 처리 할 때 효과적인 여러 글로벌 로드 밸런싱 기능을 포함(e.g. DNS 엔드포인트 상태 확인 및 여러 엔드포인트 사이의 Failover 조치 기능)

##### **Elastic IP**
- IP 주소를 프로래머블하게 다시 매핑하여 인스턴스 또는 Availability Zone 장애를 마스킹 할 수 있음.
- IP 주소는 동적인 클라우드 컴퓨팅을 위해 설계된 고정 IP 주소.

##### **ELB(Elastic Load Balancing)**
- Health Check 및 Incoming 애플리케이션 트래픽을 여러 EC2 인스턴스에 자동으로 분배.

##### **Amazon Virtual Private Cloud(Amazon VPC)**
- 정의된 가상 네트워크에서 리소스를 시작할 수 있는 AWS 클라우드의 비공개로 격리된 섹션을 프로비저닝.

##### **Amazon Direct Connect**
- On-premise 환경에서 AWS로 전용 네트워크 연결을 쉽게 설정할 수 있음.

#### **5. Databases**
---
- RDS, DynamoDB, Redshiftㅡㄴ 쉽게 스케일링 가능한 완전 관리형 RDBMS, NoSQL 및 데이터 웨어하우스 솔루션으로 제공됨.
- DynamoDB는 교차 Region 복제 기능 제공.
- RDS는 멀티 Availability Zone 및 Read Replica(읽기 전용 복제본)을 제공하며 한 Region에서 다른 Region으로 데이터를 스냅샷할 수 있음.

#### **6. Deployment Orchestration**
---
##### **CloudFormation**
- 개발자 및 시스템 관리자에게 관련 AWS 리소스 모음을 생성하고, 순서대로 예측 가능한 방식으로 프로비저닝 할 수 있는 쉬운 방법을 제공

##### **Elastic Beanstalk**
- 웹 애플리케이션 및 서비스 배포, 확장을 위한 서비스.

##### **OpsWorks**
- 모든 유형과 크기의 애플리케이션을 쉽게 배포하고 운영 할 수 있도록 도와주는 애플리케이션 매지니먼트 서비스.
- Environment는 일련의 계층으로 정의 될 수 있으며, 각 계층은 애플리케이션의 계층들로 구성될 수 있음.
- 자동 Host 교체 기능이 있으므로, 인스턴스 장애시 자동으로 교체.
- 준비 단계에서 Environment를 템플릿화하고, 복구 단계에서 AWS CloudFormationㅇ과 결합하여 사용 가능.
- 미리 저장된 Config에서 스택들을 빠르게 프로비저닝해 정의된 RTO 지원 가능.

#### **Key factors for Disaster Planning**
---
##### **Recovery Time Objective(RTO)**
- 중단 발생 시점부터, OLA(Opertational Level Agreement)에 정의된대로 비즈니스 프로세스를 서비스 수준으로 복원하는 데 소요되는 시간.
- RTO가 한 시간이고, 재난이 정오에 발생한 경우 DR 프로세스는 한 시간 이내인, 오후 1시 까지 시스템을 허용 가능한 서비스 수준으로 복원해야함.

##### **Recovery Point Objective(RPO)**
- 데이터 손실을 얼마나 감당할 수 있는가에 대한 관점에서 Acceptable한 데이터 손실의 양(시간), 현재로부터 가장 가까운 복원지점까지의 시간 목표.
- 정오에 재난이 발생했고, RPO가 한 시간이면 시스템은 오전 11전에 시스템에 있던 모든 데이터를 복구해야함.

#### **Disaster Reconvery Senarios**
---
- DR 시나리오는 기존 데이터센터의 기본 인프라와 AWS 모두를 활용해 구현 가능.
- 기본 사이트가 AWS 멀티 Region 기능을 사용해 AWS에 구동중인 경우에도 DR 복구 시나리오가 적용됨.
- 아래의 조합과 변형은 항상 가능.

##### **Disaster Reconvery Scenarios Options**
1. Backup & Restore(백업 및 데이터 복원)
2. Pilot Light(최소한의 중요 기능만)
3. Warm Standby(완전히 기능적으로 축소된 버전으로)
4. Multi-Site(Active-Active)

![Image](/assets/images/aws/Disaster-Recovery.png){:    style=" width: 80%; margin: 0 auto; display: block;"}

- DR 시나리오 옵션에서, 왼쪽에서 오른쪽으로 갈수록 RTO와 RPO는 낮아지지만, Cost는 상승.

#### **Backup & Restore**
---

##### **Backup phase**
- 대부분의 기존 환경에서 데이터는 테이프에 백업되고, 중단이나 재난이 발생할 경우, 시스템을 복원하는데 시간이 오래 걸리는 오프 사이트로 정기적으로 전송됨.
![Image](/assets/images/aws/backupoptions.png){:    style=" width: 80%; margin: 0 auto; display: block;"}

1. Amazon S3를 사용하여 데이터를 백업하고, 빠른 복원을 수행 할 수 있으며, 모든 지역에서 사용 가능.
2. AWS Import/Export를 사용해, 인터넷을 우회하여 물리적 스토리지 디바이스에서 AWS에 보내면, Amazon의 고속 인터넷 연결을 사용해 스토리지 디바이스에서 직접 데이터 전송. 
3. Amazon Glacier는 몇 시간의 검섹 시간이 수용 가능한 데이터 보관에 사용될 수 있음.
4. AWS Storage Gateway를 통해 On-premise 데이터 볼륨의 스냅샷(생성된 EBS 볼륨에 사용되는)을 백업을 위해 S3에 Transparent하게 복사 할 수 있음. 백업 솔루션(Gateway-stored 볼륨), 또는 기본 데이터 저장소(Gateway-cached 볼륨)로 사용 가능.
5. AWS Direct Connect를 사용해, On-premise에서 Amazon으로 데이터를 일관되고 빠른 속도로 직접 전송 가능.
6. EBS 볼륨, RDS 데이터베이스 및 Redshift 데이터웨어하우스의 스냅샷을 S3에 저장할 수 있음.

##### **Restore phase**
- 백업된 데이터를 사용해 컴퓨팅 및 데이터베이스 인스턴스를 신속하게 복원 및 생성.
![Image](/assets/images/aws/restore-phase.png){:    style=" width: 80%; margin: 0 auto; display: block;"}

1. 데이터를 AWS에 백업할 적절한 도구 및 방법 선택
2. 데이터에 대한 적절한 Retention Policy 확인
3. 암호화 및 액세스 정책을 포함하여 데이터에 대한 적절한 보안 조치가 준비되었는지 확인.
4. 데이터의 복구와 시스템 복구를 정기적 테스트

#### **Pilot Light**
---
- Pilot Light DR 시나리오 옵션에서는 최소 버전의 환경이 항상 클라우드에서 실행되며, 애플리케이션의 기능적으로 크리티컬한 부분들을 호스팅한다.
- 이 접근 방법에서 Database를 예로 들면,
1. 데이터를 지속적으로 업데이트하고 Replicate 해야하는 데이터베이스와 같이, AWS에서 시스템의 가장 중요한 핵심요소를 구성하고 실행하여 파일럿 기능을 유지.
2. 복구하는 동안, (애플리케이션 및 웹 서버와 같은)Full-scale 프로덕션 환경이 사전 구성된 AMI 및 EBS 볼륨 스냅샷을 활용해 핵심 코어를 중심으로 빠르게 프로비저닝 될 수 있음.
3. 네트워킹의 경우, 트래픽을 여러 인스턴스에 분배하고, 로드밸런서를 가리키는 DNS나 사전 할당된 Elastic IP 주소를 사용할 수 있는 ELB를 사용할 수 있다.

- Preparation phase steps :
1. Data Critical한 데이터를 복제하거나 미러링하도록 EC2 인스턴스 또는 RDS 인스턴스 설정
2. AWS에서 지원되는 모든 사용자 custom 소프트웨어 패키지를 사용할 수 있는지 확인.
3. 빠른 복구가 필요한 주요 서버의 AMI를 생성해 유지
4. 이러한 서버를 정기적으로 실행하고 테스트한 후 소프트웨어 업데이트 및 Config 변경 사항 적용
5. AWS 리소스 프로비저닝 자동화를 고려.
![Image](/assets/images/aws/Preparation-Phase-Pilot-Light-Scenario.png){:    style=" width: 80%; margin: 0 auto; display: block;"}

- Recovery Phase steps :
1. custom AMI로 EC2 인스턴스 기반 애플리케이션 실행.
2. 기존 데이터베이스와 데이터 store 인스턴스의 크기를 조정해, 증가된 트래픽을 처리(e.g. RDS를 사용하면 수직으로 쉽게 확장할 수 있고, EC2 인스턴스는 수평으로 쉽게 확장 할 수 있음.
3. 추가적인 데이터베이스와 데이터 store 인스턴스를 추가하여 데이터 계층에서의 DR 사이트 탄력성을 제공하기. 탄력성을 향상시키기 위해 RDS에 멀티 Region 활성화하기..
4. DNS가 EC2 서버를 가리키도록 변경.
5. non-AMI 기반 시스템들을 자동화된 방식으로 구성하고 설치.
![Image](/assets/images/aws/Recovery-Phase-Pilot-Light-Scenario.png){:    style=" width: 80%; margin: 0 auto; display: block;"}

#### **Warm Standby**
- Warm Standby DR 시나리오에서 비즈니스 Critical한 시스템과 기능적으로 완전히 동일한 환경이 항상 클라우드에서 실행.
- 이 설정은 테스트, 품질 보증 및 내부적으로 사용하기 위한 용도로 사용 가능.
- 재난이 발생하면 시스템을 쉽게 Scale-up, out 방식으로 생산 로드를 처리할 수 있음.

- Preparation Phase steps : 
1. 데이터를 복제하거나 미러링 하도록 EC2 인스턴스를 설정.
2. 더 빠른 프로비저닝을 위해 AMI 생성 및 유지 관리.
3. 최소한의 EC2 인스턴스 또는 AWS 인프라를 사용해 애플리케이션 실행
4. 실제 환경에 맞게 소프트웨어 및 Config 파일을 패치하고 업데이트.
![Image](/assets/images/aws/Preparation-Phase-Warm-Standby.png){:    style=" width: 80%; margin: 0 auto; display: block;"}
