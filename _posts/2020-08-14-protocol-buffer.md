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
[**Why Protocol Buffer**](#why-protocol-buffer)  

#### Intoduction
---
Protocol Buffer를 이해하려면 우선 Serialization이 무엇인지 알아야 한다. Wikipedia의 설명을 인용해보면,
> Serialization
"Serialization is the process of translating data structures or object state into a format that can be stored (for example, in a file or memory buffer) or transmitted (for example, across a network connection link) and reconstructed later (possibly in a different computer environment)"

파일로 저장하거나 네트워크를 통해 전송할 데이터가 있다고 해보자. 대상 데이터를 다루면서, 다음과 같이 여러 개선 단계를 거칠 것이다.
1. 처음으로 생각할 수 있는건 프로그래밍 언어의 built-in 직렬화 라이브러리! 자바에는 `serialization`, 파이썬에는 `pickle` 등이 있다.
2. 그러다가 이러한 방식이 그 프로그래밍 언어에서 국한되어, 다른 플랫폼과 확장선상에서 더 이상 확장되지 않는다는 것을 느끼게되면 `JSON`이나 `XML`과 같이 언어에 구애받지 않는 포맷으로 바꾸게 될 것이다.
3. JSON의  장황함, 느린 파싱 성능, 정수와 플로팅 포인트를 구분하지 않는다는 점 등등에 짜증이 날 떄 쯤이면 바이너리 형태의 프로토콜을 고민할 수 있다.
4. 하지만, 바이너리 JSON의 형태도 그렇게 간결하진 않아 여기에도 불만족스러울 수 있다. 왜냐면 여기에서도 필드 이름을 계속해서 저장하니까...
5. 이런 고민 단계를 거치다가 보면 **Protocol Buffer**나 Avro, Thrift 등의 도입을 고려하게 된다.


`Protocol Buffer`(이하 protobuf)는 네트워크나 파일로 저장 될 수 있는 데이터를 직렬화하는 방법이다. 기존의 JSON과 XML도 데이터 직렬화에 사용되는 프로토콜이지만, 여러 다른 플랫폼 기반 중심에서 데이터를 전송하는 시나리오에서는 완전히 최적화되어 있지는 않다. 물론 Human-readable 하다는 장점이 있기는하지만, *Space-intensive*하다(이유는 바로 아래 절에서 확인 가능하다). 대규모 트래픽 처리에는 적합하지 않은 것이다. 이게 바로 Protocol Buffer가 만들어진 이유다. 
Protocol Buffer는 데이터 형식에서 수행하는 많은 일들을 제거하고 가능한 빨리 데이터를 직렬화하고 역직렬화하는 기능에만 집중하도록 함으로써 JSON 및 XML 보다 더 빠르게 동작하도록 최적화되어 있다. 또한 전송 데이터를 가능한 작게 만들어서 네트워크 대역폭을 줄인다.
Protocol Buffer에서 직렬화할 데이터 spec은 `.proto` 파일에 기록하는데, 파일에는 `message`를 구성한 다음, 대상 프로그래밍 언어로 코드를 생성하기 위해 `protoc`으로  컴파일한다.

#### Why Protocol Buffer
---
간단하다.
- Size
- Efficiency

JSON 포맷으로  다음과 같은 데이터가 있다고 하자.
```json
{
	"userName": "Martin",
    "favouriteNumber": 1337,
    "interests": ["daydreaming", "hacking"]
}
```
위의 데이터는 공백을 제거했을 때, 82 바이트 공간을 차지한다. 이를 protobuf 스키마로 나타내면 어떤식이 될까?
다음은 위의 JSON 객체를 protobuf 스키마로 나타낸 것이다.
```protobuf
message Person {
	required string user_name		= 1;
	optional int64	favourite_number	= 2;
	repeated string	interests		= 3;
}
```
메시지 형식은 위에서 보는바와 같이 간단하다. 내 눈에는 go나 java의 interface를 보는 것 같기도 하다. 메시지 타입 내에는 field들이 있는데, 각 field는 field name, field type, field tag로 이루어져 있다. 더 자세한 문법은 뒤에서 알아보기로 하자. 
위의 메시지를 바이너리로 인코딩하면 다음과 같이 33 바이트 공간만을 차지하게 된다.

![Image](/assets/images/protobuf.png){:style="     width: 80%; margin: 0 auto; display: block;"}
  [(*Fig. from 'Schema evolution in Avro, Protocol Buffers and Thrift'*)](https://martin.kleppmann.com/2012/12/05/schema-evolution-in-avro-protocol-buffers-thrift.html)

위의 그림을 이해해보면 protobuf의 압축성을 직관적으로 파악할 수 있다. 바이너리화된 데이터의 맨 앞 1바이트는 메타데이터로 두 가지 정보를 가지고 있는데, 앞의 5bit는 `field tag`를 나타내며, 뒤의 3bit는 `field type`을 나타낸다.
그 뒤에 있는 두 번째 바이트는 뒤에서부터 이어질 데이터의 길이를 저장해서 나타낸다. 바이너리화된 protobuf는 분명히 self-decriptive 하지 않지만, 매우 가볍고 전송하기 쉬운 바이너리 스트림으로 압축할 수 있다는 장점이 있다. 조사에 따르면 XML에 대해서는 1/3, JSON에 대해서는 1/2 크기를 차지한다고 한다. 이는 공간뿐 아니라 컴퓨팅에서 가장 비싼 리소스인 네트워크 대역폭인 것을 감안할 때(석사 연구 주제들이 대부분 분산프레임워크에서 네트워크 트래픽 줄이기였던 내 주관이긴 하지만...), 대역폭 감소와 이에 따른 전송 시간 감소는 너무나도 매력적인 메리트이다. 
위에서 언급한 두 가지 큰 장점 외에도 여러 장점이 있다.
- Validation
- 쉽게 확장 가능함.
- 타입 안전성을 보장.
- 하위 호환성.
- Language Independency
- Faster serialization/deserialization


