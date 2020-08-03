---
layout: post
comment: false
title: Effective Go (번역 및 요약 정리)
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
---
- 패키지명은 소문자, 한 단어로만 네이밍(언더바(-), 대소문자 혼용 필요 없도록)  
- Naming 지저분하게 하는 것들을 피할 것.
	- ex) bufio 패키지의 Buffered Reader는 BufReader가 아닌 Reader로 불림. 사용자가 bufio.Reader로 사용하게되고, 이게 더 명확하고 깔끔하니까. 임포트된 객체들은 항상 패키지명과 함께 사용되므로, io.Reader와 충돌하지 않음.
	- ex) ring.Ring이라는 구조체의 인스턴스를 생성하는 함수는 NewRing으로 네이밍 할 수도 있지만, 패키지가 ring으로 불리므로, 이때 사용하는 함수는 그냥 New로 네이밍하고, `ring.New()`로 사용. 
	- 위와 같이 사용하기 위해 패키지 구조를 활용할 것.
	- ex) `once.Do`의 경우 once.Do(setup)으로 사용해도 문제가 없으며, once.DoOrWaitUntilDone(setup)으로 한다고 해서 가독성만 나빠질 뿐,  나아지는 것이 없다. 긴 이름은 다는 것 보다, 주석 활용할 것.

#### Getter
---
- getter, setter를 Automatic하게 지원하지는 않으나, 사용되도 좋음.
- getter의 이름에 Get을 넣는 것은 go style이 아님. 만약에 owner 라는 private 필드를 가지고 있다면 getter는 GetOwner가 아닌 Owner라고 네이밍, setter는 SetOnwer라고 네이밍 하면 됨.
{% highlight go %}
owner := obj.Owner()
if owner != user {
	obj.SetOwner(user)
}
{% endhighlight %}

#### Interface name
---
- one-method 인터페이스는 메서드 이름에 `-er` 접미사 붙임. ex) Reader, Writer, Formatter, CloseNotifier

## Control Structures
---
#### If
---
- if와 switch가 `초기화 구문` 허용
{% highlight go %}
if err := file.Chmod(0604); err != nil {
	log.Print(err)
	return err
}
{% endhighlight %}
- 불필요한 else문 생략

#### For
---
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
---
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
---
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
---
#### Named result parameters
---
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
---
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
`defer` 함수의 파라미터들은 함수가 호출 될 때가 아니라, `defer`가 실행될 때 평가된다.
또한, 함수 실행시 변숙밧이 변하는 것에 대해 걱정할 필요가 없다. 특정 defer 호출 위치에서 여러개의 함수 호출을 지연할 수 있는 것이다.

{% highlight go %}
for i := 0; i < 5; i++ {
    defer fmt.Printf("%d ", i)
}
{% endhighlight %}
연기된 함수는 LIFO 순서에 의해 실행되며, 4 3 2 1 0 을 출력한다.

## Data
---
Go에서 메모리를 할당하기 위한 방식으로는 built-in 함수 `new`와 `make`가 있다

#### new를 이용한 메모리 할당
---
`new`는 메모리를 할당하긴 하지만, 다른 언어에 존재하는 new와는 다르게 메모리를 초기화하지 않고 단지 값을 제로화환다. 다시 말해, new(T)는 Type T의 새로운 객체에 제로값이 저장된 공간을 할당하고 그 객체의 주소인, `*T`값을 반환하는 것이다. 새로 제로값으로 할당된 타입 T를 가리키는 포인터를 반환하는 것이다.
`new`를 통해 반환된 메모리는 제로값을 가지고 있기 때문에, 초기화 과정없이 사용된 타입의 제로값을 쓸 수 있도록 자료구조를 설계하면 좋다. 예로써, `byte.Buffer`는 문서에서 "Buffer의 제로값은 바로 사용할 수 있는 빈 버퍼이다"라고 한다. `sync.Mutex`도 어떠한 contructor나 Init 메서드가 없이, 제로값은 잠기지 않은 mutex로 정의되어 있다.

제로값의 이러한 이점은 전이적인 특성이 있다.
{% highlight go %}
type SyncedBuffer struct {
	lock 	sync.Mutex
	buffer	bytes.Buffer
}

p := new(SyncedBuffer)
var v SyncedBuffer
{% endhighlight %}
`SyncedBuffer` 타입은 메로리 할당이나 선언만으로 당장 사용할 수 있다.

#### Constructor and Composite Literal
---
때로는 제로값만으로는 충분하지 않고, 생성자로 초기화해야 할 필요가 있다.
{% highlight go %}
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }
    f := new(File)
    f.fd = fd
    f.name = name
    f.dirinfo = nil
    f.nepipe = 0
    return f
}
{% endhighlight %}
위 예시에는 불필요하게 코드들이 반복되어 있다(boiler plate). 이를 합성 리터럴(composite literal)로 간소화할 수 있고, 표현이 실행될 때마다 새로운 인스턴스를 만들어 낸다.

{% highlight go %}
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }
    f := File{fd, name, nil, 0}
    return &f
}
{% endhighlight %}
만약, 합성 리터럴이 필드를 아예 가지지 않을 떄에는, 제로값을 생성한다. `new(File)`은 `&File{}`과 동일한 표현이다.

#### make를 이용한 메모리 할당
---
`make(T,args)`는 `new(T)`와 다른 목적의 서비스를 제공한다. 이는 slice, map, channel에만 사용하고 (*T가 아닌) Type T의 제로값이 아닌 초기화된 값을 반환한다. 이는 위의 세 타입이 사용전에 반드시 내부적으로 초기화 되어야하는 자료구조를 가리키고 있기 때문이다. 예로써, slice는 데이터를 가리키는 포인터, Len, Capacity를 인자로 가지며, 이 항목들이 초기화되기 전까지는 `nil`이다. 

다음 예제는 `new`와 `make`의 차이점을 보여준다.
{% highlight go %}
var p *[]int = new([]int)       // slice 구조체를 할당한다; *p == nil; 거의 유용하지 않다
var v  []int = make([]int, 100) // slice v는 이제 100개의 int를 갖는 배열을 참조한다

// 불필요하게 복잡한 경우:
var p *[]int = new([]int)
*p = make([]int, 100, 100)

// Go 언어다운 경우:
v := make([]int, 100)
{% endhighlight %}
`make`는 map, slice, channel에만 적용되며, 포인터를 리턴하지 않는다. 포인터를 이용하기 위해서는 new를 사용해 메모리를 할당하거나, 변수 주소를 얻는 방법을 이용한다.

#### Array
---
- 배열은 Value로, 배열을 다른 배열에 Assign 할 때 모든 요소가 복사된다.
- 배열 크기는 Type의 한 부분으로, 예를 들어 `[10]int`와 `[20]int`는 서로 다른 타입이다.
함수에 pass할 때, 복사를 막아 효율성을 위한다면 배열 포인터를 사용할 수 있다.
{% highlight go %}
func Sum(a *[3]float64) (sum float64) {
    for _, v := range *a {
        sum += v
    }
    return
}

array := [...]float64{7.0, 8.5, 9.1}
x := Sum(&array)
{% endhighlight %}
하지만 이는 Go style이 아니다.

#### Slices
---
Slice는 내부의 배열을 가리키는 레퍼런스를 가지고 있어, 만약 다른 slice에 Assign 할 때, 둘다 같은 배열을 가리키게 된다. 
{% highlight go %}
func (f *File) Read(buf []byte) (n int, err error)

n, err := f.Read(buf[0:32])
{% endhighlight %}
위 메서드는 읽은 바이트 수와 에러 값을 리턴한다. 그 다음 라인에서는, buf의 첫 32 바이트를 읽어 들이기 위해 버퍼를 Slice 한다. 이런 슬라이싱은 자주 사용되며, 효율적이다.

`Slice`의 capacity는 `cap`을 통해 얻을 수 있다. 아래에 slice에 데이터를 append 할 수 있는 함수가 있다. Len이 Capacity를 초과하면, Slice의 메모리는 재할당된다. 이 함수는 `len`과 `cap`이 `nil slice`에서 0을 리턴한다는 사실을 이용하고 있다.
{% highlight go %}
func Append(slice, data []byte) []byte {
    l := len(slice)
    if l + len(data) > cap(slice) {  // reallocate
        // Allocate double what's needed, for future growth.
        newSlice := make([]byte, (l+len(data))*2)
        // The copy function is predeclared and works for any slice type.
        copy(newSlice, slice)
        slice = newSlice
    }
    slice = slice[0:l+len(data)]
    copy(slice[l:], data)
    return slice
}
{% endhighlight %}
Slice가 passed by Value 되었으므로, 처리 후 return 되어야 한다.

#### Two Dimensional Slice
---
이차원 slice를 만드려면...
{% highlight go %}
type Transform [3][3]float64  // A 3x3 array, really an array of arrays.
type LinesOfText [][]byte     // A slice of byte slices.
{% endhighlight %}

Slice는 크기가 변할 수 있기 떄문에, 각 Slice 들의 크기는 서로 다를 수 있다.
{% highlight go %}
text := LinesOfText{
	[]byte("Now is the time"),
	[]byte("for all good gophers"),
	[]byte("to bring some fun to the party."),
}
{% endhighlight %}
이차원 slice를 메모리에 할당하기 위해서 두 가지 방법을 사용할 수 있다.
첫번째는 slice를 독립적으로 할당하는 것이고, 두번째는 slice 하나를 할당한 뒤, 각 slice로 자른 다음 포인터를 가지는 방식이다.
첫번째, 
{% highlight go %}
// Allocate the top-level slice.
picture := make([][]uint8, YSize) // One row per unit of y.
// Loop over the rows, allocating the slice for each row.
for i := range picture {
	picture[i] = make([]uint8, XSize)
}
{% endhighlight %}
두번째,
{% highlight go %}
picture := make([][]uint8, YSize) // 유닛 y마다 한 줄씩.
// 모든 픽셀들을 담을 수 있는 큰 slice를 할당하라.
pixels := make([]uint8, XSize*YSize) // picture는 [][]uint8 타입이지만 pixels는 []uint8 타입.
// 각 줄을 반복하면서, 남겨진 pixels slice의 처음부터 크기대로 슬라이싱하라.
for i := range picture {
	picture[i], pixels = pixels[:XSize], pixels[XSize:]
}

{% endhighlight %}
slice가 커질 수 있다면, 독립적으로 할당해서 다음 부분을 Overwrite하는 경우를 방지해야한다. 그렇지 않다면, 메로리 할당을 한번에 하는 것이 더 효율적일 수 있다.

#### Map
---
`Map`의 Key는 `equality 연산`이 정의되어 있는 타입이라면, 어느 경우에나 사용할 수 있다.
- Integer, Floating Point, Complex number, strings, pointer, struct, equality를 지원하는 동적 타입인 Interface), Array
Slice는 key로 사용할 수 없는데, equality가 정의되어 있지 않기 때문이다. Slice와 마찬가지로 내부 데이터를 가지므로, 함수에 map을 pass하고 데이터 변경이 발생하면, 호출자에서도 변경된다.
Map에 없는 key에 대한 fetch는 타입에 해당하는 제로값을 반환한다.
bool Type을 가진 map으로 `Set`을 구현할 수 있다.
{%highlight go %}
attended := map[string]bool{
    "Ann": true,
    "Joe": true,
    ...
}

if attended[person] { // 만약 person이 맵에 없다면 false일 것이다.
    fmt.Println(person, "was at the meeting")
}
{% endhighlight %}

map에 없는 값과 제로값을 구분해야할 필요가 있는 경우도 있는데, 복수 할당의 형태로 구분 가능하다.
{% highlight go %}
var seconds int
var ok bool
seconds, ok = timeZone[tz]
{% endhighlight %}
이를 "comma ok" 관용구라 한다. tz가 있으면, ok는 true, 없으면 seconds는 제로값이 되고 ok는 false가 된다.

#### Append
---
`append`의 시그니처는 도식적으로 아래와 같다.
{% highlight go %}
func append(slice []T, elements ...T) []T
{% endhighlight %}
여기서 T는 어떠한 Type의 플레이스 홀더다. Go 에서는 Caller에 의해 결정되는 타입 T를 쓰는 함수를 만들 수 없다(역 : generic 처럼, 후에 generic이 추가된다고 하니 두고봐야 할 듯). 그래서 append는 내장함수이고, 컴파일러의 지원이 필요하다.
append는 slice의 끝에 요소들을 붙이고 결과를 리턴한다. 

{% highlight go %}
x := []int{1,2,3}
x = append(x, 4, 5, 6)
fmt.Println(x)
{% endhighlight %}

이 예제는 [1 2 3 4 5 6]을 출력한다.
slice에 slice를 붙이고 싶다면 `...`을 이용한다.

{% highlight go %}
x := []int{1,2,3}
y := []int{4,5,6}
x = append(x, y...)
fmt.Println(x)
{% endhighlight %}

...이 없으면 컴파일 되지 않는다. y는 int 타입이 아니기 때문이다.

## Initialization
---
#### Constants
---
Go에서는 열거형(enum) 상수를 iota라는 enumerator를 이용해 생성한다. iota는 암묵적으로 반복될 수 있어, 복잡한 값들로 구성된 집합 생성을 쉽게 한다.
{% highlight go %}
type ByteSize float64

const (
    _           = iota // 공백 식별자를 이용해서 값인 0을 무시
    KB ByteSize = 1 << (10 * iota)
    MB
    GB
    TB
    PB
    EB
    ZB
    YB
)
{% endhighlight %}

`ByteSize` 자료형의 String 메소드

{% highlight go %}
func (b ByteSize) String() string {
    switch {
    case b >= YB:
        return fmt.Sprintf("%.2fYB", b/YB)
    case b >= ZB:
        return fmt.Sprintf("%.2fZB", b/ZB)
    case b >= EB:
        return fmt.Sprintf("%.2fEB", b/EB)
    case b >= PB:
        return fmt.Sprintf("%.2fPB", b/PB)
    case b >= TB:
        return fmt.Sprintf("%.2fTB", b/TB)
    case b >= GB:
        return fmt.Sprintf("%.2fGB", b/GB)
    case b >= MB:
        return fmt.Sprintf("%.2fMB", b/MB)
    case b >= KB:
        return fmt.Sprintf("%.2fKB", b/KB)
    }
    return fmt.Sprintf("%.2fB", b)
}
{% endhighlight %}

#### Variable
---
변수는 상수와 같은 방식으로 초기화하며, 런타임에 계산되는 표현식이어도 된다.

{% highlight go %}
var (
    home   = os.Getenv("HOME")
    user   = os.Getenv("USER")
    gopath = os.Getenv("GOPATH")
)
{% endhighlight %}

#### Init function
---
init 함수를 정의해 필요한 상태를 셋업할 수 있다. init 함수는 매개변수를 가지지 않는다. init 함수는 import된 모든 패키지들이 초기화되고 패키지 내의 모든 변수 선언이 평가된 이후에 호출된다.
선언의 형태로 표현하지 못하는 것들 외에도, 실제 프로그램이 실행되기 전에 프로그램 상태를 검증하고, 올바르게 동작하도록 복구하는데 자주 사용된다.

{% highlight go %}
func init() {
    if user == "" {
        log.Fatal("$USER not set")
    }
    if home == "" {
        home = "/home/" + user
    }
    if gopath == "" {
        gopath = home + "/go"
    }
    // gopath may be overridden by --gopath flag on command line.
    flag.StringVar(&gopath, "gopath", gopath, "override default GOPATH")
}
{% endhighlight %}

## Method
---
#### Pointer vs Value
---
[ByteSize](constants)에서 보듯이, 
