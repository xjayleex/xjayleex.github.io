---
layout: post
comment: true
title: Role of Zookeeper in Kafka
date: 2020-03-17
category: "Kafka"
tags: [zookeeper]
author: Jaehyun Lee
---

#### What is Apache Zookeeper
- Apache Zookeeper는 분산 어플리케키이션을 위해 Naming Registry, 분산 프레임워크 설정과, 동기화를 지원합니다. 

![Image](/assets/images/zookeeperkafka.png){:style=" width: 80%; margin: 0 auto; display: block;"}

Zookeeper는 하둡이나 HBase와 같은 하둡 에코시스템을 사용할 때, 체계적인 서비스를 구축하도록 도와줍니다.

#### Zookeeper in Kafka
- Zookeeper는 Kafka의 Consumer, Broker에 의해서 사용되는 여러가지 공유 데이터를 저장하는 역할을 수행합니다.
 ![Image](/assets/images/zoorole.png){:style=" width: 80%; marg    in: 0 auto; display: block;"}

