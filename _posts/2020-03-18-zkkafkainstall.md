---
layout: post
comment: false
title: Zookeeper, Kafka 설치 & systemd Service 등롣
date: 2020-03-18
category: "Kafka"
tags: [Kafka]
author: Jaehyun Lee
---

#### Zookeeper 설치
---
- Namenode와 Datanode 프로세스가 정상적으로 올라왔다면, Web UI에 접근할 수 있도록 namenode.http.address Port를 노출시켜줬습니다.

  {% highlight bash %} $ kubectl expose service master-svc \ 
  --port=[namenode.http.address] --external-ip [node-ip] --name=hdfs-webapp {% endhighlight %}


#### Kafka 설치

#### Change NIC Using in Kubernetes & Flannel 
---

{% highlight bash %}
# sudo vim /etc/systemd/system/zookeeper-server.service
{% endhighlight %}
{% highlight bash %}
[Unit]
Description=zookeeper-server
After=network.target
RequiresMountsFor=/data/hdd

[Service]
Type=forking
User=jay
Group=jay
SyslogIdentifier=zookeeper-server
WorkingDirectory=/
TimeoutStartSec=10min
PIDFILE="/data/ssd/zookeeper_data/kafka_data/zookeeper_server.pid"
ExecStart=/data/hdd/softwares/zookeeper/bin/zkServer.sh start
ExecStop=/data/hdd/softwares/zookeeper/bin/zkServer.sh stop

[Install]
WantedBy=multi-user.target
{% endhighlight %}

