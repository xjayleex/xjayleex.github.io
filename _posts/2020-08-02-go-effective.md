---
layout: post
comment: false
title: Effective Go (번역 및 요약 정리)
date: 2020-08-02
category: "Golang"
tags: [Golang, Go Effective]
author: Jaehyun Lee
---

#### [Effective Go](https://golang.org/doc/effective_go.html) 의 개인 참고용 번역 및 요약본입니다.
### Contents  
[**Naming**](#naming)  
 * [**Package names**](#package-names)  
	[**Getter**](#getter)  
	[**Interface names**](#interface-names)  
[**Control Structures**](#control-structures)  
	[**If**](#if)  
	[**For**](#for)  
	[**Switch**](#switch)  
	[**Type Switch**](#type-switch)  
[**Function**](#function)
	[**Named result parameters**](#named-result-parameters)
	[**Defer**](#defer)
[**Data**](#data)
	[**Allocation with New**](#allocation-with-new)
	[**Constructor and Composite literal**](#constructor-and-composite-literal)
	[**Allocation with Make**](#allocation-with-make)
	[**Array**](#array)
... [**Slices**](#slices)
... [**Two Dimensional Slice**](#two-dimensional-slice)
... [**Map**](#map)
... [**Append**](#append)
[**Initialization**](#initialization)
... [**Constants**](#constants)
... [**Variable**](#variable)
... [**The init function**](#the-init-function)
[**Method**](#method)
... [**Pointers vs. Values**](#pointers-vs.-values)
[**Interface and other types**](#interface-and-other-types)
... [**Interface**](#interface)
... [**Conversions**](#conversions)
... [**Interface conversions and type assertions**](#interface-conversions-and-type-assertions)
... [**Generality**](#generality)
... [**Interfaces and methods**](#interfaces-and-methods)
[**The blank identifier**](#the-blank-identifier)
... [**The blank identifier in multiple assignment**](#the-blank-identifier-in-multiple-assignment)
... [**Unused imports and variables**](#unused-imports-and-variables)
... [**Import for side effect**](#import-for-side-effect)
... [**Interface checks**](#interface-checks)
[**Embedding**](#embedding)
[**Errors**](#errors)
... [**Panic**](#panic)
... [**Recover**](#recover)

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

#### Interface names
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
Go에서 메모리를 할당하기 위한 방식으로는 built-in 함수 `new`와 `make`가 있다.

#### Allocation with New
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

#### Constructor and Composite literal
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

#### Allocation with Make
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

#### The init function
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
#### Pointers vs. Values
---
[**ByteSize**](#constants)에서 볼 수 있듯, 메서드는 모든 타입에 대해 정의할 수 있다. 
Slice에 대한 논의에서 Append 함수를 작성했다. 이를 Slice의 메서드로서 정의 할 수 있다.

{% highlight go %}
type ByteSlice []byte
func (slice ByteSlice) Append(data []byte) []byte {
	...
}
{% endhighlight %}
위의 함수는 여전히 슬라이스를 리턴해야할 필요가 있다. `ByteSlice`에 대한 포인터를 리시버로 받을 수 있게 재정의함으로써 이러한 오버헤드를 없앨 수 있다.

{% highlight go %}
func (p *ByteSlice) Append(data []byte) {
    slice := *p
	...
    *p = slice
}
{% endhighlight %}
이를 더 발전시켜 표준 `Write` 메소드처럼 만들어보면,
{% highlight go %}
func (p *ByteSlice) Write(data []byte) (n int, err error) {
    slice := *p
	...
    *p = slice
    return len(data), nil
}
{% endhighlight %}
위의 `*ByteSlice`는 표준 인터페이스인 `io.Write`를 따르게되며, 다음과 같이 다루기 편해진다.
{% highlight go %}
var b ByteSlice
fmt.Fprintf(&b, "This hour has %d days\n", 7)
{% endhighlight %}
리시버로 포인터를 쓸 것인지 Value를 쓸 것인지에 대한 규칙은,
Value만 사용하는 메서드는 포인터와 Value에서 모두 사용할 수 있으며,
포인터 메서드의 경우에는 포인터에서만 사용 가능하다.
Go에서는 Value에서 포인터 메서드를 실행하는 것을 허용하지 않는다. 주소를 얻을 수 있는 Value의 경우, 포인터 메소드를 Value에 대해 실행할 경우 자동으로 주소 연산을 넣어준다. 위의 예시에서 `b`는 주소로 접근 할 수 있으므로 `b.Write`만으로 `Write` 메서드를 호출할 수 있다. 컴파일러가 이를 `(&b).Write`로 바꿔줄 것이다.

## Interface and other types
---
#### Interface
---
Go 인터페이스를 이용해 객체의 행위를 지정해줄 수 있다. 어떠한 객체가 정해진 행동을 할 수 있다면, 호환되는 타입으로 사용 할 수 있다는 의미이다(이게 진정한 인터페이스의 의미인듯?).
Go 인터페이스에서는 보편적으로 한 두개의 메서드를 지정해준다. 인터페이스 이름은 메서드(동사)로 부터 나온다.
각 타입은 복수의 인터페이스를 구현할 수 있다. 예로써, `sort.Interface`를 구현하고 있는 `collection`을 들 수 있다.
{% highlight go %}
type Interface interface {
    // Len is the number of elements in the collection.
    Len() int
    // Less reports whether the element with
    // index i should sort before the element with index j.
    Less(i, j int) bool
    // Swap swaps the elements with indexes i and j.
    Swap(i, j int)
}
{% endhighlight %}
다음의 `Sequence`는 두 개의 인터페이스를 구현하고 있다.
{% highlight go %}
type Sequence []int

// sort.Interface를 위한 필수적인 메서드들.
func (s Sequence) Len() int {
    return len(s)
}
func (s Sequence) Less(i, j int) bool {
    return s[i] < s[j]
}
func (s Sequence) Swap(i, j int) {
    s[i], s[j] = s[j], s[i]
}

// 프린팅에 필요한 메서드 - 프린트하기 전에 요소들을 정렬함.
func (s Sequence) String() string {
    sort.Sort(s)
    str := "["
    for i, elem := range s {
        if i > 0 {
            str += " "
        }
        str += fmt.Sprint(elem)
    }
    return str + "]"
}
{% endhighlight %}

#### Conversions
---
`Sequence`의 String 메서드는 Sprint가 슬라이스를 가지고 하는 일을 반복하고 있는데, Sprint를 실행하기 전 Sequence를 []int로 변환해 작업을 줄일 수 있다.
{% highlight go %}
func (s Sequence) String() string {
    sort.Sort(s)
    return fmt.Sprint([]int(s))
}
{% endhighlight %}
Seqeuence와 []int 두 타입이 이름을 제외하고 동일하기 떄문에 서로 변환할 수 있는 것이다. 이러한 타입 변환은 새로운 값을 만들어 내지 않는다.
Go에서 다른 메소드를 사용하기 위해 타입 변환을 사용하는 것인 Go style이다. 예로써, `sort.IntSlice`를 사용해 위의 전체 코드를 다음과 같이 간소화할 수 있다.
{% highlight go %}
type Sequence []int

func (s Sequence) String() string {
    sort.IntSlice(s).Sort()
    return fmt.Sprint([]int(s))
}
{% endhighlight %}
`Sequence`가 복수의 인터페이스를 구현하는 대신, 하나의 객체가 복수의 타입(Sequence, sort.IntSlice, []int)로 변환 될 수 있는 점을 이용하고 있다.

#### Interface conversions and type assertions
---
[**Type Switch**](#type-switch)는 conversion의 하나의 형태로, 인터페이스를 받아 switch문의 각 case에 맞게 타입을 변환한다. 아래의 코드는 value가 이미 string인 경우 인터페이스의 실제 string 값을, value가 String 메소드를 가지고 있을 경우에는 String 메소드를 실행한 결과를 리턴한다.

{% highlight go %}
type Stringer interface {
    String() string
}

var value interface{} 
switch str := value.(type) {
case string:
    return str
case Stringer:
    return str.String()
}
{% endhighlight %}
첫번째 case는 구체적인 값을 찾은 경우고, 두번째 case는 인터페이스를 또 다른 인터페이스르 변환한 경우이다.
오로지 한 타입에만 관심이 있는 경우, 예를 들어, value가 string을 저장하는 것을 알고 있는 상태라면 `Type Assertion`을 쓸 수 있다. `Type Assertion`은 인터페이스를 명시하는 타입의 값으로 추출한다.
{% highlight go %}
//value.(typeName)
str := value.(string)
{% endhighlight %}
하지만 값이 string을 가지고 있지 않은 경우, 런타임 에러가 발생한다. 이러한 상황을 위해 `"Comma, ok"` 관용구를 사용해 값을 검사한다.
{% highlight go %}
if str, ok := value.(string); ok {
    return str
} else if str, ok := value.(Stringer); ok {
    return str.String()
}
{% endhighlight %}

#### Generality
---
어떤 타입이 인터페이스를 구현하기 위해서만 존재한다면(인터페이스를 제외하고 어떠한 메소드도 노출시키지 않은 경우), 타입을 노출 시킬 필요가 없다. 이는 인터페이스에 추상화된 기능외에 다른 어떠한 기능도 제공하지 않는다는 것을 확실하게 전달한다.
이 경우, constructor는 구현 타입이 아닌 인터페이스 값을 반환해야 한다. 예로써, 해쉬 라이브러리인 `crc32.NewIEEE`와 `adler32.New`는 `hash.Hash32`를 반환한다. `crc-32`를 `adler-32`로 교체하려면 단순히 consturctor call만 바꿔주면 되며, 그 외의 코드들은 알고리즘 변화에 영향을 받지 않는다.

#### Interfaces and methods
---
다음과 같은 예에서 `Handler`를 구현하는 어느 객체든 HTTP request를 서비스할 수 있다.

{% highlight go %}
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
{% endhighlight %}
`ResponseWriter` 또한 클라이언트에 응답을 보내는데 필요한 메소드들을 제공하는 인터페이스이다. 이 메소드들은 표준 Write 메소드들을 포함하기 때문에, `http.ResponseWriter`은 `io.Writer`가 사용될 수 있는 곳이면 어디에나 사용할 수 있다.

페이지 방문 수를 세는 handler의 예시.
{% highlight go %}
type Counter struct {
    n int
}

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    ctr.n++
    fmt.Fprintf(w, "counter = %d\n", ctr.n)
}
{% endhighlight %}
(`Fprintf`가 `http.ResponseWriter`를 출력할 수 있다.)

서버를 구동할 때 사용한 Argument들을 보여주려는 경우를 가정해보자.
{% highlight go %}
func ArgServer() {
    fmt.Println(os.Args)
}
{% endhighlight %}
이를 HTTP 서버로 바꾸려면, 포인터와 인터페이스만 뺴고 어떠한 타입에나 메소드를 정의할 수 있다는 점을 이용해, 함수에 메소드를 쓸 수 있다. `http` 패키지에 다음과 같은 코드가 있다.
{% highlight go %}
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(w, req).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, req *Request) {
    f(w, req)
}
{% endhighlight %}
HandlerFunc는 어댑터로서, HandlerFunc(f)는 f를 call하는 Handler 객체이다. `HandlerFunc`는 `ServeHTTP` 메소드를 가지는 타입으로, 이 타입은 HTTP request에 응답하는 서비스를 제공한다. 
ArgServer를 HTTP로 만들기 위해, 적절한 signature를 가지도록 한다.
{% highlight go %}
// Argument server.
func ArgServer(w http.ResponseWriter, req *http.Request) {
    fmt.Fprintln(w, os.Args)
}
{% endhighlight %}
이제 ServeHTTP를 사용하기 위해 ArgServer로 바꿀 수 있다.
{% highlight go %}
// Argument server.
func ArgServer(w http.ResponseWriter, req *http.Request) {
    fmt.Fprintln(w, os.Args)
}
{% endhighlight %}
누군가가 /args를 방문하면 HTTP 서버는 HandlerFunc 타입의 ServeHTTP 메소드를 call하고, 리시버로 ArgServer를 사용한다. 이 후 HandlerFunc.ServerHTTP안에서 f(w,req)를 call한다.

## The blank identifier
---
`for range`, `map` 설명에서 보았듯, 공백 식별자는 그 어떠한 타입에 대해서나 할당될 수 있다. 
---
#### The blank identifier in multiple assignment
---
좌변에 여러개의 값을 할당하는데, 그 중 사용되지 않는 변수가 있을 경우, 공백 식별자를 두어 변수를 생성할 필요가 없게 한다.
{% highlight go %}
if _, err := os.Stat(path); os.IsNotExist(err) {
	fmt.Printf("%s does not exist\n", path)
}
{% endhighlight %}
에러를 무시하기 위해서 에러값을 버리는 코드는 잘못된 방법이다. 에러를 리턴하는 함수라면, 항상 에러를 확인해야 한다.
{% highlight go %}
// Bad! This code will crash if path does not exist.
fi, _ := os.Stat(path)
if fi.IsDir() {
    fmt.Printf("%s is a directory\n", path)
}
{% endhighlight %}

#### Unused imports and variables
---
Go에서는 미사용 import를 허용하지 않는데, 개발 중에 종종 사용하지 않는 임포트와 변수들이 생길 수 있다. 컴파일을 위해 나중에 다시 필요할 import를 지우는 것은 비효율적일 수 있고, 공백 식별자는 이를 피할 수 있도록 한다.
예로써, 미사용 임포트(`fmt`,`io`)와 변수(`fd`)를 가지고 있는 예제가 있다. 이는 컴파일 되지 않는다.
{% highlight go %}
package main

import (
    "fmt"
    "io"
    "log"
    "os"
)

func main() {
    fd, err := os.Open("test.go")
    if err != nil {
        log.Fatal(err)
    }
    // TODO: use fd.
}
{% endhighlight %}
미사용된 임포트와 변수를 공백 식별자에 할당해 컴파일 에러를 해결할 수 있다.
{% highlight go %}
package main

import (
    "fmt"
    "io"
    "log"
    "os"
)

var _ = fmt.Printf // For debugging; delete when done.
var _ io.Reader    // For debugging; delete when done.

func main() {
    fd, err := os.Open("test.go")
    if err != nil {
        log.Fatal(err)
    }
    // TODO: use fd.
    _ = fd
}
{% endhighlight %}
미사용 import 구문 할당은 임포트 구문 바로 다음에 위치시키며, 주석을 달아주어, 후에 코드 정리를 상기시키도록 한다.

#### Import for side effect
---
사용하지는 않는데, Side effect를 위해 패키지를 임포트할 수 있다. 예로써, `net/http/pprof` 패키지는 패키지의 init() 함수에서 디버깅 정보를 제공하는 HTTP 핸들러를 등록한다.
{% highlight go %}
// nethttp/pprof.go
func init() {
	http.HandleFunc("/debug/pprof/", Index)
	http.HandleFunc("/debug/pprof/cmdline", Cmdline)
	http.HandleFunc("/debug/pprof/profile", Profile)
	http.HandleFunc("/debug/pprof/symbol", Symbol)
	http.HandleFunc("/debug/pprof/trace", Trace)
}
{% endhighlight %}
Side effect만을 위해 패키지를 임포트하기 위해 임포트한 패키지를 공백 식별자로 바꾼다.

{% highlight go %}
import _ "net/http/pprof"
{% endhighlight %}
이 파일에서 패키지가 이름을 가지고 있지 않기 때문에 사용될 수 없고, 이로써 패키지가 Side effect를 위해 임포트되었음을 명학하게 할 수 있다.

#### Interface checks
---
타입은 인터페이스를 구현했다는 것을 명시적으로 선언할 필요가 없지만, 타입은 인터페이스의 메소드를 구현함으로써, 인터페이스를 구현한다. 대부분의 인터페이스 변환은 static하며 컴파일 도중 검사가 이루어진다. 예로써, `*os.File`이 `io.Reader` 인터페이스를 구현하지 않았는데 `io.Reader`를 인자로 받는 함수에 전달하면 컴파일이 되지 않는다.
런타임 때 인터페이스 검사가 이루어지는 경우도 있다. 예로써, `encoding/json`패키지의 `Marshaler` 인터페이스가 있다. JSON 인코더가 인터페이스를 구현한 타입 값을 받을 때, 표준 변환을 진행하는 대신 타입 값의 marsharling 메소드를 실행한다. 인코더는 런타임에서 타입 단언을 통해 프로퍼티를 검사한다.
{% highlight go %}
if _, ok := val.(json.Marshaler); ok {
    fmt.Printf("value %v of type %T implements json.Marshaler\n", val, val)
}
{% endhighlight %}
패키지가 인터페이스를 만족시키는 타입을 구현했는지 보장하기 위해 위와 같은 방식을 쓸 수 있다. 컴파일러가 이를 자동으로 확인하지는 않는다. 타입이 인터페이스를 만족하는데에 실패하면 JSON 인코더는 실행되지만 구현체를 사용할 수 없게되는 것이다. 인터페이스 구현을 보장하기 위해 패키지 안에서 blank identifier를 전역으로 선언한다.
{% highlight go %}
var _ json.Marshaler = (*RawMessage)(nil)
{% endhighlight %}
위의 선언에서 `*RawMessage`를 `Marshaler`로 변환시키는 할당으로 Marshaler를 구현할 것을 요구하고 있다. 이는 컴파일시 검사된다.
{% highlight go %}
type RawMessage json.RawMessage
var _ json.Marshaler = (*RawMessage)(nil)
func (r *RawMessage) MarshalJSON()([]byte,error) {
	...
}
{% endhighlight %}
위에서 공백 식별자는 선언 자체가 변수를 만드는 것이 아니라 오로지 타입 검사를 위해서만 선언되었음을 알려주고 있다.

## Embedding
---
Go는 subclassing을 제공하지 않으나, struct나 인터페이스에 Type Embedding을 통해 구현체의 일부를 빌릴 수 있다. `io.Reader`와 `io.Writer` 인터페이스로 인터페이스 임베딩을 설명할 수 있다.
```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}
```
`io`패키지의 `io.ReadWriter`는 Read와 Write를 모두 가지고 있다. 두 메소드를 `io.ReadWriter`에 새롭게 정의 할 수도 있지만, 더 좋은 방법은 위의 두 인터페이스를 임베딩하도록 인터페이스를 만드는 것이다.
```go
type ReadWriter interface {
    Reader
    Writer
}
```
인터페이스 임베딩을 위해서는 인터페이스 간에 서로 공통된 메소드가 없어야 한다.

`bufio`패키지에는 `bufio.Reader`와 `bufio.Writer` 두가지 구조체가 있다. `bufio`는 마찬가지로 임베딩을 이용해 reader와 writer를 하나의 struct에 embed해 reader/writer를 구현하고 있다 **struct안에 필드 이름이 없는 타입을 나열한다.**
```go
// ReadWriter stores pointers to a Reader and a Writer.
// It implements io.ReadWriter.
type ReadWriter struct {
    *Reader  // *bufio.Reader (o)
    *Writer  // *bufio.Writer (o)
	//reader	*Reader (x)
	//writer	*Writer (x)
}
```
(x)로 마킹한 코드처럼 선언하게 되면, reader와 writer가 가지고 있는 메서드를 사용해 `io`를 충족시키기 위해서 forwarding method를 따로 작성해야하는 오버헤드가 발생한다.
```go
func (rw *ReadWriter) Read(p []byte) (n int, err error) {
    return rw.reader.Read(p)
}
```
이를 피하기 위해 struct를 직접 임베딩한다. 임베딩된 타입의 메서드들은 자동으로 포함되며, `bufio.ReadWriter`는 `bufio.Reader`와 `bufio.Writer`의 메서드를 모두 가지게됨과 동시에, `io.Reader`, `io.Writer`, `io.ReadWriter` 인터페이스도 충족시키게 되는 것이다.  
타입을 임베드하면, 타입의 메서드들이 외부 타입의 메서드가 되지만, 호출된 메서드의 리시버는 내부 타입이다. `bufio.ReadWriter`의 Read 메서드가 호출될 때, forwarding method를 사용한 것과 똑같은 효과가 있다. 리시버는 ReadWriter의 reader필드이지, ReadWriter가 아닌 것이다.

```go
type Job struct {
    Command string
    *log.Logger
}
```
위에서 Job 타입은 `*log.Logger`에 있는 Log, Logf와 같은 메서드를 가진다. Logger에 이름을 줄 필요가 없고, Job 인스턴스가 초기화되면 Job 인스턴스에서 직접 Log를 사용할 수 있다.
```go
job.Log("starting now...")
```
Logger는 Job struct의 일반 필드이기 때문에 다음과 같이 초기화 할 수 있다.
```go
func NewJob(command string, logger *log.Logger) *Job {
    return &Job{command, logger}
}
```
또는 Composite literal을 통해,
```go
job := &Job{command, log.New(os.Stderr, "Job: ", log.Ldate)}
```
임베드드된 필드를 직접 명시해, 임베딩된 메소드를 수정할 수도 있다. 이때는 필드의 패키지명을 뺀 타입명을 필드 네임으로 사용한다. log.Logger의 경우, 패키지명 log를 뺀 job.Logger라고 쓰면 된다.
```go
func (job *Job) Logf(format string, args ...interface{}) {
    job.Logger.Logf("%q: %s", job.Command, fmt.Sprintf(format, args...))
}
```
이름이 충돌하는 문제를 해결하는 두 가지 방법이 있다.  
첫번째로, 필드와 메소드는 타입내에 더 깊숙하게 중첩되어 있는 다른 X를 가린다. log.Logger가 Command라는 필드나 메소드를 가지고 있다면, Job의 Command 필드가 우세하다.  
두번째로, 같은 이름이 같은 레벨에 있다면 에러가 발생한다. 만약 Job이 Logger라는 다른 필드나 메소드를 가지고 있다면, `log.Logger`를 임베딩하는 것은 잘못된 방식이다. 하지만 이 이름이 사용되지 않는다면 문제가 없다.

## Errors
---
Library routine[^libraryroutine]은 에러 징후가 보인다면 caller에게 자주 리턴해주어야 한다. 이전에 언급했듯, Go의 Multivalue return은 상세한 에러 내용을 제공하기 쉽게 한다. 예로써, `os.Open`은 fail시 `nil`뿐만 아니라, 무엇이 잘못되었는지에 대한 에러 내용도 리턴한다.
[^libraryroutine]: 라이브러리 루틴은 일반적으로 발새하는 문제나 작업을 처리하도록 설계된 디버깅된 code block(subroutines, procedure, function).  
관례적으로, 에러는 간단한 built-in 인터페이스 타입인 error를 가진다.
```go
type error interface {
	Error() string
}
```
라이브러리 작성자는 좀 더 풍부한 모델을 이용해 이 인터페이스를 자유롭게 구현하면서, 에러를 보여줄 뿐만 아니라 몇몇 context도 제공한다. 위에서 언급했듯이, `os.Open`은 `*os.File` value와 error를 동시에 리턴한다. 파일이 성공적으로 열리면, 에러는 `nil`이 되고, 그렇지 않으면 `os.PathError` 값을 가진다.
```go
// PathError records an error and the operation and
// file path that caused it.
type PathError struct {
    Op string    // "open", "unlink", etc.
    Path string  // The associated file.
    Err error    // Returned by the system call.
}

func (e *PathError) Error() string {
    return e.Op + " " + e.Path + ": " + e.Err.Error()
}
```
`os.PathError`의 `Error`메서드는 다음과 같은 string을 생성한다.
```go
open /etc/passwx: no such file or directory
```
이 에러는 문제가 발생할 수 있는 파일 이름, 연산, 이를 발생시키는 운영체제 에러를 포함하고 있다. 이는 문제를 발생시킨 시스템콜로부터 멀리 떨어진 곳에서 보여진다하여도 유용하다. 단순히 `no such file or directory`를 보여주는 것보다 많은 정보를 제공한다.
가능하다면 에러는 에러가 발생된 명령이나 패키지를 접두어로 쓰는 것처럼 에러의 source를 파악할 수 있도록 해야한다.
상세한 에러에 대해 관심이 있는 Caller는 추가적은 정보를 얻기 위해 Type switching이나 Type assertion을 사용할 수 있다.
```go
for try := 0; try < 2; try++ {
    file, err = os.Create(filename)
    if err == nil {
        return
    }
    if e, ok := err.(*os.PathError); ok && e.Err == syscall.ENOSPC {
        deleteTempFiles()  // 공간을 확보한다.
        continue
    }
    return
}
```
두 번째 if절에서 `err` 인터페이스를 type assertion을 통해 `os.PathError` 타입으로 변환한다. 실패시 `ok`는 false가 되고, e는 nil이 된다. 성공시 `ok`는 true가 되고, 에러가 `*os.PathError` 타입이 된다. `e`에서 구조체 필드를 통해 에러에 대한 더 많은 정보를 확인할 수 있다.

#### Panic
---

#### Recover
---
