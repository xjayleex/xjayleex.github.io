---
layout: post
comment: true
title: K8S란 무엇인가 리뷰
date: 2019-11-12
category: "Kubernetes"
tags: [Kubernetes]
author: Jaehyun Lee
---

# VM vs Container

---



- virtualization은 단일 시스템에서 여러 os 동시에 수행

- 컨테이너는 동일한 os 커널 공유. 시스템의 나머지 부분으로 프로세스 격리

- 프로세스와 그에 따른 환경을 격리해 오버헤드가 적음.

# Docker Networking

---



- docker0라는 linux bridge 디바이스가 생성됨. (brctl show)

- 동일 호스트에서 생성되는 모든 컨테이너들은 같은 docker0에 바인딩
- 각 컨테이너는 외부 또는 다른 컨테이너와 통신을 위해 linux bridge로 통신
- 컨테이너와 linux bridge 간 veth pair 생성
- 각 컨테이너는 의도한 포트만 expose해서 통신
- docker0
  - 동일 호스트, 동일 Network Container의 게이트웨이
  - K8S 클러스터 구성 시에는 CNI0가 docker0 대체
- East - West Communication
  - 동일 호스트 Container 간 통신