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

- if와 switch가`초기화 구문` 허용
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
