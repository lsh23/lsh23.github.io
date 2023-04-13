---
title: metaclass
date: 2023-03-05 17:09:04 +0900
categories: [ Python ]
tags: [ Python, MetaClass ]
comments: true
---
Python의 MetaClass에 대한 글

## 개요
python metaclass에 대해 검색해보면 가장 많이 나오는 개념적인 정의는 다음과 같다.   
`python의 metaclass는 클래스를 만드는 클래스이다.`  
해당 글에서는 클래스를 만든다라는 것에 대한 의미에 대해 간단하게 설명한다.

## Python에서 클래스는 객체이다.
```shell
>>> class MyClass:
...     pass
>>> print(MyClass) # 클래스 출력
<class '__main__.MyClass'>

>>> mc = MyClass() # 클래스 instance 출력
>>> print(mc)
<__main__.MyClass object at 0x100674490>
```

위의 코드 실행을 보면 python에서는 class 자체를 출력할 수 있다. java와 같은 언어에서 위 처럼 클래스 자체를 출력하려고 하면, 컴파일단에서 에러가 날 것이다.
python에서는 모든 것이 object(객체)이다라고 하는 말을 들어봤을 텐데,
이처럼 python에서는 instance를 찍어내는 class 역시 object(객체)이다.

## 클래스를 만든다 == metaclass의 인스턴스는 클래스이다.
python에서 `__class__`라는 매직 메소드로 객체가 어떤 클래스로 부터 만들어졌는지 확인할 수 있다.
```shell
>>> a = 1
>>> a.__class__
<class 'int'>

>>> b = "str"
>>> b.__class__
<class 'str'>

>>> c = list()
>>> c.__class__
<class 'list'>

>>> mc.__class__
<class '__main__.MyClass'>
```
위의 코드 실행을 보면,    
a는 int class로 부터 만들어진 객체이고,  
b는 str class로 부터 만들어진 객체이고,  
c는 list class로 부터 만들어진 객체이고,  
mc는 위의 예제에 우리가 선언한 MyClass로 부터 만들어진 객체임을 확인할 수 있다.

글 첫부분에서 **Python에서 클래스는 객체이다.** 라는 것을 우리는 확인했다.  
그렇다면 int, str, list, MyClass 각 클래스는 어떤 클래스로 부터 만들어진 객체인지 확인해보자.

```shell
>>> a.__class__.__class__  # int.__class__ 와 동일
<class 'type'>
>>> b.__class__.__class__  # str.__class__ 와 동일
<class 'type'>
>>> c.__class__.__class__  # list.__class__ 와 동일
<class 'type'>
>>> mc.__class__.__class__ # MyClass.__class__ 와 동일
<class 'type'>
```

모든 클래스는 type이라고 하는 클래스로부터 만들어진 객체임을 확인할 수 있다.  
아마 모든 클래스의 클래스는 metaclass이다 라는 설명도 많이 접했을 텐데,  
x의 클래스는 y이다. 라는 의미는 y로 부터 x라는 객체를 만들 수 있다는 것이고,  
여기서 y가 metaclass가 되는 것이고, y로 부터 생성되는 객체인 x가 각 개별 클래스임을 이해 할 수 있다.

## type 클래스의 클래스는?
위의 예제들을 통해서 모든 클래스는 type으로 부터 만들어진 객체임을 확인했다.  

그렇다면 type 클래스의 클래스는 무엇일까? 

```shell
>>> type
<class 'type'>
>>> type.__class__
<class 'type'>
>>> type.__class__.__class__
<class 'type'>
```

type 클래스의 클래스는 type 자기자신이다. 
따라서 아래와 같이 정리 할 수 있다.
python에서 모든 클래스는 type이라는 클래스로부터 만들어지고,
모든 클래스를 생성하는 type 클래스는 python에서 metaclass를 의미한다.
또한, type 클래스는 자기자신으로 부터 만들어지므로, type 클래스는 최상위 metaclass임을 알 수 있다. 

## Refercence
* <https://realpython.com/python-metaclasses/>
