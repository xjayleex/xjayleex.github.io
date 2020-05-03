---
layout: post
comment: false
title: Kubernetes Cluster에 HDFS 설치
date: 2020-03-17
category: "Kubernetes", "Hadoop"
tags: [Kubernetes, Hadoop]
author: Jaehyun Lee
---

##### HDFS Web App 포트 열기

---

- Master와 Worker에 

  {% highlight bash %} $ kubectl expose service master-svc --port=[namenode.http.address] --external-ip [node-ip] --name=hdfs-webapp {% endhighlight %}


