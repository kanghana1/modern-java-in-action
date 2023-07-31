# Chapter 3 - 람다 표현식(2)

## 3.5 형식 검사, 형식 추론, 제약
컴파일러가 람다의 형식을 어떻게 확인하는지, 피해야 할 사항은 무엇인지 등 깊이 있게 살펴본다. 람다로 함수형 인터페이스의 인스턴스를 만들 수 있다고 했다. 람다 표현식 자체에는 람다가 어떤 함수형 인터페이스를 구현하는의 정보가 포함되어 있지 않기에 제대로 이해하려면 람다의 실제 형식을 파악해야 한다.

### 3.5.1 형식 검사
람다가 사용되는 context를 이용해서 람다의 형식을 추론할 수 있다. 어떤 콘텍스트(람다가 전달될 메서드 파라미터나 람다가 할당되는 변수 등)에서 기대되는 람다 표현식의 형식을 **대상 형식** 이라고 부른다. 람다 표현식을 사용할 때 실제 어떤 일이 일어나는지 보여주는 예제를 확인하자.
```java
List<Apple> heavierThan150g = 
	filter(inventory, (Apple apple) -> apple.getWeight() > 150);
```
형식 확인 과정을 살펴보자.

1. filter 메서드의 선언을 확인한다.
2. filter 메서드는 두 번째 파라미터로 Predicate<Apple\> 형식(대상 형식)을 기대한다.
3. Predicate<Apple\>은 test라는 한 개의 추상 메서드를 정의하는 함수형 인터페이스다.
4. test메서드는 Apple을 받아 boolean을 반환하는 함수 디스크립터를 묘사한다.
5. filter 메서드로 전달된 이수는 이와 같은 요구사항을 만족해야 한다.
   
과정을 그림으로 나타내면 다음과 같다.
```java
filter(inventory, (Apple a) -> a.getWeight() > 150);
//1. 람다가 사용된 콘텍스트는 무엇인가? 우선 filter의 정의를 확인하자
-> filter(List<Apple>inventory, Predicate<Apple> p)
//2. 대상 형식은 Predicate<Apple>이다.(T는 Apple로 대치)
-> 대상형식
// Predicate<Apple> 인터페이스의 추상 메서드는 무엇인가?
-> boolean test(Apple apple)
//Apple을 인수로 받아 boolean을 반환하는 test메서드이다.
-> Apple -> boolean
//함수 디스크립터는 Apple -> boolean이므로 람다의 시그니처와 일치한다.
//람다도 Apple을 인수로 받아 boolean을 반환하므로 코드 형식 검사가 성공적으로 완료한다.
-> filter(inventory, (Apple a) -> a.getWeight() > 150);
```
람다 표현식이 예외를 던질 수 있다면 추상 메서드도 같은 예외를 던질 수 있도록 throws로 선언해야 한다.

### 3.5.2 같은 람다, 다른 함수형 인터페이스
