---
layout: post
comment: false
title: Go Effective 정리
date: 2020-08-02
category: "Golang"
tags: [Golang, Go Effective]
author: Jaehyun Lee
---

##### [Effective Go](https://golang.org/doc/effective_go.html) 의 개인 참고용 번역 및 요약본입니다.


[**Type Switch**](#type-switch)

## Naming
---
#### Package names

- 패키지명은 소문자, 한 단어로만 네이밍(언더바(-), 대소문자 혼용 필요 없도록)  
- Naming 지저분하게 하는 것들을 피할 것.
	- ex) bufio 패키지의 Buffered Reader는 BufReader가 아닌 Reader로 불림. 사용자가 bufio.Reader로 사용하게되고, 이게 더 명확하고 깔끔하니까. 임포트된 객체들은 항상 패키지명과 함께 사용되므로, io.Reader와 충돌하지 않음.
	- ex) ring.Ring이라는 구조체의 인스턴스를 생성하는 함수는 NewRing으로 네이밍 할 수도 있지만, 패키지가 ring으로 불리므로, 이때 사용하는 함수는 그냥 New로 네이밍하고, `ring.New()`로 사용. 
	- 위와 같이 사용하기 위해 패키지 구조를 활용할 것.
	- ex) `once.Do`의 경우 once.Do(setup)으로 사용해도 문제가 없으며, once.DoOrWaitUntilDone(setup)으로 한다고 해서 가독성만 나빠질 뿐,  나아지는 것이 없다. 긴 이름은 다는 것 보다, 주석 활용할 것.

#### Getter

- getter, setter를 Automatic하게 지원하지는 않으나, 사용되도 좋음.
- getter의 이름에 Get을 넣는 것은 go style이 아님. 만약에 owner 라는 private 필드를 가지고 있다면 getter는 GetOnwer가 아닌 Owner라고 네이밍, setter는 SetOnwer라고 네이밍 하면 됨.
{% highlight go %}
owner := obj.Owner()
if owner != user {
	obj.SetOwner(user)
}
{% endhighlight %}

#### Interface name

- one-method 인터페이스는 메서드 이름에 `-er` 접미사 붙임. ex) Reader, Writer, Formatter, CloseNotifier

## Control Structures
---
#### If

- if와 switch가 `초기화 구문` 허용
{% highlight go %}
if err := file.Chmod(0604); err != nil {
	log.Print(err)
	return err
}
{% endhighlight %}
- 불필요한 else문 생략

#### For

Array, Slice, String, Map, Channel에 대한 반복문을 작성한다면, range 구문 사용 가능
{% highlight go %}
for key, value := range oldMap {
	newMap[key] = value
}
{% endhighlight %}

만약 range 안에서 첫번째 아이템만이 필요하면, 두번째는 명시하지 않음으로써 구현
{% highlight go %}
for key := range m {
	if key.expired() {
		delete(m, key)
	}
}
{% endhighlight %}

range 안에서 두번째 아이템만이 필요하면, Blank Identifier 명시
{% highlight go %}
sum := 0
for _, value := range array {
	sum += value
}
{% endhighlight %}

Go에는 콤마(,) 연산자가 없고, `++, --`는 표현식이 아닌 명령문임. for loop에서 여러개 변수를 사용하려면 parallel assignment 사용
{% highlight go %}
for i, j := 0, len(a)-1; i < j ; i, j = i+1, j-1 {
	a[i], a[j] = a[j], a[i]
}
{% endhighlight %}

#### Switch

Go에서 Switch는 다른 언어에서 보다 더 일반적인 표현이 가능하다. 표현식은 상수이거나 정수일 필요 없다. case 구문은 위에서부터 바닥까지 해당 구문이 `true`가 아닌 동안 일치하는 값을 찾을 때까지 값을 비교해 나간다.
`if-else-if-else` 형태보다 `switch`를 사용하는 것이 **더 Go언어 답다.**
{% highlight go %}
func unhex(c byte) byte {
    switch {
    case '0' <= c && c <= '9':
        return c - '0'
    case 'a' <= c && c <= 'f':
        return c - 'a' + 10
    case 'A' <= c && c <= 'F':
        return c - 'A' + 10
    }
    return 0
}
{% endhighlight %}

`switch`에서는 자동으로 케이스 구문을 통과하는 동작이 없지만, 콤마로 구분된 목록을 사용해 표현할 수 있다.
{% highlight go %}
switch sholudEscape(c byte) bool {
	switch c {
	case ' ', '?', '&', '!' :
		return true
	}
	return false
}
{% endhighlight %}

`switch`문 이용한 바이트 슬라이스 비교 루틴
{% highlight go %}
// Compare returns an integer comparing the two byte slices,
// lexicographically.
// The result will be 0 if a == b, -1 if a < b, and +1 if a > b
func Compare(a, b []byte) int {
    for i := 0; i < len(a) && i < len(b); i++ {
        switch {
        case a[i] > b[i]:
            return 1
        case a[i] < b[i]:
            return -1
        }
    }
    switch {
    case len(a) > len(b):
        return 1
    case len(a) < len(b):
        return -1
    }
    return 0
}
{% endhighlight %}

#### Type Switch

스위치 구문은 `인터페이스 변수`의 동적 타입을 확인하는 데 사용 될 수 있다. 이러한 switch는 type assertion 문법을 사용하되, 키워드는 type을 사용한다. 
{% highlight go %}
var t interface{}
t = functionOfSomeType()
switch t := t.(type) {
default:
    fmt.Printf("unexpected type %T\n", t)     // %T prints whatever type t has
case bool:
    fmt.Printf("boolean %t\n", t)             // t has type bool
case int:
    fmt.Printf("integer %d\n", t)             // t has type int
case *bool:
    fmt.Printf("pointer to boolean %t\n", *t) // t has type *bool
case *int:
    fmt.Printf("pointer to integer %d\n", *t) // t has type *int
}
{% endhighlight %}

## Function

#### Named result parameters

이름 있는 결과 인자값을 통해 코드를 짧게, 명확하게 할 수 있다. 또, 문서화에 도움된다.
{% highlight go %}
func ReadFull(r Reader, buf []byte) (n int, err error) {
    for len(buf) > 0 && err == nil {
        var nr int
        nr, err = r.Read(buf)
        n += nr
        buf = buf[nr:]
    }
    return
}
{% endhighlight %}

#### Defer

Go의 `defer`문은 `defer`을 실행하는 함수가 리턴하기 전에 `defer`에 명시된 함수를 호출하도록 예약한다. Mutex의 잠금을 풀거나, 네트워크 Channel 등을 닫아야하는 상황에 효과적인 방법이다.
{% highlight go %}
func Contents(filename string) (string, error) {
    f, err := os.Open(filename)
    if err != nil {
        return "", err
    }
    defer f.Close()  // f.Close will run when we're finished.

    var result []byte
    buf := make([]byte, 100)
    for {
        n, err := f.Read(buf[0:])
        result = append(result, buf[0:n]...) // append is discussed later.
        if err != nil {
            if err == io.EOF {
                break
            }
            return "", err  // f will be closed if we return here.
        }
    }
    return string(result), nil // f will be closed if we return here.
}
{% endhighlight %}
`f.Close()`와 같은 함수 호출을 연기하는 것은 두 가지 장점을 가져다 준다. 첫번째, 파일을 닫는 것을 잊어버리는 실수를 하지 않도록 해준다. 두번째, `Open` 근처에 `Close`를 위치 시킴으로써 훨씬 더 Readable한 코드를 만들어준다.
`defer` 함수의 파라미터들은 함수가 호출 될 때가 아니라, `defer`가 실행될 때 ㅂ평가된다.

{% highlight go %}
for i := 0; i < 5; i++ {
    defer fmt.Printf("%d ", i)
}
{% endhighlight %}
