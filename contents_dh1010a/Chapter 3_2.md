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
대상 형식이라는 특징 때문에 같은 람다 표현식이더라도 호환되는 추상 메서드를 가진 다른 함수형 인터페이스로 사용될 수 있다.

예를 들어 이전에 살펴본 Callable과 PrivilegedAction 인터페이스는 인수를 받지 않고 제네릭 형식 T를 반환하는 함수를 정의한다. 따라서 아래 두 할당문은 모두 유효한 코드이다.
```javax
Callable<Integer> c = () -> 42;
PrivilegedAction<Integer> p = () -> 42;
```

#### 다이아몬드 연산자
다이아몬드 연산자(<>)로 콘텍스트에 따른 제네릭 형식을 추론할 수 있다. 주어진 클래스 인스턴스 표현식을 두 개 이상의 다양한 콘텍스트에 사용할 수 있다. 이때 인스턴스 표현식의 형식 인수는 콘텍스트에 의해 추론된다.
```java
List<String> listOfStrings = new ArrayList<>();
List<Integer> listOfIntegers = new ArrayList<>();
```

#### 특별한 void 호환 규칙
람다의 바디에 일반 표현식이 있으면 void를 반환하는 함수 디스크립터와 호환된다.(물론 파라미터 리스트도 호환되어야 함) 예를 들어 다음 예제에서 List의 add메서드는 Consumer 콘텍스트(T -> void)가 기대하는 void 대신 boolean을 반환하지만 유효한 코드다.
```java
// Predicate는 불리언 반환값을 갖는다.
Predicate<String> p = s -> list.add(s);
// Consumer는 void 반환값을 갖는다.
Consumer<String> c = s -> list.add(s);
```

### 3.5.3 형식 추론
자바 컴파일러는 람다 표현식이 사용된 콘텍스트(대상 형식)을 이용해서 람다 표현식과 관련된 함수형 인터페이스를 추론한다. 즉, 대상 형식을 이용해서 함수 디스크립터를 알 수 있으므로 컴파이러는 람다의 시그니처도 추론할 수 있다. 결과적으로 컴파일러는 람다 표현식의 파라미터 형식에 접근할 수 있으므로 람다 문법에서 이를 생략할 수 있다.

즉, 자바 컴파일러는 다음처럼 람다 파라미터 형식을 추론할 수 있다.
```java
List<Apple> greenApples =
	filter(inventory, apple -> GREEN.equals(apple.getColoer()));// 파라미터 a에는 형식을 명시적으로 지정하지 않았다.
```
여러 파라미터를 포함하는 람다 표현식에서는 코드 가독성 향상이 더 두드러진다.
```java
// 형식을 추론하지 않음
Comparator<Apple> c = (Apple a1, Apple a2) -> a.getWeight().compareTo(a2.getWeight());
// 형식을 추론함
Comparator<Apple> c = (a1, a2) -> a.getWeight().compareTo(a2.getWeight());
```

### 3.5.4 지역 변수 사용
지금까지 살펴본 모든 람다 표현식은 인수를 자신의 바디 안에서만 사용했다. 하지만 람다 표현식에서는 익명 함수가 하는 것처럼 **자유 변수**(파라미터로 넘겨진 변수가 아닌 외부에서 정의된 변수)를 활용할 수 있다. 이와 같은 동작을 **람다 캡처링**이라고 부른다.
```java
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber);		
//portnumber 사용 = 람다 캡쳐링
```
람다는 인스턴스 변수와 정적 변수를 자유롭게 캡처(자신의 바디에서 참조할 수 있도록)할 수 있지만, 지역 변수는 명시적으로 final로 선언되어 있어야 하거나 실질적으로 final로 선언된 변수와 똑같이 사용해야만 한다.즉, 람다 표현식은 한 번만 할당할 수 있는 지역변수를 캡처할 수 있다.
```java
// 에러: 람다에서 참고하는 지역 변수는 final로 선언되거나 실질적으로 final처럼 취급되어야 한다. 아래 예제는 두번 할당하므로 컴파일이 불가능하다.
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber);
portNumber = 31337;
```

#### 지역 변수의 제약
왜 지역 변수에 이런 제약이 필요한가?
-> 인스턴스 변수와 지역 변수는 태생부터가 다르다.
* 인스턴스 변수는 힙에 저장되는 반면 지역 변수는 스택에 위치한다.
* 스택은 스레드마다 고유한 영역이므로 다른 스레드에서 접근할 수 없다. 그렇기 때문에 다른 스레드의 람다 표현식에서도 해당 값에 접근하기 위해 복사본을 제공하도록 한다. → 그래서 람다 캡쳐링이라는 용어를 사용한다.
* 람다가 스레드에서 실행된다면 변수를 할당한 스레드가 사라져서 변수 할당이 해제되었는데도 람다를 실행하는 스레드에서 해당 변수에 접근하려 할 수 있다.
* 따라서 복사본의 값이 바뀌지 않아야 하므로 지역 변수에는 한 번만 값을 할당해야 한다는 제약이 생긴 것이다.
* 또한 지역 변수의 제약 때문에 외부 변수를 변화시키는 일반적인 명령형 프로그래밍 패턴(병렬화를 방해하는 요소)에 제동을 걸 수 있다.
* **명령형 프로그래밍** - 어떻게 할지를 표현. 알고리즘을 명시하고, 목표를 명시하지 않는다.
  * for (int i = 0; i < 10; i++) {...}

#### 클로저
원칙적으로 클로저란 함수의 비지역 변수를 자유롭게 참조할 수 있는 함수의 인스턴스를 가리킨다. 예를 들어 클로저를 다른 함수의 인수로 전달할 수 있다. 클로저는 클로저 외부에 정의된 변수의 값에 접근하고, 값을 바꿀 수 있다.

자바8의 람다와 익명 클래스는 클로저와 비슷한 동작을 수행한다. 람다와 익명 클래스 모두 메서드의 인수로 전달될 수 있으므로 자신의 외부 영역의 변수에 접근할 수 있다. 다만 둘다 람다가 정의된 메서드의 지역 변수의 값은 바꿀 수 없다. 람다가 정의된 메서드의 지역 변숫값은 final이여야 한다.

지역 변수 값은 스택에 존재하므로 자신을 정의한 스레드와 생존을 같이 해야하며, 따라서 지역 변수는 final이어야 한다.
가변 지역 변수를 새로운 스레드에서 캡처할 수 있다면 안전하지 않은 동작을 수행할 가능성이 생긴다.

## 3.6 메서드 참조
메서드 참조는 특정 람다 표현식을 축약한 것이라고 생각하면 된다. 메서드 참조를 이용하면 기존의 메서드 정의를 재활용해서 람다처럼 전달할 수 있다. 때로는 람다 표현식보다 메서드 참조를 사용하는 것이 더 가독성이 좋으며 자연스러울 수 있다.

다음은 메서드 참조와 새로운 자바8  API를 활용한 정렬 예제이다.
```java
// 기존의 람다 표현식
inventory.sort((Apple a1, Apple a2) 
	-> a1.getWeight().compareTo(a2.getWeight()));

// 메서드 참조와 java.util.Comparator.comparing을 활용한 코드
inventory.sort(comparing(Apple::getWeight));
```
### 3.6.1 요약
#### 메서드 참조가 왜 중요한가?
* 메서드 참조를 이용하면 기존 메서드 구현으로 람다 표현식을 만들 수 있다. 이때 명시적으로 메서드명을 참조함으로써 **가독성을 높일 수 있다.**

#### 메서드 참조는 어떻게 활용할까?
* 메서드명 앞에 구분자(::)를 붙이는 방식으로 메서드 참조를 활용할 수 있다. Apple::getWeight는 Apple클래스에 정의된 getWeight의 메서드 참조다. 실제로 메서드를 호출하는 것은 아니므로 괄호는 필요없다.
* 결과적으로 (Apple a) -> getWeight()를 축약한 것이다.
* 메서드 참조를 새로운 기능이 아니라 하나의 메서드를 참조하는 람다를 편리하게 표현할 수 있는 문법으로 간주할 수 있다. 메서드 참조를 이용하면 같은 기능을 더 간결하게 구현할 수 있다.

#### 메서드 참조를 만드는 방법
메서드 참조는 세가지 유형으로 구분할 수 있다.
1. 정적 메서드 참조
   * 예를 들어 Integer의 parseInt 메서드는 Integer::parseInt로 표현할 수 있다.
2. 다양한 형식의 인스턴스 메서드 참조
   * 예를 들어 String의 length 메서드는 String::length로 표현할 수 있다.
3. 기존 객체의 인스턴스 메서드 참조
   * 예를 들어 Transaction 객체를 할당받은 expensiveTransaction 지역변수 -> getValue 메서드가 있다면 -> expensiveTransaction::getValue라고 표현할 수 있다.

```java
// 헬퍼 메서드
private boolean isValidName(String string) {
	return Character.isUpperCase(string.charAt(0));
}

// 람다 표현식
filter(list, (String s) -> isValidName(s));

// 메서드 참조
filter(list, this::isValidName)
```
Predicate<String>을 필요로 하는 적당한 상황에서 메서드 참조를 사용할 수 있다. 생성자, 배열 생성자, super 호출 등에 사용할 수 있는 특별한 형식의 메서드 참조도 있다. 예제를 통해서 확인해보자.
```java
// List에 포함된 문자열을 대소문자를 구분하지 않고 정렬하는 예제
// List의 sort()는 인수로 Comparator를 기대함
// Comparator는 (T, T) → int라는 함수 디스크립터를 가짐
List<String> str = Arrays.asList("a", "b", "A", "B");
str.sort((s1, s2) -> s1.compareToIgnoreCase(s2));
str.sort(String::compareToIgnoreCase);//메서드 참조 사용
```
컴파일러는 람다 표현식의 형식을 검사하던 방식과 비슷한 과정으로 메서드 참조가 주어진 함수형 인터페이스와 호환하는지 확인한다. 즉, 메서드 참조는 콘텍스트의 형식과 일치해야 한다.

## 3.6.2 생성자 참조
ClassName::new처럼 클래스명과 new 키워드를 이용해 기존 생성자의 참조를 만들 수 있다. 정적 메서드의 참조를 만드는 법과 비슷하다. 예를 들어 Supplier의 () -> Apple과 같은 시그니처를 갖는 생성자가 있다고 가정하자.
```java

// 람다 표현식은 디폴트 생성자를 가진 Apple을 만든다.
Supplier<Apple> c1 = () -> new Apple();
Apple a1 = c1.get();

// 생성자 참조
Supplier<Apple> c1 = Apple::new;
Apple a1 = c1.get();

//Supplier의 get 메서드를 호출해서 새로운 Apple객체를 만들 수 있다.
```
Apple(Integer weight)라는 시그니처를 갖는 생성자는 Function 인터페이스의 시그니처와 같다.
```java
// 람다 표현식
Function<Integer, Apple> c2 = (weight) -> new Apple(weight);
Apple a2 = c2.apply(110); 

// 생성자 참조
Function<Integer, Apple> c2 = Apple::new;
Apple a2 = c2.apply(110);

//Function의 apply 메서드에 무게를 인수로 호출해서 새로운 Apple 객체를 만들 수 있다.
```
다음 코드에서 Intger를 포함하는 리스트의 각 요소를 우리가 정의했던 map같은 메서드를 이용해서 Apple 생성자로 전달한다. 결과적으로 다양한 무게를 포함하는 사과 리스트가 만들어진다.
```java
List<Integer> weights = Arrays.asList(7, 3, 4, 10);
List<Apple> apples = map(weights, Apple::new); //map 메서드로 생성자 참조 전달
public List<Apple> map(List<Integer> list, Function<Integer, Apple> f) {
   List<Apple> result = new ArrayList<>();
   for(Integer i: list) {
      result.add(f.apply(i));
   }
   return result;
}
```
Apple(String color, Integer weight)처럼 두 인수를 갖는 생성자는 BiFunction 인터페이스와 같은 시그니처를 가지므로 다음처럼 할 수 있다.
```java
// 람다 표현식
BiFunction<Color, Integer, Apple> c3 = (color, weight) -> new Apple(color, weight);
Apple a3 = c3.apply(GREEN, 110);

// 생성자 참조
BiFunction<Color, Integer, Apple> c3 = Apple::new;
Apple a3 = c3.apply(GREEN, 110);
```
인스턴스화하지 않고도 생성자에 접근할 수 있는 기능을 다양한 상황에 응용할 수 있다. 예를 들어 Map으로 생성자와 문자열값을 관련시킬 수 있다. 그리고 String과 Integer가 주어졌을때 다양한 무게를 갖는 여러 종류의 과일을 만드는 giveMeFruit라는 메서드를 만들 수 있다.
```java
static Map<String, Function<Integer, Fruit>> map = new HashMap<>();
static {
	map.put("apple", Apple::new);
	map.put("orange", Orange::new);
}

public static Fruit giveMeFruit(String fruit, Integer weight) {
	return map.get(fruit.toLowerCase()).apply(weight);
   //Funtion의 apply메서드에 정수 무게 파라미터를 제공해서 Fruit를 만들 수 있다.
}
```
→ 핵심은 람다 표현식 처럼 함수 디스크립터와 같은 시그니처를 가진 경우에 참조가 가능하다는 것이다.
## 3.7 람다, 메서드 참조 활용하기