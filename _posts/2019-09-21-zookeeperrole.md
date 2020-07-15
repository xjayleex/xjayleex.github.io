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

##### Kafka Brokers
- State
Zookeeper는 항상 정기적으로 하트비트를 보내어 Broker가 살아있는지를 확인합니다. 

- Quotas
Producing / Consuming quota에 대한 정보를 저장합니다. 이 설정은 /config/clients나 kafka/bin/kafka-configs.sh 스크립트를 통해 설정될 수 있습니다.

- Replicas
Zookeeper는 카프카의 각 토픽에 대해서 in-sync replica(ISR)를 저장합니다. 그리고 카프카의 leader 노드가 fail하게 되면 주키퍼가 살아있는 follower 노드 중에서 새 leader를 선출합니다.


