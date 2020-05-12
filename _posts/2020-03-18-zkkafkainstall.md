---
layout: post
comment: false
title: Zookeeper, Kafka 설치 & systemd Service 등록
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

#### Zookeeper & Kafka systemd Service로 등록하기 
---
Zookeeper와 Kafka의 호스트 머신이 재시작되었을 때를 대비해, systemdService로 등록해줍니다.

{% highlight bash %}
$ vim /etc/systemd/system/zookeeper-server.service
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

{% highlight bash %}
$ vim /etc/systemd/system/kafka-server.service
{% endhighlight %}
{% highlight bash %}
[Unit]
Description=kafka-server
After=network.target
RequiresMountsFor=/data/hdd

[Service]
Type=forking
User=jay
Group=jay
SyslogIdentifier=kafka-server
WorkingDirectory=/
TimeoutStartSec=10min
ExecStart=/data/hdd/softwares/kafka_2.12-2.5.0/bin/kafka-server-start.sh -daemon /data/hdd/softwares/kafka_2.12-2.5.0/config/server.properties
ExecStop=/data/hdd/softwares/kafka_2.12-2.5.0/bin/kafka-server-stop.sh

[Install]
WantedBy=multi-user.target
{% endhighlight %}
