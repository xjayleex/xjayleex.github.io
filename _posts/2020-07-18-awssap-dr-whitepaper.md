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

