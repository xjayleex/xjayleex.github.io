---
layout: post
comment: false
title: Kubernetes Cluster에 HDFS 설치
date: 2020-03-17
category: "Kubernetes"
tags: [Kubernetes]
author: Jaehyun Lee
---

#### HDFS Web App 포트 열기
---
- Namenode와 Datanode 프로세스가 정상적으로 올라왔다면, Web UI에 접근할 수 있도록 namenode.http.address Port를 노출시켜줬습니다.

  {% highlight bash %} $ kubectl expose service master-svc \ 
  --port=[namenode.http.address] --external-ip [node-ip] --name=hdfs-webapp {% endhighlight %}


#### Kubernetes Reset Scripts

#### Change NIC Using in Kubernetes & Flannel 
---
Kubernetes 클러스터에 HDFS를 설치한 후, 파일 I/O 테스트를 해보니 성능이 만족스럽지 못했습니다. 그도 그럴 수 밖에 없는게 서버들이
1G 스위치에 물려있었기 때문에... 10G 스위치로 다시 구성해야겠단 생각이 들었습니다. 호스트 OS를 다 재설치한 후라 10G 인터페이스가
올라와 있지 않았고, 네트워크 드라이버부터 다시 설치했습니다. (생각보다 10G 네트워크 드라이버 설치 방법이 온라인 상에 많지 않았습니다.
혹시나해서, 이 곳에 정리해두었습니다.)
{% highlight bash %}
$ ifconfig
enp0s25: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 2XX.XXX.XXX.XXX  netmask 255.255.254.0  broadcast 2XX.XXX.XXX.XXX
		......

p1p1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.0.9  netmask 255.0.0.0  broadcast 10.255.255.255
		.......
{% endhighlight %}
제 서버의 네트워크 상황입니다. 1G NIC가 enp0s25 인터페이스로 디폴트 인터페이스로 잡혀있고, p4p1 인터페이스가 10G NIC 입니다.
flannel은 설치시에 --iface 옵션으로 사용할 NIC를 명시하지 않으면, 디폴트 인터페이스를 사용하도록 yaml 파일이 작성되어있었고,
이 부분을 수정했습니다.

{% highlight yaml %}
      containers:
      - name: kube-flannel
        image: quay.io/coreos/flannel:v0.11.0-amd64
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        - --iface=p4p1
{% endhighlight %}

기존 flannel 설치시에 사용했던 yaml 파일로 기존의 flannel 리소스를 제거해주고, 변경한 옵션으로 flannel을 다시 실행해줍니다.

