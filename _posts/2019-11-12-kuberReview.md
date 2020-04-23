---
layout: post
comment: false
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

  

#### Docker Networking

---



- docker0라는 linux bridge 디바이스가 생성됨. (brctl show)

- 동일 호스트에서 생성되는 모든 컨테이너들은 같은 docker0에 바인딩

- 각 컨테이너는 외부 또는 다른 컨테이너와 통신을 위해 linux bridge로 통신

- 컨테이너와 linux bridge 간 veth pair 생성

- 각 컨테이너는 의도한 포트만 expose해서 통신

- docker0

  동일 호스트, 동일 Network Container의 게이트웨이

  K8S 클러스터 구성 시에는 CNI0가 docker0 대체

- East - West Communication

  동일 호스트 Container 간 통신

- East - West Communication : **Linking**

  컨테이너가 PM 이나 VM에 떠있든 하나의 프로세스로 동작하며, 어떠한 외부 요인에 의해 쉽게 죽을 수 있음

  이러한 Container Lifecycle 특성상 IP를 이용한 통신의 한계를 보완하기 위한 컨셉

  Container hosts 파일을 이용, 도메인을 이용한 통신

  이 컨셉도 동일 호스트, 동일 brdige 내 Container 간에서만 사용 가능하다는 한계.

- East - West Communication : **Custom network**

  동일 Network 바인드 Container 간 사설 통신 가능

  동일 호스트, 다른 Network 바인드 Container 간 통신 불가

- 다수 호스트 Container 간 통신은 어떻게 할까 ?

  Overlay Network add-on 도입 필요(OVS, flannel, Calico, weave, etc)

  이러한 컨셉을 K8S가 지원

  Network Overlay add-on  별 특징이 있으므로, 상황에 맞는 적합한 add-on 선택 필요

- South - North Communication

  Container expose : -p / -P

  {% highlight bash %} $ docker run -d -p 8080:80 --name test nginx {% endhighlight %}

  Kubernetes의 경우는 Service 오브젝트를 통해서 외부에 Container 오픈 : Load Balancer, NodePort 

  Docker 호스트의 iptables를 이용한 DNAT / SNAT 을 통해서 외부와 통신

#### Kubernetes ?

---

- Multi 호스트에 배포된 컨테이너들을 자동으로 관리할 수 있도록하는 매니지먼트 툴
- Master - Work (1:N or N:N)

#### Matster Components

---

- Kubernetes Master

  Kubernetes Cluster에서 컨테이너의 관리 및 배포를 관리하는 액세스 제어 플레인.

  클러스터 복제패턴에 따라 마스터 수는 1개 이상임.

- API Server

  Kubernetes API를 노출하는 컴포넌트로, Kubernetes 오브젝트 관리/제어를 위한 Frontend

- Scheduler

  Node가 배정되지 않은 새로 생성된 Pods를 감지하고 그것이 구동될 Node를 선택함

- Controller-Manager : 4개의 컨트롤러는 논리적으로는 개별 프로세스이지만 복잡성을 낮추기 위해 단일 바이너리로 컴파일

  Node Controller : 노드가 다운되었을 때 통지와 대응

  Replication Controller : 모든 Replication Controller Object에 대해 알맞는 수의 Pods를 유지

  Service Controller : 새로운 네임스페이스에 대한 기본 계정과 API 접근 토큰 생성

- etcd

  모든 클러스터 데이터를 담는 Key-value 저장소, Replicaset, Controller, Scheduler, Kubelet 등은 etcd에 접근하기 위해 API Server를 통해 접근.

#### Worker Node Component

---

- Kubernetes Worker Node : 동작죽인 Pods를 유지, Kubernetes Runtime 환경 제공

- Kublet

  각 노드에 구동되는 Agent로 Kubernetes Master와 통신하며 Pod Spec에 기술된 컨테이너들이 정상적으로 작동되도록 함.

- Kube-proxy

  호스트 상에서 네트워크 규칙을 유지하고 연결에 대한 포워딩을 수행, 쿠버네트스 서비스 추상화를 가능하도록 함.

- Container Runtime(Docker, Containered, etc.)

- Plugin Network

  쿠버네티스 기본 네트워크인 kubenet은 기능 제약이 있어 사용자 요구사항에 따라 별더의 CNI 사용

- DNS

#### Why Container Orchestration tool ?

---

- Overlay Network add-on을 통한 멀티호스트 컨테이너들간의 통신

- 추가 컨테이너는 어디에 넣어 ?

  호스트 리소스와 Affinity  설정 등 여러 요소 기준으로 스케줄링 노드 선정 가능

- 컨테이너 Failure Resistance ?

  Replication Controller(Replicaset) : 클러스터 내에서 Pods를 어떤 replica로 유지할 것인가 설정.

  Container가 죽으면 자동으로 동일 컨테이너를 띄어줌.

  호스트가 죽으면 포함되어있던 컨테이너들을 다른 호스트에 스케줄링.

- Rolling update : 무중단 서비스 지원

#### Kubernetes 기능 정리

---

- Automatic Binpacking

  Worker Node의 가용성을 유지하면서 보유한 리소스를 충분히 활용할 수 있도록 스스로 스케줄링하며 컨테이너를 배치

- Storage Orchestration

  로컬 저장소를 선택하거나 NFS, SCSI 등과 같은 공유 네트워크 스토리지를 컨테이너에 할당, 마운트하여 사용 가능

- Secret & Config Management

  App 연동 및 접근 제어를 위한 보안 키, 설정 내역을 컨테이너 이미지의 변경 없이 업데이트 할 수 있고, 외부로 노출하지 않고 사용 가능.

- Horizontal Scailing

  CPU 사용률과 같은 메트릭을 기반으로 Pod의 Deployments, Replicaset을 스케줄링하여 수평적으로 확장 가능

- Service Discovery & Load Balancing

  컨테이너에 IP 주소를 자동으로 할당하고 클러스터 내 트래픽을 로드 밸런싱 할 수 있는 컨테이너 세트에 단일 DNS 이름을 할당

- Self Healing

  실패한 컨테이너를 자동으로 다시 시작하고, 사용자가 정의한 헬스체크에 응답이 없는 컨테이너를 종료함. 워커 노드 장애 시 사용 가능한 다른 워커 노드에 컨테이너를 다시 기동함.

- Batch Execution

  컨테이너 기반의 서비스 관리 뿐 아니라 배치를 관리할 수 있으므로 원하는 경우 실패한 컨테이너 대체 가능

- Automatic Rollbacks & Rollouts

  컨테이너의 응용 프로그램이나 구성에 대한 변경 사항을 점진적으로 업데이트하고 문제 발생 시 자동으로 롤백.



