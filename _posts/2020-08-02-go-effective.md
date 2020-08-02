---
layout: post
comment: false
title: Golang Basic
date: 2020-03-17
category: "Golang"
tags: [Golang]
author: Jaehyun Lee
---

##### [Effective Go](https://golang.org/doc/effective_go.html)의 개인 참고용 번역 및 요약본입니다.
---

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
- getter의 이름에 Get을 넣는 것은 go style이 아님. 만약에 owner 라는 private 필드를 가지고 있다면 getter는 GetOnwer가 아닌 Owner라고 네이밍, setter는 SetOnwer라고 네이밍 하면 됨.
{% highlight go %}
owner := obj.Owner()
if owner != user {
	obj.SetOwner(user)
}
{% endhighlight %}
