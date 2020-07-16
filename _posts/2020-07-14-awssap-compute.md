---
layout: post
comment: false
title: AWS - Compute
date: 2020-03-17
category: "AWS"
tags: [AWS]
author: Jaehyun Lee
---
## Understand EC2

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

### AMI
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

- 루트 디바이스 크기 제한 16GiB
- 기본적으로, EC2는 인스턴스의 모든 기능을 중지하여 생성 프로세스 중 일관된 상태를 유지하기 위해 AMI를 생성하기 전에 인스턴스의 전원을 차단한다.
- No reboot 옵션을 통해 전원을 차단하지 않도록 할 수 있는데, 파일시스템에 따라 일관성이 보장되지 않을 수 있다.
- EC2는 AMI 생성 프로세스 중에 인스턴스의 루트 볼륨과 인스턴스에 연결된 다른 EBS 볼륨의 스냅샷을 생성한다. 인스턴스에 연결된 볼륨이 암호화되어 있는 경우 Amazon EBS 암호화를 지원하는 인스턴스에서만 새 AMI가 성공적으로 시작된다.
- 루트 디바이스 볼륨 외에 인스턴스에 Instance Store 볼륨이나 EBS 볼륨을 추가하는 경우, 새 AMI에 대한 블록 디바이스 매핑과, 새 AMI에서 자동으로 시작하는 인스턴스에 대한 블록 디바이스 매핑에 이러한 볼륨에 대한 정보가 포함된다. 
- 새 인스턴스에 대한 블록 디바이스 매핑에 지정된 Instance Store 볼륨은 새로운 볼륨이므로 AMI를 생성하는 데 사용된 인스턴스에 대한 Instance Store 볼륨의 데이터가 포함되어 있지 않다(임시 스토리지니까.). -> EBS 볼륨 데이터는 유지된다.
- AMI를 생성하기 전에 볼륨의 스냅샷을 생성하는 것이 더 효율적일 수 있고, 이 경우 작은 증분적 스냅샷만 만드는 것이 권장된다.

##### **AMI Creation (Instance Store-Backed Linux AMI)**
![Image](/assets/images/aws/ami_create_instance_store.png){:style=" width: 80%; margin: 0 auto; display: block;"}
1. 만들려는 AMI#2와 비슷한 AMI#1에서 인스턴스를 시작한다. 인스턴스에 연결해 인스턴스를 사용자 지정할 수 있다.
2. 인스턴스가 올바르게 구성되면, 이 인스턴스를 번들링한다. 
3. 이미지 매니페스트(image.manifest.xml)와 루트 볼륨 템플릿을 포함하는 파일(image.part.xx)로 구성된 번들이 만들어진다.
4. 이 번들을 Amazon S3 버킷으로 업로드하고 AMI를 등록한다.
5. 새 AMI를 사용해 인스턴스를 시작하는 경우, Amazon S3로 업로드한 번들을 사용해 인스턴스용 루트 볼륨이 생성된다.

- 루트 디바이스 크기 제한 10GiB
- 스토리지 공간에 대해 사용자가 삭제할 때까지 빌링됨.
- 루트 디바이스 볼륨 외에 인스턴스에 Instance Store 볼륨이나 EBS 볼륨을 추가하는 경우, 새 AMI에 대한 블록 디바이스 매핑과, 새 AMI에서 자동으로 시작하는 인스턴스에 대한 블록 디바이스 매핑에 이러한 볼륨에 대한 정보가 포함된다. 

##### **EBS 지원 AMI를 통한 암호화 사용**
- Amazon EBS 스냅샷의 지원을 받는 AMI에서는 데이터 볼륨과 루트 볼륨 모두의 스냅샷을 암호화하고 AMI에 연결할 수 있다.
- AMI 사본 이미지를 이용해 암호화되지 않은 스냅샷을 지원하는 AMI에서 암호화된 스냅샷을 지원하는 AMI를 생성할 수 있다. 기본적으로 사본 이미지는 스냅샷의 암호화 상태를 유지한다.
- EBS 볼륨처럼 AMI의 스냅샷을 기본 AWS KMS 고객 마스터키(CMK) 혹은, 지정한 고객 관리형 키로 암호화 할 수 있다. 어느 경우든 선택한 키에 대한 사용권한이 있어야만 가능하다.
- 암호화된 스냅샷이 있는 AMI는 AWS 계정 간에 공유할 수 있다.

##### **AMI Copying**
- EBS-backed, Instance-backed AMI 모두 복사할 수 있다.
- AMI 복사
	- 동일한 AMI가 생성되지만, 고유 식별자로 구별되는 AMI가 생성된다.
	- EBS-backed AMI의 경우, 동일하지만 별개의 대상 스냅샷으로 복사된다.
	- 암호화 상태는 유지된다.
	- 하지만, 시작권한, 사용자 정의 태그, S3 버킷 권한은 새 AMI로 복사되지 않는다. 복사 작업 완료 후, 새로운 시작 권한, 사용자 정의 태그 및 S3 버킷 권한을 적용할 수 있다.
- 대상 AMI에 영향을 미치지 않고 원본 AMI를 변경하거나 다시 등록 가능하다. 반대의 경우도 마찬가지고.
- 적절한 권한으로 소유 또는 공유된 AMI를 복사할 수 있다.(적절한이라면 어떤 것일까?)
- AMI는 리전별로 생성되며 리전 내 또는 리전간에 복사 할 수 있어, 일관된 글로벌 배포를 지원하므로 확장성과 가용성이 뛰어는 애플리케이션을 구축할 수 있다.
- 암호화된 스냅샷을 사용해 AMI를 복사하고, 복사 과정 중 암호화 상태를 변경할 수 있다.
- CopyImage 작업 중 암호화는 EBS-backed AMI에만 적용된다. Instance Store-backed AMI에서는 스냅샷을 사용하지 않기 때문에, AMI 사본을 사용해 암호화 상태를 변경할 수 없다.
- 암호화되지 않은 스냅샷을 복사해 암호화된 스냅샷을 생성할 수 있지만, 암호화된 스냅샷을 복사해 암호화되지 않은 스냅샷을 생성할 수는 없다.

##### **Deregistering AMI**
- 생성된 AMI에는 빌링이 됨.
- AMI를 등록 취소해도 EBS 스냅샷과 S#버킷의 번들은 삭제되지 않으며, 별도로 제거해야 한다.
- AMI 등록 취소는 이미 생성된 인스턴스에는 영향을 미치지 않는다.
- EBS-Backed AMI를 등록 취소 후, EBS 스냅샷을 삭제한다.
- Instance Store-Backed AMI를 등록 취소 후, S3 버킷에 있는 번들을 삭제한다.

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

### EC2 Instance Types
---
- EC2 인스턴스 타입은 인스턴스에 사용되는 Host 하드웨어를 결정한다.
- 각 인스턴스 유형은 서로 다른 컴퓨팅, 메모리, 스토리지 용량을 제공하는데, 이 용량에 따라 서로 다른 Instance Family로 분류됨.
- EC2에서는 실제로 사용되는 하드웨어에 관계 없이 각 인스턴스에 일정하고 예측 가능한 CPU 용량 제공.
- EC2는 Host에 있는 CPU, Memory, Instance Storage 등의 일부 리소스를 특정 인스턴스에 전용으로 할당.
- 하지만, Network, 디스크 하위 시스템과 같은 기타 리소스는 여러 인스턴스와 공유한다. Host의 각 인스턴스가 이러한 공유 리소스 중 하나를 최대한 많이 사용하려고 할 경우 해당 리소스는 각 인스턴스에 고르게 분배된다. 하지만 리소스 사용률이 저조한 경우에는 리소스에 여유가 있는 한 특정 인스턴스가 해당 리소스를 더 많이 소비할 수 있다.

##### **EC2 Instance Type Selection 기준**
---
- 일부 인스턴스 유형은 HVM 가상화 유형만 지원하고 이외의 인스턴스 유형은 PV 및 HVM 가상화 유형을 모두 지원한다. 하지만 기반 하드웨어 활용의 이점을 위해 HVM 사용을 권장한다.
- VPC에서 모든 인스턴스를 사용할 수 있지만, EC2-Classic에서는 사용할 수 없는 인스턴스들이 많다. AWS는 향상된 네트워킹, Multiple IP 주소, 보다 정밀한 보안 제어 등을 활용할 수 있도록 VPC를 사용하길 권장한다.
- 일부 인스턴스 유형은 EBS 볼륨만 지원하는 반면 다른 인스턴스 유형은 EBS 및 Instance Store 볼륨 모두를 지원한다. Instance Store 볼륨을 지원하는 일부 인스턴스는 SSD를 사용해 높은 Random I/O 성능을 제공한다.
- 일부 인스턴스 유형을 EBS 최적화 인스턴스로 시작하면 Amazon EBS I/O 전용 용량을 더 많이 확보할 수 있다. EBS에 최적화된 인스턴스를 사용하면 Amazon EBS I/O와 인스턴스의 다른 네트워크 간의 경합을 제거하여 EBS 볼륨에 대해 일관되게 우수한 성능을 제공할 수 있다.
- 일부 인스턴스는 고성능 컴퓨팅에 최적화되기 위해 배치 그룹안에서 구동 될 수 있다.
- 향상된 네트워킹이 지원되는 인스턴스에서 이를 활성화하면, 지연 시간을 줄이고 네트워크 Jitter를 낮추며 PPS(Packet Per Second) 성능을 높일 수 있다.

##### **EBS-최적화 인스턴스**
---
- EBS 최적화 인스턴스는 최적화된 구성 스택을 사용하며, EBS I/O를 위한 추가 전용 용량을 제공한다. 이러한 최적화를 통해 인스턴스에서 EBS I/O와 기타 트래픽 간의 경합이 최소화되어 EBS 볼륨의 성능이 극대화된다.
- EBS 최적화 인스턴스는, optional하게 500 Mbps ~ 4000 Mbps의 대역폭을 제공한다.
- EBS 최적화 인스터스에 연결된 경우 General Purpose SSD 볼륨은 99%의 시간 동안 기본 성능 및 버스트 성능을 제공하도록 설계되며, Provisioned IOPS(SSD) 볼륨은 99.9%의ㅅ 시간 동안 프로비저닝 성능을 제공하도록 설계된다. HDD 및 Cold HDD는 모두 99%의 기간 동안 90& 버스트 처리량에 대해 성능 일관성을 보장한다.
- 기본적으로 EBS에 최적화되지 않은 인스턴스에 대해 EBS 최적화를 활성화 할 수 있다.

##### **Instance Types**
---
![Image](/assets/images/aws/aws-instance-types.png){:style=" width: 80%; margin: 0 auto; display: block;"}

##### **T2 Instances (General Purpose)**
- T2 인스턴스는 적절한 기준 성능을 제공하고 워크로드에 필요한 성능을 향상시킬 수 있는 버스트 기능이 제공된다.
- 전체 CPU를 일관되게 사용하지 않지만, 때때로 Bursty한 작업을 해야하는 워크로드를 위해 사용된다.
- 적합한 워크로드 : 웹 서버, 개발 환경, 원격 데스크톱 및 소규모 데이터베이스
- Requirements
	- HVM AMI로만 시작할 수 있다.
	- VPC로만 시작할 수 있으며, EC2-Classic 플랫폼에서 지원되지 않는다.
	- On-Demand와 예약 인스턴스로 사용할 수 있지만, Spot 인스턴스는 불가.
	- 기본적으로, 20 (soft limit) T2 인스턴스를 동시에 구동 가능.
	- Dedicated instance에서 구동될 수 없음.

##### **CPU Credits**
---
- 성능 순간 확장 가능 인스턴스는 기본 수준의 CPU 사용률을 제공하면서 기본 수준 이상으로 CPU 사용률을 버스트하는 기능을 제공하는데, 기준 사용률과 버스트 기능은 CPU 크레딧에 의해 좌우된다.
- CPU 크레딧은 1분 동안 전체 CPU 코어의 100% 사용률을 제공한다. 예를 들어 CPU 크레딧 하나는 2분 동안 25% 사용률로 실행되는 vCPU 2개에 해당 될 수 있다.
- T2 인스턴스는 Start up 성능을 위해 시작 크레딧을 받는데, 이 시작 크레딧은  CPU 크레딧 밸런드 한도에 포함되지 않아 만료되지 않지만, 인스턴스가 CPU 크레딧을 사용할 때, 맨 처음으로 사용된다.
- 각 T2 인스턴스는 인스턴스 크기에 따라 특정 비율의 시간당 CPU 크레딧을 지속적으로 획득한다.(e.g. t2.nano earns 3/hour while a t2.large earns 36/hour)

