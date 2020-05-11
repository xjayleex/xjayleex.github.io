---
layout: post
comment: false
title: Kubernetes Service
date: 2020-03-17
category: "Kubernetes"
tags: [Kubernetes]
author: Jaehyun Lee
---

#### Kubernetes Service
---
- Namenode와 Datanode 프로세스가 정상적으로 올라왔다면, Web UI에 접근할 수 있도록 namenode.http.address Port를 노출시켜줬습니다.

  {% highlight bash %} $ kubectl expose service master-svc \ 
  --port=[namenode.http.address] --external-ip [node-ip] --name=hdfs-webapp {% endhighlight %}
