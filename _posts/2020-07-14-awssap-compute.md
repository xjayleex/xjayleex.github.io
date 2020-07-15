---
layout: post
comment: false
title: AWS - Compute
date: 2020-03-17
category: "AWS"
tags: [AWS]
author: Jaehyun Lee
---
### Understand EC2



#### **EC2 Feature**
---
- **EC2 Instance**  :  가상 컴퓨팅 환경
- **AMI(Amazon Machine Image)**  :  서버에 필요한 OS와 추가적인 소프트웨어들이 패키지 형태로 구성된 상태로 제공되는 템플릿으로 인스턴스를 쉽게 생성 가능
- **Instance Type**  :  여러가지 구성으로 CPU, Memory, Storage, Network 제공
- **Key Pair**  :  AWS는 Public Key를 저장하고 유저는 Private Key를 안전한 장소에 보관하는 방식으로 인스턴스 로그인 정보를 보호
- **EBS(Amazon Elastic Volume)**을 이용해 영구적 데이터 저장
- **Region & Availability Zones**  :  인스턴스와 Amazone EBS 볼륨 등의 리소스를 다른 물리적 장소에서 액세스 가능
- **Secure Group**을 통해 인스턴스에 연결할 수 있는 프로토콜, 포트, 소스 IP 범위를 지정하는 Firewall 기능
- **EIP(Elastic IP)**  :  동적 클라우드 컴퓨팅을 위한 고정 IPv4 주소
- **Tag**  :  사용자가 생성하여 EC2 리소스에 할당할 수 있는 Metadata
- **VPC(Virtual Priavte Clouds)**  :  AWS 클라우드에서는 논리적으로 격리되어 있지만, 원할 때 마다 사용자 네트워크와 연결할 수 있는 가상 네트워크

#### **Accessing EC2**
---
##### Amazon EC2 Console
- Web 기반 사용자 인터페이스  

##### AWS CLI
- Window, Mac, Linux 지원  

##### AWS Query
- "Action"이라는 쿼리 변수로, GET, POST Request로 HTTP, HTTPS Request 가능  

##### AWS SDK Libraries
- HTTP나 HTTPS Request를 직접 보내는 대신, 각 언어가 제공하는 고유의 API를 사용하도록 라이브러리 제공  
- 이 라이브러리를 통해서 HTTP/HTTPS Request에 암호화된 사인, Request Retrying, Error Response 핸들링 등의 작업을 자동화할 수 있는 기능 제공

#### **AWS EC2 - AMI**
---
- AMI는 인스턴스를 시작하는 데 필요한 정보를 제공
- 동일한 구성의 인스턴스가 여러 개 필요할 때에 한 AMI에서 여러 인스턴스 구동 가능
- VPC 내에서, 다양한 AMI를 사용해 인스턴스들을 구동 가능

##### AMI는 다음을 포함
- 1개 이상의 EBS Snapshot 또는, 인스턴스 저장 지원 AMI의 경우, 인스턴스의 루트 볼륨에 대한 템플릿(e.g. OS, Applcation Server, Apps)
- AMI를 사용해 인스턴스를 시작할 수 있는 AWS 계정을 제어하는 시작 권한
- 시작될 때 인스턴스에 연결할 볼륨을 지정하는 블록 디바이스 매핑

#### **AMI Types**
---
다음 유형을 기준으로 사용할 AMI 선택 가능
##### - Region & Availability Zone
##### - Operating System
##### - Computer Architecture
##### - Launch Permissions 
- AMI 소유자는 시작 권한을 지정하여 가용성 결정
- Public  모든 AWS 계정에 시작 권한 부여
- Explicit  특정 AWS 계정들에 시작 권한 부여
- Private/Implicit  Creator Account Only  

##### - Root Device Storage
- 모든 AMI는 Amazon EBS나 Instance Store를 루트 디바이스 스토리지로 사용

- EBS Backed
- EBS 볼륨은 EC2 인스턴스의 생명주기로부터 독립적으로 유지된다.
- EBS backed 인스턴스는 볼륨 삭제없이 중지될 수 있다.
- EBS 인스턴스는 Delete On Termination 플래그가 활성화되어 있지 않은 이상, 볼륨 데이터를 유지한다. (Default는 True로 설정되어 있음)
- 인스턴스를 사용하기 전에 스냅샷에서 인스턴스를 부팅하는 데 필요한 부분만 검색하면 되므로 EBS Backed 인스턴스는 Instance Store Backed 인스턴스보다 훨씬 빠르게 부팅된다.
- AMI 생성에 CreateImage API  단일 명령/호출을 사용해 AMI 생성이 쉽다.

- Instance Store Backed
- 인스턴스 생명주기에 의존하는 임시 스토리지로 인스턴스가 종료되거나, Instance Store 볼륨이 연결된 EBS Backed 인스턴스가 중지되면, Instance Store가 삭제됨.
- Instance Store 볼륨에는 '중지' 없음.
- 인스턴스가 이용가능한 상태로 만들어지기 전에, S3로 부터 모든 부분들이 끌어져와야되기 때문에 EBS Backed 인스턴스에 비해 부팅 시간이 길다. 
- Instance Store Backed Linux AMI를 생성하려면 Amazon EC2 AMI 도구를 설치해야 한다.

#### **Linux Virtualization Types**
---

Linux Amazon 머신 이미지는 PV(반가상화) 또는 HVM(하드웨어 가상 머신)의 두 가지 유형 가상화를 사용하는데, 주요 차이점은 부팅 방법과 더 나은 성능을 위해서 특수 하드웨어 확장(CPU, 네트워크, 스토리지)을 활용할 수 있는지 여부에 있다. 최상의 성능을 위해서 AWS는 인스턴스를 시작할 때 현재 세대 인스턴스 유형 및 JVM AMI를 사용하는 것을 권장하고 있다.

##### HVM AMI
- HVM AMI는 이미지 루트 블록 디바이스의 마스터 부트 레코드를 실행해서 완벽히 가상화된 하드웨어 및 부트 세트를 함께 제공한다.
- HVM 가상화 타입에서는 Bare Metal에서 구동할 때 처럼 가상 머신에서 운영체제를 수정하지 않고 실행할 수 있다.
- Amazon EC2 호스트 시스템은 게스트 시스템에 제공하는 기본 하드웨어의 일부 또는 모두를 에뮬레이트 한다.
- HVM 게스트는 PV 게스트와 달리 Hardware extension을 이용해 하부의 하드웨어에 빠르게 접근할 수 있다.
- GPU 프로세싱이나 향상된 네트워킹을 이용하려면 HVM AMI가 필요한데, 이에 대한 명령이 통과하기 위해서는 OS는 기본 하드웨어 플랫폼에 접근할 수 있어야 한다. 이를 접근할 수 있도록하는 것이 HVM 가상화.
- 현재 모든 인스턴스 유형이 HVM AMI 지원한다.

##### PV AMI
- PV AMI는 PV-GRUB이라는 특수 부트 로더를 통해서 부팅된다. 이 로더는 부팅 시퀀스 시작 후 유저 이미지의 menu.lst 파일에 지정된 커널을 체인 로드한다.
- PV 게스트는 향상된 네트워킹과 GPU 프로세싱과 같은 Hardware Extension을 활용할 수 없다.
- C1, C3, HS1, M1, M2, M3, T1 등과 같은 전 세대 인스턴스 유형은 PV AMI를 지원하고, 최신 세대 인스턴스 유형은 PV AMI를 지원하지 않는다.

#### **Shared AMI**
---
- **Shared AMI**  :  다른 개발자가 사용할 수 있도록 공유된 AMI
- Amazon에서는 사용자들의 공유된 AMI의 무결성이나 보안성을 보장하지 않으므로, 주의 필요.
- AMI에 시작 권한을 정의해, 공개적으로나, 특정 계쩡에만 공유 할 수 있다.

##### Guidelines for Shared Linux AMI
- Boot Time에 AMI Tool을 업데이트 할 것.
- 루트 사용자에 대한 암호 기반 원격 로그인을 비활성화.
- 로컬 루트 접근 비활성화.
- SSH Host Key Pair 삭제함으로써, "Man-in-the-middle" 공격 가능성 낮추기.
- Public Key 자격 증명 설치
	- EC2에서는 인스턴스 시작 시, 사용자가 Public/Private 키 페어 이름을 설정하는 것을 허용한다.
	- 인스턴스를 시작할 때, 유효한 키페어 이름을 제공해야하며, 퍼블릭 키는 인스턴스 메타데이터에 대한 HTTP 쿼리를 통해 인스턴스에서 사용할 수 있게 된다.
	- 유저는 루트 패스워드 없이 키 페어를 사용해 AMI 인스턴스를 실행할 수 있다.
- sshd DNS 체크 비활성화(Optional)
	- sshd DNS 확인을 비활성화하면 sshd 보안은 저하되지만, DNS 확인이 실패했을 때에도 SSH 로그인이 가능하게 해준다.
- AMI는 계정 ID로 표시되기 때문에 공유 AMI 제공자를 간편하게 확인할 수 있는 방법이 존재하지 않고 있다. 그래서 AMI 설명과 AMI ID를 EC2 forum에 게시하는 것이 권장된다.
- 애초에 민감한 데이터나 소프트웨어를 AMI에 저장하지 않기.

#### **AMI Lifecycle**
---
- AMI를 생성 및 등록한 다음 새로운 인스턴스를 시작하기위해 사용할 수 있다.
- AMI를 동일 리전 또는 다른 리전으로 복사할 수 있다.
- 더 이상 필요 없는 AMI는 등록 취소할 수 있다.

##### **AMI Creation (EBS-Backed Linux AMI)**
![Image](/assets/images/aws/running-instance.png){:style=" width: 80%; margin: 0 auto; display: block;"}

- EBS-Backed Linux AMI는 EBS 인스턴스에서 직접 혹은 EBS 스냅샷에서 생성할 수 있다.

1. 만들려는 AMI#2와 비슷한 AMI#1에서 인스턴스를 시작한다. 인스턴스에 연결해 인스턴스를 사용자 지정할 수 있다. 
2. 인스턴스가 올바르게 구성되면 AMI를 생성하기 전에 인스턴스를 중단하여 데이터 무결성을 확인한다.
3. AMI#2를 생성하거나, EBS 스냅샷을 생성한뒤 스냅샷으로부터 AMI#2를 생성한다.
4. EBS-Backed AMI는 자동으로 등록되고, AMI#2는 새로운 인스턴스를 만들 때 사용될 수 있다.


#### **Amazon Linux 2 and Amazon Linux AMI**
---
Amazon Linux 2와 Linux AMI는 다음과 같은 기능을 제공할 수 있도록 유지 및 관리된다.
- EC2에서 동작하는 어플리케이션의 고성능, 보안, 안정적인 구동 환경
	- 원격 루트 SSH 접속을 기본값으로 제공하지는 않는다.
	- Brute-Force 암호 공격 방지를 위해 암호 인증을 사용할 수 없다.
	- Amazon Linux 인스턴스에 대해 SSH 로그인을 사용하려면 시작 시 인스턴스에 키 페어를 제공해야 한다.
	- SSH 액세스를 허용하도록 인스턴스를 시작하는 데 사용되는 보안 그룹을 설정해야 한다. 기본적으로 SSH를 사용해 원격으로 로그인할 수 있는 계정은 ec2-user뿐이다. (이 계정에는 sudo 권한도 있음)
	- 여러 버전의 MySQL, PostgreSQL, Python, Ruby, Tomcat 및 더 일반적인 패키지에 대한 Repo  제공.
	- 최신 컴포넌트들을 포함하도록 정기적으로 업데이트되며, yum repo에서 실행중인 인스턴스에 설치할 수 있도록 제공된다.

