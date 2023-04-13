---
title: name mangling과 접근 제어자
date: 2023-02-12 15:14:35 +0900
categories: [ Python ]
tags: [ Python, Name Mangling, Access Modifier ]
comments: true
---
python에서 접근 제어자를 지정하는 방법과 Naming Mangling에 대한 글

## 개요
Java나 CPP와 같은 OOP 언어에서 public, private, protected와 같은 keyword로 멤버 변수의 접근을 제어한다.  
python은 인터프리터 언어로 따로 이런 접근 제어자(Access Modifier) keyword가 존재하지 않는다.  
해당 글에서는 python에서는 어떻게 접근 제어를 하는지 알아본다.

## Underscore와 Name Mangling

먼저 결론부터 말하면, python에서는 접근 제어자로 underscore기호(`_`)를 사용한다.

underscore가 0개면 public  
underscore가 1개면 protected  
underscore가 2개면 private
```python
>>> class Student:
        def __init__(self, name, age, address):
            self.name = name            # public
            self._age = age             # protected
            self.__address = address    # private
    
>>> s = Student('jason', 16, '375 North St')
>>> print(s.name)
jason

>>> print(s._age)
16

>>> print(s.__address)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Student' object has no attribute '__address'

>>> print(dir(s))
['_Student__address', '__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', '_age', 'name']
```
위의 실행 결과를 보면 `__address` 접근 시 `no attributee error`가 발생하는 것을 알 수있다.  
dir 함수를 통해서 Student 인스턴스의 attribute를 출력해보면 `name`, `_age`는 존재하지만 `__address`는 존재하지 않는다.

그런데 attribute 배열의 가장 첫번째 값을 보면 `_Student__address`라는 값이 존재한다.  
private member는 python의 name mangling을 통해서 `_클래스이름__멤버이름`형태로 변환된다.
이와 같이 name mangling을 통해서 외부에서 private member로 선언한 변수이름(위 예시에서 __address) 그대로 접근을 막아준다.

또한 private member의 경우, 상속 관계에 있을 때 자식 클래스에서 부모 클래스의 private member를 접근하거나, override가 불가능하다.
아래 예시를 통해 name mangling을 통해서 이러한 특성이 어떻게 지켜지는지 확인해보자.

```python
>>> class CollegeStudent(Student):
        def __init__(self, name, age, address, college):
            super().__init__(name, age, address)
            self.name = name            
            self._age = age             
            self.__address = address
            self.college = college
    
>>> c = CollegeStudent('jason', 20, '375 North St' ,'Boston')
>>> print(dir(c))
['_CollegeStudent__address', '_Student__address', '__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', '_age', 'college', 'name']
```

CollegeStudent가 Student를 상속받는 구조이다.  

CollegeStudent의 attribute 값을 보면 `_CollegeStudent__address`, `_Student__address` 두가지 값이 존재한다.  
즉 CollegeStudent 클래스의 self.__address가 부모 클래스의 `_Student__address`를 참조하는 것이 아니라 CollegeStudent 클래스만의 __address 변수에 대한 선언이 되고 
name mangling을 통해 `_CollegeStudent__address`의 형태로 변환되어 추가가 된다는 것을 알 수 있다.  

해당 글에서는 Field에 대한 예시로 설명을 했지만, 메소드도 마찬가지이다. 메소드 이름 앞에 underscore 기호를 이용하여 접근제어가 가능하고 name mangling이 적용된다.


## Name Mangling이 적용된 변수에 대한 접근은 지양하자
위의 내용들을 통해서 underscore와 name mangling를 통해 python에서는 접근 제어가 어떻게 가능한지 확인할 수 있었다.

하지만 private member에 name mangling이 적용되었다고 하더라도, 아래와 같이 직접 접근 및 수정이 가능하다.
```python
>>> s = Student('jason', 16, '375 North St')
>>> s._Student__address
'375 North St'
>>> s._Student__address = 'changed'
>>> s._Student__address
'changed'
```

하지만 python 개발에 있어서 name mangling가 적용된 변수에 대해 예시처럼 접근하는 방식은 사용하면 안된다.  
python에 접근제어를 위해 name mangling를 도입 및 적용한 것인데, 위와 같은 행위는 접근제어를 깨트리는 행위와 같기 때문이다.




## Refercence
* <https://www.geeksforgeeks.org/name-mangling-in-python//>
* <https://medium.com/analytics-vidhya/python-name-mangling-and-how-to-use-underscores-e67b529f744f/>