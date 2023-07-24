# Chapter 2 - 동작 파라미터화 코드 전달하기

이 장에서 살펴볼 내용은 다음과 같다.
  
* 변화하는 요구사항에 대응
* 동작 파라미터화 
* 익명 클래스
* 람다 표현식 미리보기
* 실전 예제: Comparator, Runnable, GUI

어떤 상황에서 일을 하든 소비자 요구사항은 항상 바뀐다. 변화하는 요구사항은 소프트웨어 엔지니어링에서 피할 수 없는 문제다. 이렇게 시시각각 변하는 사용자 요구사항에 어떻게 대응해야 할까?

특히 우리의 엔지니어링적인 비용이 가장 최소화될 수 있으면 좋을 것이다. 그뿐 아니라 새로 추가한 기능은 쉽게 구현할 수 있어야 하며 장기적인 관점에서 유지보수가 쉬워야 한다. **동작 파라미터화**를 이용하면 자주 바뀌는 요구사항에 효과적으로 대응할 수 있다.

동작 파라미터화란 아직은 어떻게 실행할 것인지 결정하지 않은 코드 블록을 의미한다. 이 코드블록은 나중에 프로그램에서 호출한다. 즉, 코드 블록의 실행은 나중으로 미뤄진다. 예를 들어 나중에 실행될 메서드의 인수로 코드 블록을 전달할 수 있다.결과적으로 코드 블록에 따라 메서드의  동작이 파라미터화 된다.

기존의 자바는 동작 파라미터화를 추가하려면 쓸데없는 코드가 늘어난다. 자바 8은 람다 표현식으로 이 문제를 해결한다.

## 2.1 변화하는 요구사항에 대응하기
변화에 대응하는 코드를 구현하는 것은 어려운 일이다. 일단 하나의 예제를 선정한 다음 예제 코드를 점차 개선하면서 유연한 코드를 만드는 모범사례를 보여줄 것이다.

### 2.1.1 첫번째 시도: 녹색 사과 필터링
```java
enum Color { RED, GREEN } //사과 색을 정의하는 Color num

//==첫 번째 시도 결과 코드==//
public static List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (GREEN.equals(apple.getColor())) {
            result.add(apple);
        }
  }
  return result;
}
```
위 코드는 녹색 사과를 선택하는 데 필요한 조건을 가리킨다. 녹색이 아닌 빨간 사과를 필터링을 하려면 어떻게 해야할까?

-> 새로운 메서드를 복사해 만들고 if문의 조건을 바꾸면 되지만 더 다양한 색을 필터링하는 등의 변화에는 적절한 대응이 어렵다. 이런 상황에서는 좋은 규칙이 있다.

####  거의 비슷한 코드가 반복 존재한다면 그 코드를 추상화한다. 

### 2.1.2두 번째 시도 : 색을 파라미터화
어떻게 해야 filterGreenApples의 코드를 반복 사용하지 않고 빨간 사과를 필터링 하는 함수를 구현할 수 있을까?

-> 색을 파라미터화 할 수 있도록 메서드에 파라미터를 추가하면 변화하는 요구사항에 좀 더 유연하게 대응할 수 있을 것이다.

```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, **Color color**) {
	List<Apple> result = new ArrayList<>();
	for (Apple appe: inventory) {
		if **(apple.getColor().equals(color))** {
			result.add(people);
		}
	}
	return result;
}

// 다음과 같이 호출할 수 있게 되었다.
List<Apple> greenApples = filterApplesByColor(inventory, GREEN);
List<Apple> redApples = filterApplesByColor(inventory, RED);
```
하지만 색뿐만 아니라 무게를 기준으로 분류해야하는 상황이 온다면? 무게 정보 파라미터 또한 추가해주어야 한다.

```java
public static List<Apple> filterApplesByWeight(List<Apple> inventory, **int weight**) {
	List<Apple> result = new ArrayList<>();
	for (Apple appe: inventory) {
		if **(apple.getWeight() >= weight)** {
			result.add(people);
		}
	}
	return result;
}
```
위와 같은 코드도 좋은 해결책이라고 할 수 있다. 하지만 구현 코드를 자세히 보면 색 필터링 코드와 대부분 중복된다는 것을 볼 수 있다.

이는 소프트웨어 공학의 DRY(don't repeat youself)원칙을 어기는 것이다. 탐색 과정을 고쳐서 성능을 개선하려면 무슨 일이 일어날까?
 
 -> 한 줄이 아닌 메서드 전체 구현을 고쳐야 한다. 즉, 엔지니어링적으로 비싼 대가를 치러야 한다.

 ### 2.1.3 세 번째 시도: 가능한 모든 속성으로 필터링
 모든 속성을 메서드 파라미터로 추가하고, 판단의 기준이 되는것을 가리키는 flag를 추가한 모습이다.
 ```java
public static List<Apple> filterApples(List<Apple> inventory,Color color, int weight, boolean flag) {
	List<Apple> result = new ArrayList<>();
	for (Apple appe: inventory) {
		if (apple.getColor().equals(color)) {
			result.add(people);
		}
	}
	return result;
}
// 위 메서드를 다음처럼 사용할 수 잇다.
List<Apple> greenApples = filterApples(inventory, GREEN, 0, true);
List<Apple> heavyApples = filterApples(inventory, null, 150, false); 
```
매우 좋지않은 코드임이 분명하다. true와 false는 무엇을 의미하며, 요구사항 변화에 유연하게 대응할 수 없다. (사과의 크기, 모양, 출하지 등으로 필터링해야한다면? 심지어 녹색 사과 중에 무거운 사과를 필터링하고 싶다면?)

지금까지는 문자열, 정수, 불리언 등의 값으로 파라미터화 하였고 문제가 잘 정의되어있는 상황에서는 이 방법이 잘 동작할 수 있다. 하지만 filterApples에 어떤 기준으로 사과를 필터링할 것인지 효과적으로 전달할 수 있다면 더 좋을 것이다. 

## 2.2 동작 파라미터화
앞서 2.1절에서 우리는 파라미터를 추가하는 방법이 아닌 변화하는 요구사항에 좀 더 유연하게 대응할 수 있는 방법이 절실하다는 것을 확인했다. 사과의 어떤 속성에 기초해서 불리언 값을 반환 (예를 들어 사과가 녹색인가? 150그램 이상인가?)하는 방법이 있다. 참 또는 거짓을 반환하는 함수를 **프레디케이트**라고 한다.

선택 조건을 결정하는 인터페이스를 정의하자. 또한 다양한 선택 조건을 대표하는 여러 버전의 ApplePredicate를 정의할 수 있다.
```java
public interface ApplePredicate {
    boolean test (Apple apple);
}
// 무거운 사과만 선택하도록 하는 predicate
public class AppleHeavyWeightPredicate implements AppplePredicate {
	public boolean test(Apple apple) {
		return apple.getWeight() > 150;
	}
}

// 녹색 사과만 선택하도록 하는 predicate
public class AppleColorPredicate implements AppplePredicate {
	public boolean test(Apple apple) {
		return GREEN.equals(apple.getColor());
	}
}
```

ApplePredicate는 사과를 선택하는 전략을 캡슐화하고 있고, 전략에 따라 AppleHeavyWeightPredicate, AppleColorPredicate를 구현하도록 되어 있는데 이를 전략 디자인 패턴(strategy design pattern) 이라고 한다.

전략 디자인 패턴은 각 알고리즘(전략이라 불리는)을 캡슐화하는 알고리즘 패밀리를 정의해둔 다음에 런타임에 알고리즘을 선택하는 기법이다. 이러한 전략 패턴은 객체의 행위를 동적으로 바꾸고 싶은 경우 직접 행위를 수정하지 않고 전략을 바꿔주기만 함으로써 행위를 유연하게 확장할 수 있도록 한다.

이렇게 만들어진 ApplePredicate 는 filterApples에서 파라미터로 받는다. 이렇게 동작 파라미터화, 즉 메서드가 다양한 동작(또는 전략)을 받아서 내부적으로 다양한 동작을 수행할 수 있다.

### 2.2.1 네 번째 시도 : 추상적 조건으로 필터링
이제 filterApples 메서드가 ApplePredicate 객체를 인수로 받도록 고치자. 이렇게 하면 filterApples 메서드 내부에서 컬렉션을 반복하는 로직과 컬렉션의 각 요소에 적용할 동작을 분리할 수 있다는 점에서 소프트웨어 엔지니어링적으로 큰 이득을 얻는다.
```java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
	List<Apple> result = new ArrayList<>();
	for(Apple apple : inventory) {
		if(p.test(apple)) {
			result.add(apple);
		}
	}
	return result;
}
```
#### 코드/동작 전달하기
프리디케이트 객체로 사과 검사 조건을 캡슐화 했 다. 첫 번째 코드에 비해 더 유연한 코드를 얻었으며 동시에 가독성도 좋아졌을 뿐 아니라 사용하기도 쉬워졌다. 이제 필요한 대로 다양한 ApplePredicate를 만들어서 filterApples 메서드로 전달할 수 있다.

```java
public class AppleRedAndHeavyPredicate implements ApplePredicate {
	public boolean test(Apple apple) {
		return RED.equals(apple.getColor()) && apple.weight() > 150;
	}
}

List<Apple> redAndHeavyApples = filterApples(inventory, new AppleRedAndHeavyPredicate());
```
우리가 전달한 ApplePredicate 객체에 의해 filterApples 메서드의 도작이 결정된다. 즉, 우리는 filterApples 메서드의 동작을 파라미터화한 것이다.

위 예제에서 가장 중요한 구현은 test 메서드다. filterApples 메서드의 새로운 동작을 정의하는 것이 test 메서드다. 하지만 안타깝게도 메서드는 객체만 인수로 받으므로 test 메서드를 ApplePredicate 객체로 감싸서 전달해야 한다. test 메서드를 구현하는 객체를 이용해서 불리언 표현식 등을 전달할 수 있으므로 이는 '코드를 전달'할 수 있는 것이나 다름 없다.

→ 추후 람다를 이용해서 여러 개의 ApplePredicate 클래스를 정의하지 않고도 표현식을 filterApples 메서드로 전달하는 방법을 설명한다.

#### 한 개의 파라미터, 다양한 동작
컬렉션 탐색 로직과 각 항목에 적용할 동작을 분리할 수 있다는 것이 동작 파라미터화의 강점이다. 따라서 한 메서드가 다른 동작을 수행하도록 재활용 할 수 있다. 따라서 유연한 API를 만들 때 동작 파라미터화가 중요한 역할을 한다.