---
layout: post
comment: true
title: Kubernetes란 무엇인가 (NCP 리뷰)
date: 2019-11-12
category: "Kubernetes"
tags: [Kubernetes]
author: Jaehyun Lee
---

#### VM vs Container

---



- virtualization은 단일 시스템에서 여러 os 동시에 수행

- 컨테이너는 동일한 os 커널 공유. 시스템의 나머지 부분으로 프로세스 격리

- 프로세스와 그에 따른 환경을 격리해 오버헤드가 적음.

####Docker Networking

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
- East - West Communication : **Linking**
  - 컨테이너가 PM 이나 VM에 떠있든 하나의 프로세스로 동작하며, 어떠한 외부 요인에 의해 쉽게 죽을 수 있음
  - 이러한 Container Lifecycle 특성상 IP를 이용한 통신의 한계를 보완하기 위한 컨셉
  - Container hosts 파일을 이용, 도메인을 이용한 통신
  - 이 컨셉도 동일 호스트, 동일 brdige 내 Container 간에서만 사용 가능하다는 한계.

- East - West Communication : **Custom network**
  - 동일 Network 바인드 Container 간 사설 통신 가능
  - 동일 호스트, 다른 Network 바인드 Container 간 통신 불가
- 다수 호스트 Container 간 통신은 어떻게 할까 ?
  - [Overlay Network add-on] 도입 필요(OVS, flannel, Calico, weave, etc)
  - 이러한 컨셉을 K8S가 지원
- East - West Communication : **Custom network**
  - Network Overlay add-on  별 특징이 있으므로, 상황에 맞는 적합한 add-on 선택 필요