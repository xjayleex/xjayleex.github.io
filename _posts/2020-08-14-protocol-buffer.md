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
3. JSON의  장황함, 느린 파싱 성능, 정수와 플로팅 포인트를 구분하지 않는다는 점 등등에 짜증이 날 떄 쯤이면 바이너리 형태의 프로토콜을 고민할 수 있다.
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
위의 데이터는 공백을 제거했을 때, 82 바이트 공간을 차지한다. 이를 Protocol Buffer로 나타내면 어떨까?
다음은 위의 JSON 객체를Protocol Buffer 스키마로 나타낸 것이다.
```protobuf
message Person {
	required string user_name			 = 1;
	optional int64	favourite_number	 = 2;
	repeated string	interests			 = 3;
}
```
이를 바이너리로 인코딩하면 다음과 같이 33 바이트 공간만을 차지하게 된다.

![Image](/assets/images/protobuf.png){:style="     width: 80%; margin: 0 auto; display: block;"}
[(*Schema evolution in Avro, Protocol Buffers and Thrift*)](https://martin.kleppmann.com/2012/12/05/schema-evolution-in-avro-protocol-buffers-thrift.html)
위의 그림을 이해해보면 Protocol Buffer의 압축성을 직관적으로 파악할 수 있다. 바이너리화된 데이터의 맨 앞 1바이트는 메타데이터로 두 가지 정보를 가지고 있는데, 앞의 5bit는 `field tag`를 나타내며, 뒤의 3bit는 `field type`을 나타낸다.
그 뒤에 있는 두 번째 바이트는 뒤에서부터 이어질 데이터의 길이를 저장해서 나타낸다.
