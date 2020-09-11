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

### Protocol Buffer Basics

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
// protobuf2 syntax
message Person {
	required string user_name		= 1;
	optional int64	favourite_number	= 2;
	repeated string	interests		= 3;
}
```
메시지 형식은 위에서 보는바와 같이 간단하다. 내 눈에는 go나 java의 interface를 보는 것 같기도 하다. 메시지 타입 내에는 field들이 있는데, 각 field는 field name, field type, field tag로 이루어져 있다. 더 자세한 문법은 뒤에서 알아보기로 하자. 
위의 메시지를 바이너리로 인코딩하면 다음과 같이 33 바이트 공간만을 차지하게 된다.

![Image](/assets/images/protobuf.png){:style="width: 80%; margin: 0 auto; display: block;"}
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

#### Supported Languages
---
protobuf2에서는 Java, Python, Objective-C, C++ 만을 지원하지만, protobuf3에서는 거의 모든 메인스트림 언어에 대해서 코드를 생성해준다.
> `Go`, `C++`, `Java`, `Python`, `Erlang`, `Ruby`, etc.  

#### Scalar Value Types
---
JSON 및 XML에서와 같이, hierarchical한 데이터 구조를  나타내기 위한 모든 데이터 타입을 사용할 수 있다. 다음은 `.proto` 파일 내에서의 protobuf 필드 타입들이며, 생성하고자 하는 대상 언어에서 각각 상응하는 데이터 타입을 확인하면 된다.
![Image](/assets/images/typemapping.png){:style="width: 90%; margin: 0 auto; display: block;"}

### **protobuf3 Language Guide**

Protocol Buffer를 사용해 각 언어 별 generated code를 생성하기 위해서는 .proto 파일을 작성해야한다. 즉, 문법을 알아야 한다. 간단한 메세지 타입부터 알아보도록 하자. 다음부터의 설명은 [*'Google Protocol Buffer' 공식 문서*](https://developers.google.com/protocol-buffers/docs/proto3)에 나와있는 예제 설명이다.

#### Defining A Message Type
---
검색 Request 마다 쿼리 문자열, 특정 결과 페이지 그리고 페이지 당 여러 검색 결과가 있는 검색 Request 메시지를 정의해보자.
```protobuf
syntax = "proto3";

message SearchRequest {
	// type field_name = field_number;
	string query = 1;
	int32 page_number = 2;
	int32 result_per_age = 3;
}
```
- 첫 번째 라인은 proto3 문법을 사용하고 있음을 나타낸다. 만약 설정하지 않고 디폴트 값을 사용했을 때에는 proto2 문법을 사용한다. 이 구문은 파일의 첫 번째 행에만 작성한다.
- 친숙하게 느껴지는 부분이다. 메시지에 포함하는 각 필드는 Type, Field Name, Field Number(tag)로 나타낸다. 

#### Assigning Field Numbers
---
메시지 정의에서 각 필드에는 고유 번호가 있다. 이 필드 번호는 메시지의 바이너리 포맷에서 필드 자체의 식별자로 사용되며, 기존에 사용되고 있던 메시지를 수정했을 때 절대로 변경되어서는 안된다.
필드 번호는 번호에 따라서 사용하는 공간이 다른데,
1~15 범위의 필드 번호는 필드 번호 및 필드 타입을 포함해 인코딩 시 1Byte 공간을 사용하고,
16-2047 범위의 필드 번호는 2바이트를 사용한다. 따라서 자주 사용하는 메시지 요소에 대해서는 1에서 15까지의 숫자를 사용해 최적화한다. 지정할 수 있는 번호 범위는 더 광범위하지만 최근 MSA 트렌드를 볼 때, 그 정도까지 필요할까 싶다.

#### Adding More Message Types
---
하나의 `.proto`파일에 여러 메시지 타입을 정의할 수 있다. 다음과 같이 서로 관련된 메시지 타입들이 있을 때, 이러한 방법을 사용한다. 
```protobuf
message SearchRequest {
	string query = 1;
	int32 page_number = 2;
	int32 result_per_age = 3;
}

message SearchResponse{
	...
}
```

#### Reserved Fields
---
기존에 사용하고 있던 레거시 proto 파일이 있는데 기존 메시지에서 사용하고 있던 필드 중, 어떤 부분을 제거해야하는 상황이라고 해보자. 이 때 필드를 완전히 제거하거나 주석 처리하면 안된다. 이렇게 되면 추후에 이 proto 파일을 관리하는 프로그래머는 다른 필드를 추가하거나 업데이트 할 때 기존에 넘버링되어 있던 필드 번호를 재사용할 수 있게된다. 이 경우 동일한 proto 파일의 이전 버전을 로드 하게 될 경우 심각한 문제가 발생할 수 있다. 이러한 문제를 방지하기 위해 `reserved` 키워드를 사용해, 제거된 필드의 필드 번호를 예약 지정할 수 있다. protoc 컴파일러는 향후 프로그래머가 `reserved` 식별자를 사용하려고 하면 에러를 발생시킬 것이다. 단, 한 `reserved` 구문에서 필드 이름과 필드 번호를 혼용해서 쓸 수는 없다.
```protobuf
message Msg{
	reserved 2, 15, 9 to 11;
	reserved "foo", "bar";
```
#### Code generated from proto file
---
`.proto` 파일을 컴파일 할 때, `protoc` 컴파일러는 getter, setter 설정 및, output 스트림으로의 메시지 직렬화, input stream으로부터의 메시지 파싱을 포함해, 파일에서 기술한 메시지 유형으로 작업하는데에 필요한 코드들을 선택한 언어로 생성한다. 대표적인 언어에 대해서 각각 살펴보자면, 
- go의 경우, 각 메시지 타입에 대한 go 타입을 가진 `.pb.go` 파일을,
- C++의 경우 `.h` 및 `.cc` 파일을, 
- Java의 경우 메시지 타입의 클래스 인스턴스를 생성하기 위한 Builder 클래스가 있는 `.java` 파일.

#### Default Values
---
앞서 `.proto` 파일 내의 필드 타입과 대상 프로그래밍 언어(본문에서는 go)의 타입간 매핑에 대해서 언급했다.
메시지를 파싱 할 때, 메시지에 `singular` 요소가 포함되어 있지 않으면, 파싱된 객체에 상응하는 필드가 필드 타입의 기본값으로 설정된다. 

![Image](/assets/images/proto_default.png){:style="width : 90%; margin: 0 auto; display: block;"}

#### Enumerations
---
Scalar 타입뿐만 아니라, 친숙한 enum 타입도 사용 할 수 있다. 메시지 유형을 정의 할 때, 미리 정의된 목록 중 하나만 포함하도록 할 수 있는 것이다. 예로써, SearchRequest 메시지에 각 요청에 대해 분류를 나타낼 `Kinds` 필드를 추가한다고 생각해보자. 검색 결과의 분류로는 아마도 `WEB`, `IMAGES`, `NEWS`, `PRODUCTS` 등이 있을 수 있다. 또한, 이는 바로 `enum` 열거형으로 나타낼 수 있다.
```protobuf
message SearchRequest {
	string query = 1;
	int32 page_number = 2;
	int32 result_per_page = 3;
	enum Kinds{
		WEB = 0;
		IMAGES = 1;
		NEWS = 2;
		PRODUCTS = 3;
	}
	Kinds kinds = 4;
}
```
위에서 보는바와 같이 `Kinds`의 원소는 0부터 매핑된다. 모든 열거형의 정의는 0으로 첫번째 요소로 매핑되는 상수를 포함해야 한다. 그래야지 열거형에 대한 default 값으로 zero value를 사용할 수 있기 때문이다. 또, proto2에서는 열거형 값이 default로 0이여서 호환성을 위함이다.

열거형 상수에 alias를 할 수도 있다. 이렇게 하기 위해서는 `allow_alias` 옵션이 `true`로 설정되어야 한다. 그렇지 않고 사용하게 되면 컴파일러는 alias를 찾지 못헀다는 에러 메세지를 출력한다.

```protobuf
message MyMessage1 {
  enum EnumAllowingAlias {
    option allow_alias = true;
    UNKNOWN = 0;
    STARTED = 1;
    RUNNING = 1;
  }
}
message MyMessage2 {
  enum EnumNotAllowingAlias {
    UNKNOWN = 0;
    STARTED = 1;
    // RUNNING = 1;  // Uncommenting this line will cause a compile error inside 
	//Google and a warning message outside.
  }
}
```

Enumerator의 상수는 32bit 정수 범위 내에 있어야 한다.
열거형 값은 `varint encoding`을 사용하므로 음수 값은 권장되지 않는다.
열거형은 .proto 파일의 모든 메시지 정의에서 재사용 가능하다. 재사용하려면 `_MessageType_._EnumType_`과 같은 문법을 사용하면 된다.
열거형을 사용하는 `.proto`에서 protoc을 실행하면 생성된 코드에는 java의 경우 `enum`, python의 경우 `EnumDescriptor`라는 클래스가 생성된다. 이는 정수로된 상수 기호 집합을 생성하기 위해 사용된다.

역직렬화 과정에서는 인식되지 않은 열거형 값들이 표현되는 방식은 언어에 따라 다르다.

#### Using Other Message Types
---
다른 메시지 타입을 필드 타입으로 임포트해서 사용 할 수 있다. 예로써, `SearchResponse` 메시지에 `Result`  메시지를 포함하고 싶은 상황을 가정해보자. 동일한 `.proto` 파일에 `Result` 메시지 타입을 정의한 다음 `SearchResponse`에 `Result` 타입 필드를 선언할 수 있다.

```protobuf
message SearchResponse {
	repeated Result results = 1;
}

message Result {
	string url = 1;
	string title = 2;
	repeated string snippets = 3;
}
```

#### Importing Definitions
---
필드 타입으로 사용하는 메시지 타입이 다른 `.proto` 파일에 정의되어 있는 경우에는 그 정의를 `import` 해서 사용할 수 있다. 

```protobuf
import "myproject/other_protos.proto";
```

#### References
---
- [*'Schema evolution in Avro, Protocol Buffers and Thrift'*](https://martin.kleppmann.com/2012/12/05/schema-evolution-in-avro-protocol-buffers-thrift.html)  
- [*'Understanding Protocol Buffers'*](https://medium.com/better-programming/understanding-protocol-buffers-43c5bced0d47)  

