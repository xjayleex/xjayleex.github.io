---
layout: post
comment: false
title: Protocol Buffer
date: 2020-08-14
category: "back-end"
tags: [protobuf,GRPC]
update: 2020-08-14
author: Jaehyun Lee
---
![Image](/assets/images/aws/protobuf.png){:style="     width: 80%; margin: 0 auto; display: block;"}

> ### Contents
[**Introduction**](#introduction)  
[**Encoding**](#encoding)  

#### Intoduction
---
파일로 저장하거나 네트워크를 통해 전송할 데이터가 있다고 해보자. 대상 데이터를 다루면서, 여러 개선 단계가 있을 것이다.
1. 처음으로 생각할 수 있는건 프로그래밍 언어의 built-in 직렬화 라이브러리! 자바에는 `serialization`, 파이썬에는 `pickle` 등이 있다.
2. 그러다가 이러한 방식이 그 프로그래밍 언어에서 국한되어 더 이상 확장되지 않는다는 것을 느끼게되면 `JSON`이나 `XML`과 같이 언어에 구애받지 않는 형식으로 바꾸게 될 것이다.
3. JSON이 너무 장황한 점, 느린 파싱 성능, 정수와 플로팅 포인트를 구분하지 않는다는 점 등등에 짜증이 날 떄 쯤이면 바이너리 형태의 프로토콜을 고민할 수 있다.
4. 하지만, 바이너리 JSON의 형태도 그렇게 간결하진 않아 여기에도 불만족스러울 수 있다. 왜냐면 여기에서도 필드 이름을 계속해서 저장하니까...
5. 이런 고민 단계를 거치다가 보면 `Protocol Buffer`나 Avro, Thrift 등의 도입을 고려하게 된다.


`Protocol Buffer`(이하 protobuf)는 네트워크나 파일로 저장 될 수 있는 데이터를 직렬화하는 방법이다. 기존의 JSON과 XML도 데이터 직렬화에 사용되는 프로토콜이지만, 여러 다른 플랫폼 기반의 

#### Schema
---
JSON 포맷으로  다음과 같은 데이터가 있다고 하자.
```json
{
	"userName": "Martin",
    "favouriteNumber": 1337,
    "interests": ["daydreaming", "hacking"]
}
```
![Image](/assets/images/protobuf.png){:style="     width: 80%; margin: 0 auto; display: block;"}
