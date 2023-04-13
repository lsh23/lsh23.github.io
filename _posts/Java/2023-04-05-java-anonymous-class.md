---
title: 익명 클래스
date: 2023-04-05 17:20:25 +0900
categories: [ Java ]
tags: [ Java, Anonymous Class, Functional Interface ]
comments: true
---
익명 클래스와 익명 클래스를 람다 표현식으로 치환하는 방법과 함수형 인터페이스에 대한 글
## 익명 클래스란
익명 클래스(Anonymous Class)는 내부 클래스의 일종이면서, 단 하나의 객체만을 위한 일회용 클래스이다. 클래스의 선언과 객체 생성을 동시에 한다.  
익명 클래스를 통해서 단일 클래스 상속, 단일 인터페이스 구현한 객체를 일회성으로 생성할 수 있다.

## 일반적인 상속과, 구현
```java
class Car{
    public void start(){
        System.out.println("start");
    }
}

class MyCar extends Car{  // Car 상속
    @Override
    public void start(){
        System.out.println("My car start");
    }
}

Interface Engine{
    void start();
}

class MyEngine implements Engine{ // Engine 구현
    @Override
    public void start(){
        System.out.println("My Engine start");
    }
}

public class Main {
    public static void main(String[] args) {
        Car car = new MyCar();
        car.start();
        Engine engine = new MyEngine();
        engine.start();
    }
}
```
익명 클래스가 아닌 일반적인 상속과 구현 예시 코드다.  
MyCar는 부모 클래스 Car를 상속받고, MyEngine은 Engine 인터페이스를 구현한 클래스이다.
이렇게 상속, 구현한 클래스를 main 함수에서 new 키워드로 객체 생성 후 사용할 수 있다.

## 익명 클래스를 이용한 상속, 구현
```java
public class Main {
    public static void main(String[] args) {
        Car car = new Car(){
            @Override
            public void start(){
                System.out.println("Anonymous car start");
            }
        };
        car.start();
        Engine engine = new Engine(){
            @Override
            public void start(){
                System.out.println("Anonymous Engine start");
            }
        };
        engine.start();
    }
}

익명 클래스를 이용해서 일회성 상속, 구현한 객체 생성 예시 코드다.
아래와 같은 문법으로 익명 클래스를 사용한다.
```java
new 클래스/인터페이스이름() {
    // 클래스 상세 or 인터페이스 구현
};
```
위의 예시 코드에서 생성된 car의 start() 메소드 호출 결과는 `Anonymous car start`를 출력할 것이다.  
start() 메소드의 호출 결과가 일회성이 아닌 경우에는 익명 클래스가 아닌 일반적인 상속, 구현 클래스를 작성 후 해당 클래스 타입으로 객체를 생성해야한다.  
또한 익명 클래스는 단일 상속, 단일 구현만 가능하기 때문에 부모 클래스를 상속 받으면서 인터페이스를 구현하는 경우나, 여러개의 인터페이스를 구현하는 케이스에대해서 사용할 수 없다.

## 익명 클래스를 이용한 인터페이스 구현 - 축약

익명 클래스를 이용해서 인터페이스를 구현할 때 람다 표현식으로 코드를 축약할 수 있다.  
단, 조건이 하나 있는데 인터페이스의 추상함수가 오직 1개 있는 경우다.  

위의 예시에서 Engine 인터페이스의 경우 추상함수 start() 1개만 가지고 있기 때문에 해당 조건을 만족한다. 따라서 코드를 다음과 같이 축약할 수 있다.
```java
public class Main {
    public static void main(String[] args) {
        Engine engine = () -> {
            System.out.println("Anonymous Engine start");
        };
        engine.start();
    }
}

```
이렇게 람다식 표현으로 축약 가능한 인터페이스를 **함수형 인터페이스(Functional Interface)**라고 한다.

## 함수형 인터페이스(Functional Interface) 활용 - Comparator

자바에서 정렬시에 많이 사용 되는 Comparator 인터페이스 역시, 함수형 인터페이스다.

Comparator 소스코드를 보면 다음과 같다.
```java
@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);    // 추상 메서드
    boolean equals(Object obj); // 추상 메서드?
    ...
}
```
추상메소드로 보이는 메소드가 2개가 있다. 하지만 equals의 경우 java.lang.Object의 equals 메소드로 자동으로 오버라이딩이 되기 때문에 추상 메소드가 아니다. 

따라서 Comparator의 익명 클래스 객체를 compare에 대한 구현 부분을 람다 표현식을 사용해서 생성할 수 있다.
예시 코드는 다음과 같다.

```java

public class MyInteger {
    private int value;

    public MyInteger(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }

    @Override
    public String toString() {
        return Integer.toString(value);
    }

    public static void main(String[] args) {
        List<MyInteger> integers = Arrays.asList(new MyInteger(2), new MyInteger(3), new MyInteger(6), new MyInteger(10));
        integers.sort((a,b) -> Integer.compare(a.getValue(), b.getValue()));
        System.out.println(integers);
    }
}
```

List의 sort 함수는 Comparator를 인자로 받는다.  
따라서 위 처럼 람다 표현식으로 함수형 인터페이스의 1회성 객체를 만들어서 넣어서 사용할 수 있다.  
이처럼 함수형 인터페이스의 타입을 인자로 받아서 처리하는 함수를 사용할 때, 1회성 로직이 필요한 경우에 유용하게 사용할 수 있다.

## Refercence
* <https://mkyong.com/java8/is-comparator-a-function-interface-but-it-has-two-abstract-methods/>

