---
layout: post
comment: false
title: AWS Cloud Migration 6R
date: 2020-07-17
category: "AWS"
tags: [AWS]
author: Jaehyun Lee
---
### **AWS Cloud Migration**
클라우드로 전환하는 주요 이유는...
- Operational Cost - 인프라, 수요에 대한 적절한 공급 능력, Cost에 기반한 탄력적인 사용 및 투명성에 대한 단위 가격
- Workforce Productivity - 요구에 따른 빠른 다양한 서비스의 Start-up 가용성
- Cost Avoidance - 하드웨어에 대한 지속적인 유지 보수 비용을 제거
- Operational Resilience - 운영에 대한 회복탄력성을 높여 조직적 리스크 제거
- Business Agility - 시장 상황에 민첩하게 대응

#### **Cloud Stage of Adoption (클라우드 도입 단계)**
![Image](/assets/images/aws/cloud-stages-of-adoption.png){:style=" width:     80%; margin: 0 auto; display: block;"}

##### Project Phase
- 클라우드의 이점과 클라우드에 친숙해지기 위한 프로젝트 실행 단계

##### Foundation
- 경험한 클라우드의 이점을 바탕으로, 클라우드 적용에 대한 기반 마련
- Landing Zone(모범 사례에 따른 안전한 다중 계정 AWS 환경을 빠르게 설정할 수 있도록하는 솔루션), CCoE(Cloud Center of Exellence),운영 모델 생성 및 보안 및 규정 준수 준비가 포함된다.

##### Migration
- 많은 IT Portfolio에 클라우드 적용을 확장함에 따라 미션 크리티컬한 애플리케이션이나 전체 데이터센터를 마이그레이션.

##### Revention
- 이제 클라우드 환경에서의 작업이 완료되었으므로, 출시 시간을 단축시키고, 혁신에 대한 관심을 높여 AWS의 유연성과 기능을 활용해 비즈니스 혁심 및 재창조에 집충.

#### **Migration Process**
![Image](/assets/images/aws/migration_process.png){:style=" width:     80%; margin: 0 auto; display: block;"}

##### **Phase 1: Migration Prepartion and Business Planning**
- 적절한 목표를 결정하고, 클라우드 이점에 대한 아이디어 수립
- 기반이되는 경험들로부터 시작해 마이그레이션을 위한 예비 비즈니스 사례를 개발한다. 기존 애플리케이션의 age와 아키텍처 및 제약 조건를 고려해 목표 설정

##### **Phase 2: Portfolio Discovery and Planning
- IT Portfolio(애플리케이션 간의 디펜던시)를 이해하고, 어떠한 유형의 마이그레이션 전력이 비즈니스 케이스에 따른 목표를 만족시킬 수 있을지 고려하기.
- Portfolio Discovery 및 마이그레이션 접근 방식을 통해 전체 비즈니스 사례를 구축할 수 있다.

##### **Phase 3 & Phase 4 : Designing, Migrating and Validating Application**
- Portfolio 레벨에서 개별 애플리케이션 레벨로 포커스를 옮겨 각 애플리케이션을 설계, 마이그레이션 및 유효성 검증 하기
- 각 애플리케이션은 6가지 일반적인 애플리케이션 전략 중 하나에 따라 설계, 마이그레이션 및 검증된다.(6R)
- 여러 애플리케이션을 마이그레이션하고, 조직이 뒤처 질 수 있는 계획을 세우는 데 기본 경험이 있으면 마이그레이션을 가속화하고 규모적 목표를 달성해야한다.
- AWS는 On-premise에서 AWS로 애플리케이션 및 데이터를 이동하는 데 도움이 되는 마이그레이션 서비스를 제공하고 있다. (AWS Server Migration Service(SMS), AWS Database Miration Service(DMS)
	
##### **Operate**
- 애플리케이션을 마이그레이션 한후, 새로운 기반에 대해 반복하고, 오래된 시스템을 없애고, 지속적으로 최신 운영 모델을 위해 반복한다.
- 운영 모델은 더 많은 애플리케이션을 마이그레이션 할 때 지속적으로 개선되는 인력, 프로세스, 기술의 총 집합이 된다.

#### **Application Migration Strategies**
- 마이그레이션 전략은 비즈니스 및 기술 요구 사항을 고려해 어떤 조건이 있는지와 IT Portfolio에 어던 것이 적합한 것인지에 따라 달라진다.
- 다음은 Gartner가 2011년에 요약한 "5R"을 바탕으로 구축된 "6R" 전략이다.

![Image](/assets/images/aws/6r.png){:style=" width:     80%; margin: 0 auto; display: block;"}

##### **1. Rehost ("lift and shift")**
- 클라우드로 애플리케이션을 그대로 옮기기.
- 비즈니스 Case에 맞게 마이그레이션 및 확장을 신속하게 구현할 수 있음.
- 조직이 이미 클라우드 기술을 개발했거나, 애플리케이션이 이미  데이터가 마이그레이션되고 트래픽을 처리하고 있는 경우 애플리케이션을 다시 설계 할 수 있는 더 나은 기회를 제공.
- Rehosting은 AWS Server Migration Service와 같은 도구를 사용해 자동화하거나 수동으로 수행 할 수 있다.

##### **2. Replatform ("lift, tinker and shift")**
- 주요한 변경없이 최적화를 통해 애플리케이션을 클라우드로 이동.
- Replatform은 애플리케이션의 핵심 아키텍처를 변경하지 않고도 실질적인 이점을 얻을 수 있다. (예를 들어 DB에는 RDS를 사용하고, 애플리케이션에 대해서는 Elastic Beanstalk 사용)

##### **3. Repurchase ("drop and shop")**
- 애플리케이션 삭제 및 완전히 새로운 솔루션으로 옮기기.
- Build vs Buy 모델에서, 좀 더 많은 구매가 단기 팀에서는 비용이 더 들 수 있지만, 출시 시간은 더 빠를 수 있다.
- 다른 제품으로 옮기면, 조직이 기존에 사용한 라이선스 모델을 변경하려는 의사로 볼 수 있다.

##### **4. Refactor / Re-Architect**
- 애플리케이션 주요 변경 사항을 클라우드로 이동
- Build vs Buy 모델에서, 더 많은 빌드는 시간이 오래 걸림.
- 기존 환경에서 달성하기 어려운 민첩성과 비즈니스 연속성 개선에 대한 새로운 기능, 규모, 성능을 추가하려는 강력한 비즈니스 니즈에 의해 주도.

##### **5. Retire**
- 더 이상 필요하지 않은 애플리케이션 내리기.
- 더 이상 유용하지 않은 IT 자산을 확인하면, 비즈니스 Case를 개선하고, 많이 사용되는 리소스를 유지하는 데, 포커스를 집중 할 수 있다.

##### **6. Retain**
- 현재 환경에서 애플리케이션을 그대로 유지.
- 마이그레이션에 대한 준비가 되어 있지않거나 우선 순위가 낮은, 그리고 타이트한 디펜던시를 가진 IT Portfolio 부분들을 그대로 유지한다.
