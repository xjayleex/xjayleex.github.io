---
layout: post
comment: false
title: Go Maps in action
date: 2020-08-06
category: "Golang"
tags: [Golang, Go Maps]
update: 2020-08-06
author: Jaehyun Lee
---

#### Intoduction
---
해시 테이블을 컴퓨터 사이언스에서 가장 유용한 자료구조 중 하나로, 다양한 특징을 제공하는 여러 구현들이 있지만, 보통은 빠른 참조, 추가, 삭제 기능을 제공한다. Go에서는 해시테이블을 구현하는 bulit-in map 타입을 제공한다.

#### Declaration and Initialization
---
Go의map 타입은 다음과 같은 형태를 가진다.
```go
map[KeyType]ValueType
```
Key 타입으로는 `comparable`한 타입이라면 어떠한 타입이든 사용할 수 있고, Value 타입으로는 어떠한 타입이든 사용할 수 있다(심지어 또 다른 map 타입까지도).

다음에 나타낸 변수 `m`은 string key에 integer 값이 매핑되는 map이다.
```go
var m map[string]int
```
map 타입은 포인터와 슬라이와 같은 레퍼런스 타입으로 위의 `m`은 `nil` 값을 가지는 것으로, 어떠한 초기화 된 map도 가르키고 있지 않은 상태다. `nil map`을 읽을 때 빈 map처럼 동작하지만, 쓰려할 때는  runtime panic을 발생시키므로, 초기화 후에 사용하여야 한다. 다음은 map 초기화를 위해 make 함수를 사용하는 예이다.

```go
m = make(map[string]int)
```

make 함수는 해시 맵 자료구조를 할당, 초기화해서 리턴하여, map 변수로 하여금 가르키도록 한다. 본문에서는 map의 사용 방법에 대해 다루며, 구현 자체에 대해서는 다루지 않는다.

#### Working with maps
---
Go는 map을 쉽게 사용하기 위한 간결한 문법을 제공한다. 다음 구문은 "route" key에 값으로 66을 세팅하고 새 변수에 "route" key에 해당하는 값을 뽑아내는 예제이다.

```go
m["route"] = 66
v := m["route"]
```

만약 요청한 키가 없으면, value 타입의 `zero value`를 얻게 된다. 다음 예제의 맵은 integer 형이므로 0을 리턴한다.
```go
v := m["root"]
// v == 0
```

built-in `len` 함수는 map에 있는 아이템의 개수를 리턴한다.
```go
n := len(m)
```

built-in `delete` 함수는  맵으로부터 해당하는 엔트리를 삭제한다.
```go
delete(m, "route")
```
`delete` 함수는 아무것도 리턴하지 않으며, 해당 key가 존재하지 않으면 아무것도 하지 않는다.

'Comma, ok'를 통해 key 존재 여부를 알 수 있다.
```go
v, ok := m["route"]
```
위 구문에서, 첫번째 값 (v)는 key "route"에 저장된 값이 할당되며, 키가 없다면 타입의 zero value가 할당된다. 두번째 값 (ok)에는 키가 존재하는지 안하는지를 나타내는 bool 타입이 할당된다.

range 루프를 통해 map 아이템에 대해 iterate 할 수 있다.
```go
for key, value := range m {
	fmt.Println("Key : ", key, "Value : ", value)
}
```

다음은 map 리터럴을 이용해 map을 초기화하는 예제이다.
```go
m := map[string]int {
	"rsc":	3711,
	"r": 	1222,
	"gri":	8312
}
```
위와 같은 문법은 빈 맵을 초기화하기 위해서도 사용할 수 있다.
```go
m = map[string]int{}
```

#### Exploting zero values
---
맵에 해당 key가 없을 때 zero value를 리턴하도록 하는 매커니즘은 맵 이용을 편리하도록 해준다.

예로써, bool 타입을 value로 가지는 맵은 `Set`과 유사한 데이터 구조로 사용할 수 있다(bool의 zero value는 0임). 다음 예제는 연결 리스트의 노드들을 순회하며 값을 출력한다. 이는 Node 포인터의 맵을 이용해 연결리스트의 사이클을 감지하고 있다.
```go
type Node struct {
	Next	*Node
	Value	interface{}
}
var first *Node

visited := make(map[*Node]bool)
for n := first; n != nil; n = n.Next {
	if visited[n] {
		fmt.Println("cycle detected")
		break
	}
	visited[n] = true
	fmt.Println(n.Value)
}
```
visited[n]은 n이 방문된 경우 True이고 없으면 False이다. 맵에 n이 있는지 없는지 테스트하기 위해 two-value 형식을 사용할 필요가 없이, zero value로 검사할 수 있다.

zero value의 유용한 또 다른 예는 슬라이스 맵이다. nil slice에 append하면 새 슬라이스를 할당하므로 슬라이스 맵에 값을 추가하는 것은 한 줄로 표시된다. 키가 있는지 확인할 필요가 없다. 다음 예제에서 people 슬라이스는 Person 구조체 값으로 채워진다. 각 Person 객체에는 Name과 Likes 슬라이스가 있다. 각 Likes 객체마다 이를 좋아하는 사람들의 리스트 맵을 만드는 예쩨이다.

```go
type Person struct {
        Name  string
        Likes []string
}
var people []*Person

likes := make(map[string][]*Person)
for _, p := range people {
    for _, l := range p.Likes {
        likes[l] = append(likes[l], p)
    }
}
```
"cheese"를 좋아하는 사람을 출력하려면,
```go
for _, p := range likes["cheese"] {
	fmt.Println(p.Name, "likes cheese.")
}
```

#### Key types
---
처음에 언급했듯, 맵의 키로는 comparable한 어떤 타입이든 사용할 수 있다. 언어 사양은 이를 정확하게 정의하고 있다. 대표적인 키 타입으로는 boolean, numeric, string, 포인터, 채널, 인터페이스, 그리고 이러한 타입을 포함하는 구조체와 배열이다. Slice, Map, 함수는 ==를 사용해 비교할 수 없기 때문에 맵 키로 사용할 수 없다. 구조체는 데이터를 여러 차원으로 키잉할 때 사용할 수 있다.
아래부터의 예시는 맵의 맵을 사용해 국가별 웹 페이지 Hit을 집계하는 것을 보여준다.
```go
hits := make(map[string]map[string]int)
```
아래 표현식은 한국인이 페이지를 로드한 횟수를 검색한다.
```go
n := hits["/doc/"]["kr"]
```
이 방식에서는 데이터를 추가하는 방법이 그리 매끄럽지는 못한데, 앞서 있는 맵이 존재하는지 확인하는 과정이 필요하다.
```go
func add(m map[string]map[string]int, path, country string){
	mm, ok := m[path]
	if !ok {
		mm = make(map[string]int)
		m[path] = mm
	}
	mm[country] += 1
}
add(hits, "/doc/", "us")
```
구조체를 이용하면 하나의 맵으로도 이를 간단히 구현할 수 있다.
```go
type Key struct {
	Path, Country string
}
hits := make(map[Key]int)
```
이를 이용해 웹 페이지 Hit 맵을 다시 만든다면 아래와 같이 표현할 수 있다.
```go
type Key struct{
	Path, Country string
}

hits[Key{"/doc/", "us"} += 1
```
