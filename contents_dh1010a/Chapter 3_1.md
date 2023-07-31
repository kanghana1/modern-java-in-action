# Chapter 3 - 람다 표현식(1)
지난 챕터에서 동작 파라미터화를 이용해서 변화하는 요구사항에 효과적으로 대응하는 코드를 구현할 수 있음을 확인했다. 또한 정의한 코드 블록을 다른 메서드로 전달할 수 있다.

이 챕터에서는 더 깔끔한 코드로 동작을 구현하고 전달하는 자바8의 새로운 기능인 람다 표현식을 설명한다. 람다 표현식은 익명 클래스처럼 이름이 없는 함수면서 메서드를 인수로 전달할 수 있으므로 일단은 비슷하다고 생각할 수 있다.

람다 표현식을 어떻게 만드는지, 어떻게 사용하는지, 어떻게 코드를 간결하게 만들 수 있는지 설명한다. 또한 자바 8 API에 추가된 중요한 인터페이스와 형식 추론 등의 기능, 람다와 함께 위력을 발휘하는 메서드 참조를 설명한다.

## 3.1 람다란 무엇인가?
**람다 표현식**은 메서드로 전달할 수 있는 익명 함수를 단순화 한 것이라고 할 수 있다. 이름은 없지만 파라미터 리스트, 바디, 변환 형식, 발생할 수 있는 예외 리스트는 가질 수 있다. 특징을 살펴보자.

* 익명
  * 보통의 메서드와 달리 이름이 없으므로 익명이라 표현한다. 구현해야 할 코드에 대한 걱정거리가 줄어든다.
* 함수
  * 람다는 메서드처럼 특정 클래스에 종속되지 않으므로 함수라고 부른다. 하지만 메서드처럼 파라미터 리스트, 바디, 반환 형식, 가능한 예외 리스트를 포함한다.
* 전달
  * 람다 표현식을 메서드 인수로 전달하거나 변수로 저장할 수 있다.
* 간결성
  * 익명 클래스처럼 많은 자질구레한 코드를 구현할 필요가 없다.
  
#### 람다 표현식이 왜 중요할까?
코드를 전달하는 과정에서 자질구레한 코드가 많이 생기는 문제를 해결할 수 있다. 즉, 람다를 이용해서 간결한 방식으로 코드를 전달할 수 있다. 기술적으로 자바 8 이전의 자바로 할 수 없었던 일을 제공하는 것은 아니지만 동작 파라미터 형식의 코드를 더 쉽고 간결하고 유연하게 구현할 수 있다.

람다는 세 부분으로 이루어진다.
```java
(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```
* 파라미터 리스트 (Apple a1, Apple a2)
  * Comparator의 compare 메서드 파라미터(사과 두 개)
* 화살표
  * 화살표(->)는 람다의 파라미터 리스트와 바디를 구분한다.
* 람다 바디
  * 두 사과의 무게를 비교한다. 람다의 반환값에 해당하는 표현식이다.

자바 설계자는 이미 C#이나 스칼라 같은 비슷한 기능을 가진 다른 언어와 비슷한 문법을 자바에 적용하기로 했다. 다음은 람다의 기본 문법이다.
* 표현식 스타일 : (parameters) -> expression
* 블록 스타일 : (parameters) -> { statements; }

표현식 스타일의 경우 표현식에 return이 함축되어 있으므로 return을 명시적으로 사용하지 않아도 된다. 블록 스타일에는 여러 행의 문장을 표현하는 구문이 들어가며, 리턴 타입이 void가 아니라면 명시적으로 return을 사용해야한다.

#### 람다 사용사례 및 예제
|사용 사례|람다 예제|
|---|---|
|불리언 표현식|(List<String> list) → list.isEmpty()|
|객체 생성|() → new Apple(10)|
|객체에서 소비|(Apple a) → { System.out.println(a.getWeight());}|
|객체에서 선택/추출|(String s) → s.length()|
|두 값을 조합|(int a, int b) → a * b|
|두 객체 비교|(Apple a1, Apple a2) → a1.getWeight().compareTo(a2.getWeight())|

## 3.2 어디에, 어떻게 람다를 사용할까?
정확히 어디에서 람다를 사용할 수 있다는 건가? -> **함수형 인터페이스**라는 문맥에서 람다 표현식을 사용할 수 있다.

### 3.2.1 함수형 인터페이스
간단히 말해 함수형 인터페이스는 정확히 하나의 추상 메서드를 지정하는 인터페이스이다. 지금까지 살펴본 자바 API의 함수형 인터페이스로 Comparator, Runnable 등이 있다.

> 인터페이스는 디폴트 메서드(인터페이스의 메서드를 구현하지 않은 클래스를 고려해서 기본 구현을 제공하는 바디를 포함하는 메서드)를 포함할 수 있다. 많은 디폴트 메서드가 있더라도 **추상 메서드가 오직 하나**면 함수형 인터페이스다.

함수형 인터페이스로 무엇을 할 수 있을까?람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있으므로 **전체 표현식을 함수형 인터페이스의 인스턴스로 취급**(기술적으로 따지면 함수형 인터페이스를 구현한 클래스의 인스턴스) 할 수 있다. 익명 내부 클래스로도 같은 기능을 구현할 수 있다(덜 깔끔하긴 함).

다음은 Runnable이 오직 하나의 추상메서드 run을 정의하는 함수형 인터페이스이므로 올바른 코드다.
```java
// 람다 사용 
Runnable r1 = () -> System.out.println("Hello world 1");

// 익명 클래스 사용
Runnable r2 = new Runnable() {
	public void run() {
		System.out.println("Hello World 2");
	}
}

public static void process(Runnable r) {
	r.run();
}
process(r1);
process(r2);
process(() -> System.out.println("Hello Wordl 3")); // 람다 표현식으로 직접 전달도 가능
```

### 3.2.2. 함수 디스크립터
함수형 인터페이스의 추상 메서드 시그니처는 람다 표현식의 시그니처를 가리킨다. 람다 표현식의 시그니처를 서술하는 메서드를 **함수 디스크립터** 라고 부른다.

**메서드 시그니처란?** : 자바 프로그래밍 언어에서 메서드 시그니처는 메서드 명과 파라미터의 순서, 타입, 개수를 의미한다. 예를 들어 Runnable 인터페 이스의 유일한 추상 메서드 run은 인수와 반환값이 없으므로(void 반환)Runnable 인터페이스는 인수와 반환값이 없는 시그니처로 생각할 수 있다. 이는 () -> void 와 같은 표기법으로 나타낼 수 있다. (Apple, Apple) -> int는 두개의 apple을 인수로 받아 int를 반환하는 함수를 가리킨다.

-> 3.4절에서 더 다양한 종류의 함수 디스크립터를 소개한다.

**람다 표현식의 형식을 어떻게 검사할까?** 3.5절에서 '형식 검사, 형식 추론, 제약'에서 컴파일러가 람다 표현식의 유효성을 확인하는 방법을 설명한다. 일단은 람다 표현식은 변수에 할당하거나 함수형 인터페이스를 인수로 받는 메서드로 전달할 수 있으모로, 함수형 인터페이스의 추상 메서드와 같은 시그니처를 갖는다는 사실을 기억해두자.

예를 들어 이전 예제에서는 다음처럼 process 메서드에 직접 람다 표현식을 전달하였다.
```java
public void process(Runnable r) {
	r.run();
}

process(() -> System.out.println("This is awesome"))
```
위 코드를 실행하면 ‘This is awesome’이 출력된다. () → System.out.println(”This is awesome”)은 인수가 없으며 void를 반환하는 람다 표현식이다. 이는 Runnable 인터페이스의 run 메서드 시그니처와 같다.

 왜 함수형 인터페이스를 인수로 받는 메서드에만 람다 표현식을 사용할 수 있을까?
 * 설계자들은 언어를 더 복잡하게 만들지 않는 현재 방법을 선택했다.
 * 자바 프로그래머가 하나의 추상 메서드를 갖는 인터페이스(ex. 이벤트 처리 인터페이스)에 익숙하다는 점도 고려했다.

#### @FunctionalInterface는 무엇인가?
함수형 인터페이스임을 가리키는 어노테이션이다. @FunctionalInterface로 인터페이스를 선언했지만 실제로 함수형 인터페이스가 아니라면 컴파일러가 에러를 발생시킨다.

## 3.3 람다 활용 : 실행 어라운드 패턴
자원 처리에 사용하는 순환 패턴은 자원을 열고, 처리한 다음에, 자원을 닫는 순서로 이루어진다. 설정과 정리 과정은 대부분 비슷하다. 즉, 실제 자원을 처리하는 코드를 설정과 저정리 두 과정이 둘러싸는 형태를 갖는다.이와 같은 형식의 코드를 **실행 어라운드 패턴**이라고 부른다.

아래 예제에서 굵은 글씨는 파일에서 한 행을 읽는 코드이다.(예제는 자바 7에 새로 추가된 try-with-resource이다. 이를 사용하면 자원을 명시적으로 닫을 필요가 없으므로 간결한 코드를 구현하는 데 도움을 준다.)
```java
public String processFile() throws IOException {
	**try (BufferedReader br = new BufferedReader(new FileReader("data.txt")))** {
		return br.readLine();
	}
}
```
try-with-resource란? → try()에서 선언된 객체들에 대해서 try가 종료될 때 자동으로 자원을 해제해주는 기능이다. (AutoCloseable을 구현한 객체에 한에서만 close()를 호출)

### 3.3.1 1단계: 동작 파라미터화를 기억하라
현재 코드는 파일에서 한 번에 한 줄만 읽을 수 있다. 한 번에 두 줄을 읽거나 가장 자주 사용되는 단어를 반환하려면 어떻게 해야 할까?
->기존의 설정,정리 과정은 재사용하고 processFile메서드만 다른 동작을 수행하도록 명령할 수 있도록 해야한다. -> processFile의 동작을 파라미터화 해야한다. process가 BufferedReader을 이용해서 다를 동작을 수행할 수 있도록 processFile메서드로 동작을 전달해야 한다.

**람다**를 이용해서 동작을 전달할 수 있다.우선 BufferedReader를 인수로 받아서 String을 반환하는 람다가 필요하다.다음은 BufferedReader에서 두 행을 출력하는 코드이다.
```java
// BufferedReader -> String
String result = processFile((BuffredReader br) -> br.readLine() + br.readLine());
```

### 3.3.2 2단계: 함수형 인터페이스를 이용해서 동작 전달
함수형 인터페이스 자리에 람다를 사용할 수 있다. 따라서 BufferedReader -> String과 IOException을 throw할 수 있는 시그니처와 일치하는 함수형 인터페이스를 만들어야 한다.

이 인터페이스를 BufferdReaderProcess라고 정의하자.
```java
@FuntionalInterface
public interface BufferedReaderProcessor {
	String process(BuffredReader b) throws IOException;
}
```
정의한 인터페이스를 processFile 메서드의 인수로 전달할 수 있다.
```java
public String processFile(BufferedReaderProcessor p) throws IOException {
	...
}
```

### 3.3.3 3단계 : 동작 실행
이제 BuffredReaderProcessor에 정의된 process 메서드의 시그니처(BufferedReader → String)와 일치하는 람다를 전달할 수 있다. 람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있으며, 전달된 코드는 함수형 인터페이스의 인스턴스로 전달된 코드와 같은 방식으로 처리한다.

따라서 processFile 바디 내에서 BufferedReaderProcessor 객체의 process를 호출할 수 있다.
```java
public String processFile(BufferedReaderProcessor p) throws IOException {
	try (BuffredReader br = new BuffredReader(new FileReader("data.txt"))) {
		return p.process(br);
	}
}
```

### 3.3.4 4단계 : 람다 전달
이제 람다를 이용해 다양한 동작을 processFile메서드로 전달할 수 있다.다음은 각각 한 행과 두 행을 처리하는 코드다.
```java
// 한 행을 처리
String oneLine = processFile((BufferedReader br) -> br.readLine());

// 두 행을 처리
Strine twoLines = processFile((BuffredReader br) -> br.readLine() + br.readLine());
```
지금까지 함수형 인터페이스를 이용해서 람다를 전달하는 방법을 확인했으며 인터페이스도 정의하였다.

다양한 람다를 전달하는 데 재활용 할 수 있도록 자바 8에 추가된 새로운 인터페이스를 다음 절에서 살펴보려고 한다.

## 3.4 함수형 인터페이스 사용
앞서 말했듯 함수형 인터페이스는 오직 하나의 추상 메서드를 지정한다. 함수형 인터페이스의 추상 메서드는 람다 표현식의 시그니처를 묘사한다. 함수형 인터페이스의 추상 메서드 시그니처를 **함수 디스크립터**라고 한다.

다양한 람다 표현식을 사용하려면 공통의 함수 디스크립터를 기술하는 함수형 인터페이스 집합이 필요하다. 자바 API는 이미 다양한 함수형 인터페이스를 포함하고 있다.자바 8 라이브러리 설계자들은 java.util.function 패키지로 여러 가지 새로운 함수형 인터페이스를 제공한다.

### 3.4.1 Predicate
test라는 추상 메서드를 정의하며 test는 제너릭 형식 T의 객체를 인수로 받아 **불리언을 반환**한다.
* 따로 정의할 필요 없이 바로 사용할 수 있다
* T 형식의 객체를 사용하는 불리언 표현식이 필요한 상황에서 사용할 수 있다.

다음 예제처럼 String 객체를 인수로 받는 람다를 정의할 수 있다.
```java
@FunctionalInterface
public interface Predicate<T> {
	boolean test(T t);
}

public <T> LisT<T> filter(List<T> list, Predicate<T> p) {
	List<T> results = new ArrayList<>();
	for(T t: list) {
		if(p.test(t)) {
			results.add(t);
		}
	}
}

Predicate<String> nonEmptyStringPredicate = (String) -> !s.isEmpty();
List<String> nonEmpty= filter(listOfStrings,nonEmptyStringPredicate);
```
Predicate 인터페이스의 Javadoc 명세를 보면 and나 or 같은 메서드도 있다.

### 3.4.2 Consumer
제너릭 형식 T를 인수로 받아서 void를 반환하는 accept라는 추상 메서드를 정의한다. T 형식의 객첼르 인수로 받아서 어떤 동작을 수행하고 싶을 때 사용할 수 있다.

예를 들어 Integer 리스트를 인수로 받아서 각 항목에 어떤 동작을 수행하는 forEach 메서드를 정의할 때 활용할 수 있다.
```java
@FunctionalInterface
public interface Consumer<T> {
	void accept(T t);
}

public <T> void forEach(List<T> list, Consumer<T> c) {
	for (T t: list) {
		c.accept(t);
	}
}

forEach(
	Arrays.asList(1, 2, 3, 4, 5),
	(Integer i) -> System.out.println(i) //Consumer의 accept 메서드를 구현하는 람다
);
```

### 3.4.3 Function
제너릭 형식 T를 인수로 받아서 제너릭 형식 R 객체를 반환하는 추상 메서드 apply를 정의한다. 입력을 출력으로 매핑하는 람다를 정의할 때 Function 인터페이스를 활용할 수 있다.(Ex. 사과 무게정보 추출, 문자열을 길이와 매핑)

다음은 String 리스트를 인수로 받아 각 String의 길이를 포함하는 Integer 리스트로 변환하는 map 메서드를 정의하는 예제이다.
```java
@FunctionalInterface
public interface Function<T, R> {
	R apply(T t);
}

public <T, R> List<R> map(List<T> list, Function<T, R> f) {
	List<R> result = new ArrayList<>();
	for (T t : list) {
		result.add(f.apply(t));
	}
	return result;
}

List<Integer> l = map(
	Arrays.asList("lambdas", "in", "action"),
	(String s) -> s.length() //Function의 apply 메서드를 구현하는 람다
);
```

#### 기본형 특화
자바의 모든 형식은 참조형(Byte, Integer, Object, List 등)이 아니면 기본형(int, double, char, Byte)에 해당한다. 하지만 제너릭 파라미터 (예를 들면 Consumer<T>의 T)에는 참조형만 사용할 수 있다.

* 자바는 기본형을 참조형으로 변환하는 기능을 제공하며 이 기능을 Boxing이라고 한다.
* 참조형을 기본형으로 변환하는 동작을 UnBoxing이라고 한다.
* 프로그래머가 편리하게 코드를 구현할 수 있도록 박싱과 언박싱이 자동으로 이루어지는 AutoBoxing이라는 기능도 제공한다.

```java
List<Integer> list = new ArrayList<>();
for (int i = 300; i < 400; i++) {
	list.add(i);
}
```
위 코드는 int가 Integer로 박싱되는 유효한 코드이다. 이러한 변환과정은 비용이 소모된다. 박싱한 값은 기본형을 감싸는 래퍼며 힙에 저장된다. 따라서 박싱한 값은 메모리를 더 소비하며 기본형을 가져올 때도 메모리를 탐색하는 과정이 필요하다.

자바 8에서는 기본형을 입출력으로 사용하는 상황에서 **오토박싱 동작을 피할 수 있도록** 특별한 버전의 함수형 인터페이스를 제공한다.
```java
public interface IntPredicate {
    boolean test(int t);
}

IntPredicate<Integer> evenNumbers = (Integer i) -> i % 2 == 0;
evenNumbers.test(1000); // true (박싱 과정이 발생 x)

Predicate<Integer> oddNumbers = (Integer i) -> i % 2 != 0;
addNumbers.test(1000); // false (박싱 과정이 발생)

```
일반적으로 특정 형식을 입력으로 받는 함수형 인터페이스의 이름 앞에는 Int*, Double* 등의 형식명이 붙는다.

아래 표는 자바 API에서 제공하는 대표적인 함수형 인터페이스와 함수 디스크립터를 보여준다.

|함수형 인터페이스|함수 디스크립터|기본형 특화|
|---|---|---|
|Predicate<T>|T->boolean|IntPredicate, LongPredicate, DoublePredicate|
|Consumer<T>|T → void|IntConsumer, LongConsumer, DoubleConsumer|
|Funtion<T, R>|T → R|IntFunction<R>,IntToDoubleFunction,IntToLongFunction,LongFunction, LongToDoubleFunction, LongToIntFunction,DoubleFunction, DoubleToIntFunction, DoubleToLongFunction,ToIntFunction, ToDoubleFunction, ToLongFunction|
|...|...|이하 생략...|

아래 표는 람다와 함수형 인터페이스 예제를 보여준다.
|사용 사례|람다 예제|대응하는 함수형 인터페이스|
|---|---|---|
|불리언 표현|(List<String> list) → list.isEmpty()|Predicate<List<String>>|
|객체 생성|() → new Apple(10)|Supplier<Apple\>|
|객체에서 소비|(Apple a) → System.out.println(a.getWeight)|Consumer<Apple\>|
|객체에서 선택/추출|(String s) → s.length()|Function<String, Integer> 또는 ToIntFunction<String\>|
|두 값 조합|(int a, int b) → a*b|IntBinaryOperator|
|두 객체 비교|(Apple a1, Apple a2) → a1.getWeight().compareTo(a2.getWeight())|Comparator<Apple\> 또는 BiFunction<Apple, Apple, Integer> 또는 ToIntBiFunction<Apple, Apple>|

#### 예외, 람다, 함수형 인터페이스의 관계
함수형 인터페이스는 확인된 예외를 던지는 동작을 허용하지 않는다. 예외를 던지는 람다 표현식을 만들려면 확인된 예외를 선언하는 함수형 인터페이스를 직접 정의하거나 람다를 try-catch 블록으로 감싸야 한다.
```java
@FunctionalInterface
public interface BufferedReaderProcessor {
	String process(BufferedReader b) throws IOException;
}

// IOException은 Exception을 상속. 즉, checkedException
public class IOException extends Exception {
	...
}
```
직접 함수형 인터페이스를 만들기 어려운 상황에서는 아래 예제와 같이 명시적으로 확인된 예외를 잡을수 있도록 한다.
```java
Function<BufferedReader, String> f = (BufferedReader b) -> {
	try {
		return b.readLine();
	} catch(IOException e) {
		throw new RuntimeException(e);
	}
};
```