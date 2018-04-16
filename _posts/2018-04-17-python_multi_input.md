---
layout: post
title: Python 입력값 두 개 이상일 때 한번에 처리하기.
---
python 수업 중 한 line에 입력이 두 개 이상일 경우 어떻게 처리해야하는 지에 대해서는 따로 가르쳐준 적이 없네요. 한 학생이 Codeup 문제 풀 때 여기에 대한 해결책이 필요했나봐요. 그래서 올려요.
input()의 리턴 값은 String으로 String형 객체에는 split()이라는 메소드가 존재해요. #객체, 메소드# 같은 용어는 생소하겠지만, 중간고사 이후에 얕게나마 배울 듯 해요.
일단은 넘어가고, 어쨋든 input().split()을 이용해서 한번에 두 개 이상의 입력을 받을 수 있어요. 밑에 예제를 참고해보세요. 

{% highlight python %}
a,b = input('Enter the input : ').split()
print("First str : " + a)
print("Second str : " + b)
{% endhighlight %}

##실행결과##
![Image](/assets/images/python_multi.png){:style=" width: 300px; margin: 0 auto; display: block;"}

