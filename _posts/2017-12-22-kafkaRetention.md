---
layout: post
comment: true
title: Kafka Retention Policy
date: 2017-12-22
category: "Kafka"
tags: [Kafka, Retention Policy]
author: Jaehyun Lee
---

|         NAME         | DESCRIPTION                                                  | default | server default property         |
| :------------------: | ------------------------------------------------------------ | ------- | ------------------------------- |
|    cleanup.policy    | "delete" or "compact".  delete는 retention size나 time 도달 시 오래된 세그먼트 버림.  compact는 토픽에 대한 log compaction 세팅 활성화시킴 |         | log.cleanup.policy              |
| delete.retention.ms  | 압축된 토픽에 대한 retention time                            | 24h     | log.cleaner.delete.retention.ms |
| file.delete.delay.ms | 파일시스템으로부터 파일을 삭제하기 전에 기다리는 시간        | 60000   | log.segment.delete.delay.ms     |
|    flush.messages    | fsync flush 하는 max messages                                |         | log.flush.interval.messages     |
|       flush.ms       | fsync flush interval.  그냥 사용하지 않는 것 추천..? OS 플러시 메커니즘에 맡기는게 낫다함. |         | log.flush.interval.ms           |
| **retention.bytes**  | 삭제하기 전에 보관할 파티션 당 로그 수. 기본값은 -1. 토픽별로 설정이 가능하며, 로그의 시간이나 크기제한에 도달하면 삭제됨. |         | log.retention.bytes             |
|   **retention.ms**   | 로그 세그먼트를 보유할 시간을 정의. 기본값은 7h이며 토픽별로 정의 가능. | 7h      | log.retention.{ms..hour}        |


