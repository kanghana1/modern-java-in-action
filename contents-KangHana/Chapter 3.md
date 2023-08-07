# Chapter 3 - 람다 표현식

지난 챕터에서 동작 파라미터화를 이용하면 더 유연한 코드를 만들 수 있음을 배웠습니다. 익명 클래스로 다양한 동작을 구현할 수 있지만, 코드가 깔끔하지는 않았습니다.

이러한 점을 보완하기 위해 자바 8의 새로운 기능인 람다 표현식을 사용합니다.

익명클래스와 람다 표현식은

1. 이름이 없는 함수
2. 메소드를 인수로 전달할 수 있음

위 두가지 특징을 공통으로 갖고 있기에 비슷하다고 할 수 있습니다.

해당 챕터의 대주제는 다음과 같습니다.

- 람다란 무엇인가
- 람다 사용법
- 실행 어라운드 패턴
- 함수형 인터페이스와 형식 추론
- 메소드 참조

지금부터 위 목차를 위주로 정리해 보려고 합니다.
<br>

<hr>
<br>

## ■ 1. 람다란 무엇인가?

람다표현식은 메소드로 전달할 수 있는 익명함수를 단순화한 것입니다. 람다표현식은 이름은 없지만 파라미터 리스트, 바디, 반환형식, 발생할 수 있는 예외 리스트는 가질 수 있습니다.

<br>

#### ▶ 람다의 특징

- 익명
  - 보통의 메소드와 달리 이름이 없어 익명이라 표현
  - 구현코드에 대한 걱정거리가 줄어듦
- 함수
  - 다른 메소드처럼 특정 클래스에 종속되지 않아 함수라고 부름
  - 포함 요소는 다른 메소드와 동일 (파라미터 리스트, 바디, 반환형식, 발생할 수 있는 예외 리스트)
- 전달
  - 람다 표현식을 메소드 인수로 전달 가능
  - 변수로 저장 가능
- 간결성
  - 익명클래스처럼 자질구레한 코드를 구현할 필요 없음 (판에 박힌 코드 구현 필요 X)

<br>

아래 코드를 보면 기존 코드와 람다를 이용한 코드의 차이가 확연히 드러나는 것을 알 수 있습니다.

```java
// 기존코드
Comparator<Apple> byWeight = new Comparator<Apple>() {
  public int compare(Apple a1, Apple a2) {
    return a1.getWeight().compareTo(a2.getWeight());
  }
};
// 람다 이용 코드
Comparator<Apple> byWeight = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```

<br>

#### ▶ 람다의 구성

람다 표현식은 ->를 기준으로 앞부분인 파라미터, 뒷부분인 바디를 포함해 크게 세 부분으로 이루어집니다.

```java
(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```

- 파라미터 리스트
- 화살표
- 람다 바디

<br>

#### ▶ 람다 스타일

- 표현식 스타일 => (parameters) -> expression (return이 함축되어있어 return을 사용하지 않아도 됨)
- 블록 스타일 => (parameters) -> {statements;}

<br>
<hr>
<br>

## ■ 2. 어디에 어떻게 람다를 사용할까?

람다표현식은 **함수형 인터페이스**라는 문맥에서 사용할 수 있습니다.

<br>

## □ 함수형 인터페이스

**함수형 인터페이스**는 오직 **하나의 추상 메소드**만을 지정하는 인터페이스입니다.

> 추상메소드 - 구현부를 갖지 않는 몸체 없는 미완성 메소드

<br>
  
``` java
// 1. 한 개의 추상메소드를 가지므로 함수형 인터페이스 O
public interface Adder {
  int add(int a, int b);
}

// 2. Adder에서 상속받는 것까지 두 개의 추상메소드를 가지므로 함수형 인터페이스 X
public interface SmartAdder extends Adder {
int add(double a, double b);
}

// 3. 추상 메소드가 없으므로 함수형 인터페이스 X
public interface Nothing {  
}

````

함수형 인터페이스를 이용하면 전체 표현식을 함수형 인터페이스의 인스턴스로 취급할 수 있습니다.

<br>

```java
Runnable r1 = () -> System.out.println("Hello World"); // 람다 사용

Runnable r2 = new Runnable() { // 익명 클래스 사용
  public void run() {
    System.out.println("Hello World");
  }
};

public static void process(Runnable r) {
  r.run();
}

process(r1);
process(r2);

````

## □ 함수 디스크립터

함수형 인터페이스의 추상 메소드 **시그니처**는 람다 표현식의 시그니처(메소드의 이름, 파라미터)를 가리킵니다.

> **▷ 함수 디스크립터** : 람다 표현식의 시그니처를 서술하는 메소드
>
> 예시 )
> Runnable 인터페이스의 유일한 추상 메소드 run은 인수와 반환값이 없으므로 (void 반환) 인수와 반환값이 없는 시그니처라고 생각할 수 있음

### ▷ **람다표현식**은

- 변수에 할당 가능
- 함수형 인터페이스를 인수로 받는 메소드로 전달 가능
- 함수형 인터페이스의 추상메소드와 같은 시그니처를 가짐

<br>
<hr>
<br>

## ■ 3. 람다 활용 : 실행 어라운드 패턴

#### ▷ 순환 패턴 : 자원 처리에 사용

1. 자원 열기
2. 처리
3. 자원 닫기

순환패턴은 위와 같은 순서로 일어집니다. 설정과 정리 과정은 대부분 비슷하며, 이 말은 즉 실제 자원을 처리하는 코드를 설정과 정리 두 과정이 둘러싸는 형태를 갖는다는 것을 의미합니다.

이와 같은 과정을 **실행 어라운드 패턴**이라고 부릅니다.

```java
public String processFile() throws IOException {
  try { BufferdReader br = new BufferedReader(new FileReader("data.txt")) { // 파일에서 한 행 읽는 코드
  // 자바 7에서 추가된 try-with-resources 구문 사용해서 간결한 코드 만듦
    return br.readLine();
  }
}
```

<br>

### **▶ 1단계 : 동작 파라미터화를 기억하라**

<br>

위 코드를 한 번에 두 줄을 읽거나 가장 자주 사용되는 단어를 반환하도록 바꾸려면 processFile의 동작을 파라미터화 하면 됩니다. ( 재사용성 높이기 )

파라미터화 함으로서 람다를 이용해 다른 동작을 수행하도록 전달할 수 있게 됩니다.

```java
String result = processFile((BufferdReader br) -> br.readLine() + br.readLine()); // 두 행 출력
```

<br>

### **▶ 2단계 : 함수형 인터페이스를 이용해서 동작 전달**

<br>

함수형 인터페이스 자리에 람다를 사용할 수 있습니다.

=> BufferedReader -> String과 IOException을 던질 수 있는 시그니처와 일치하는 함수형 인터페이스를 만들어야 합니다.

```java
@FunctionalInterface
public interface BufferedReaderProcess {
  String process(BufferedReader b) throws IOException;
}
// 정의한 인터페이스를 processFile 메소드의 인수로 전달 가능
public String processFile(BufferedReaderProcessor p) throws IOException {
  ...
}
```

<br>

### **▶ 3단계 : 동작 실행**

<br>

#### 람다 표현식으로..

- 함수형 인터페이스의 추상 메소드 구현을 직접 전달 가능
- 전달된 코드는 함수형 인터페이스의 인스턴스로 전달된 코드와 같은 방식으로 처리

=>> processFile 바디 내에서 BufferedReaderProcessor 객체의 process를 호출할 수 있습니다.

```java
public String processFile(BufferedReaderProcessor p) throws IOException {
  try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
    return p.process(br); // BufferedReader 객체 처리
  }
}
```

<br>

### **▶ 4단계 : 람다 전달**

<br>

이제 람다를 이용해서 다양한 동작을 processFile 메소드로 전달할 수 있습니다.

```java
// 한 행 처리 코드
String oneLine = processFile((BufferedReader br) -> br.readLine());

// 두 행 처리 코드
String twoLines = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

<br>

### **정리하면 !**

실행 어라운드 패턴을 적용하는 네 단계 과정은

1. 동적 파라미터화 하기
2. 인터페이스 이용해 동작 전달
3. 동작 실행 (여기서 인터페이스의 메소드 사용 가능)
4. 람다 전달

<br>
<hr>
<br>

## ■ 4. 함수형 인터페이스 사용

앞 내용에서 배운 함수형 인터페이스는 다음과 같습니다.

- 함수형 인터페이스는 **오직 하나의 추상 메소드**를 지정
- 함수형 인터페이스의 **추상 메소드는 람다 표현식의 시그니처 묘사**
- 함수형 인터페이스의 **추상 메소드 시그니처 = 함수 디스크립터**

다양한 함수 표현식을 사용하려면 => 공통의 함수 디스크립터를 기술하는 함수형 인터페이스 집합이 필요합니다.

자바 8 라이브러리 설계자들은 java.util.fucntion 패키지로 여러 가지 새로운 함수형 인터페이스를 제공합니다. 해당 절에서는 Predicate, Consumer, Function 인터페이스를 소개드릴 예정입니다.

<br>

### **▶ Predicate**

<br>

java.util.function.Predicate<T> 인터페이스는 **test** 라는 추상 메소드를 정의하며, test는 제네릭 형식 T의 객체를 인수로 받아 불리언을 반환합니다.

해당 메소드는 따로 정의할 필요 없이 바로 사용할 수 있다는 특징이 있습니다.

- **_이용 상황 : T형식의 객체를 사용하는 불리언 표현식이 필요한 상황에서 사용_**

```java
@FunctionalInterface
public interface Predicate<T> {
  boolean test(T t);
}

public <T> List<T> filter(List<T> list, Predicate<T> p) {
  List<T> result = new ArrayList<>();
  for (T t : list) {
    if(p.test(t)) result.add(t);
  }
  return result;
}

Predicate<String> nonEmptyStringPredicate = (String s) -> !s.isEmpty();
List<String> nonEmpty = filter(listOfStrings, nonEmptyStringPredicate);

```

추가로 predicate 인터페이스에는 and나 or 같은 메소드도 있다고 합니다. (Javadoc specification 참고)

<br>

### **▶ Consumer**

<br>

java.util.function.Consumer<T> 인터페이스는 **accept**라는 추상 메소드를 정의하며, accept는 제네릭 형식 T 객체를 받아 void를 반환합니다.

- **_이용 상황 : T 형식의 객체를 인수로 받아서 어떤 동작을 수행하고 싶을 때 사용_**

```java
@FunctionalInterface
public interface Consumer<T> {
  void accept(T t);
}

public <T> void forEach(List<T> list, Consumer<T> c) {
  for(T t: list) c.accept(t);
}
forEach(
  Arrays.asList(1,2,3,4,5),
  (Integer i) -> System.out.println(i);
);
```

<br>

### **▶ Function**

<br>

java.util.function.Function<T,R> 인터페이스는 **apply**라는 추상 메소드를 정의하며, apply는 제네릭 형식 T를 인수로 받아서 제네릭 형식 R 객체를 반환합니다.

- **_이용 상황 : 입력을 출력으로 매핑하는 람다를 정의할 때 사용_**

다음은 String 리스트를 인수로 받아 각 String의 길이를 포함하는 Integer 리스트로 변환하는 map메소드를 정의하는 예제입니다.

```java
@FunctionalInterface
public interface Function<T,R> {
  R apply(T t);
}

public <T,R> List<R> map(List<T> list, Function<T,R> f) {
  List<R> result = new ArrayList<>();
  for (T t : list) {
    result.add(f.apply(t));
  }
  return result;
}

// [7,2,6]
List<Integer> l = map(
  Arrays.asList("lambdas","in","action"),
  (String s) -> s.length()
);

```

<br>

### **기본형 특화**

<br>

위 세 개의 제네릭 햠수형 인터페이스 말고도 특화된 형식의 함수형 인터페이스가 있습니다.

자바의 모든 형식은 참조형 (ex | Byte, Integer, List) 아니면 기본형 (ex | int, double, float)에 해당합니다. 하지만 제네릭 파라미터에는 **참조형**만 사용할 수 있습니다.

자바에서는 기본형을 참조형으로 변환하는 기능인 **박싱(Boxing)**, 참조형을 기본형으로 변환하는 **언박싱(Unboxing)** 을 제공합니다. 또 편의성을 위해 박싱과 언박싱이 자동으로 이루어지는 **오토박싱** 기능도 제공합니다.

```java
// int가 Integer로 박싱됨
List<Integer> list = new ArrayList<>();
for (int i = 300 ; i < 400 ; i++) {
  list.add(i);
}
```

하지만 이런 변환 과정은 비용이 소모됩니다.

박싱한 값은 기본형을 감싸는 래퍼이며, 힙에 저장됩니다. 따라서 박싱한 값은 메모리를 더 소비하고, 기본형을 가져올 때에도 메모리를 탐색해야하는 등 비효율적입니다.

자바 8에서는 기본형을 입출력으로 사용하는 상황에서 오토박싱 동작을 피할 수 있도록 특별한 버전의 함수형 인터페이스를 제공합니다.

```java
public interface IntPredicate {
  boolean test(int t);
}

IntPredicate evenNumbers = (int i) -> i % 2 == 0;
evenNumbers.test(1000); // <- 참 (박싱 X)

Predcate<Integer> oddNumbers = (int i) -> i % 2 != 0;
oddNumbers.test(1000); // <- 거짓 (박싱
```

특정 형식을 입력받는 함수형 인터페이스 이름 앞에는 DoublePredicate, IntConsumer 처럼 형식명이 붙습니다.

<br>

#### 추가로..

> 함수형 인터페이스는 확인된 예외를 던지는 동작을 허용하지 않습니다. 즉, 예외를 던지는 람다 표현식을 만들려면 아래 두 가지 중 한 가지 방식을 이용해야 합니다.
>
> 1. 확인된 예외를 선언하는 함수형 인터페이스를 직접 정의
> 2. 람다를 try/catch 블록으로 감싸기
>
> 보통 try, catch문을 많이 이용합니다.

<br>
<hr>
<br>

## ■ 5. 형식 검사, 형식 추론, 제약

람다로 함수형 인터페이스의 인스턴스를 만들려면, 람다의 실제 형식을 파악해야 합니다.

<br>

## □ 형식 검사

람다가 사용되는 콘텍스트를 이용해서 람다의 형식을 추론할 수 있습니다.

#### **>대상형식**

어떤 콘텍스트에서 기대되는 람다 표현식의 형식

<br>

예제를 확인해보면 아래와 같습니다.

```java
List<Apple> heavierThan150g = filter(inventory, (Apple apple) -> apple.getWeight() > 150);
```

해당 코드는 아래처럼 형식 확인 과정이 진행됩니다.

1. filter 메소드의 선언 확인
2. filter 메소드는 두 번째 파라미터로 Predicate<Apple> 형식을 기대
3. Predicate<Apple>는 test라는 한 개의 추상 메소드 정의하는 함수형 인터페이스
4. test 메소드는 Apple를 받아 boolean을 반환하는 함수 디스크립터 묘사
5. filter 메소드로 전달된 인수는 이와 같은 요구사항 만족

<br>

## □ 같은 람다, 다른 함수형 인터페이스

대상형식이라는 특징 때문에 같은 람다 표현식이라도 호환되는 추상 메소드를 가진 다른 함수형 인터페이스로 사용될 수 있습니다.

```java
// 두 인터페이스 모두 인수는 받지 않고, 제네릭 형식 T를 반환하는 함수 정의
Callable<Integer> c = () -> 42;
PrivilegedAction<Integer> p = () -> 42;
```

<br>

## □ 형식 추론

자바 컴파일러는 람다 표현식이 사용된 콘텍스트(대상 형식)를 이용해서 람다 표현식과 관련된 함수형 인터페이스를 추론합니다. 즉, 대상 형식을 이용해서 함수 디스크립터를 알 수 있으므로 **_컴파일러는 람다의 시그니처도 추론_** 할 수 있다는 것입니다.

그러므로 람다 문법에서 이를 생략할 수 있습니다.

여러 파라미터를 포함하는 람다 표현식에서는 코드 가독성 향상이 두드러집니다!

```java
// 형식 추론 X
Comparator<Apple> c = (Apple a1, Apple a2) -> a.getWeight().compareTo(a2.getWeight());
// 형식 추론 O
Comparator<Apple> c = (a1, a2) -> a.getWeight().compareTo(a2.getWeight());
```

하지만 정답은 없으므로 스스로 결정해야 합니다.

<br>

## □ 지역 변수 사용

람다 표현식에서는 익명 함수가 하는 것처럼 **자유 변수**(외부 변수)를 사용할 수 있습니다.

이러한 동작을 **람다 캡처링**이라고 합니다.

```java
int portNumbver = 1337;
Runnable r = () -> System.out.println(portNumbver);
```

람다는 **인스턴스 변수**와 **정적 변수**를 자유롭게 캡처(자신의 바디에서 참조)할 수 있지만, 그러려면 **지역 변수**는

1. 명시적으로 final로 선언되어 잇어야 하거나,
2. 실질적으로 final로 선언된 변수와 똑같이 사용되어야 합니다.

즉, 람다 표현식은 한 번만 할당할 수 있는 지역 변수를 캡처할 수 있습니다.

```java
int portNumbver = 1337;
Runnable r = () -> System.out.println(portNumbver);
portNumbver = 31337; // 변수에 값을 두 번 할당하므로 컴파일 못 함
```

### **▶ 지역 변수의 제약**

왜 지역변수에 이러한 제약이 있어야 할까 ?

- 인스턴스 변수 ----> **힙(Heap)** 에 저장
- 지역 변수 ----> **스택**에 저장

람다에서 지역 변수에 바로 접근할 수 있게 되면, 변수를 할당한 스레드가 사라져 할당이 해제되어도 람다를 실행하는 스레드에서는 변수에 접글하려 할 수 있습니다.

그래서 자바 구현에서는 원래 변수에 접근을 허용하지 않고, 자유 지역 변수의 복사본을 제공핣니다.

이러한 이유로 지역 변수에는 한 번만 값을 할당해야 한다는 제약이 생긴 것입니다.

<br>

### **+ 클로저**

**클로저**란 함수의 비지역 변수를 자유롭게 참조할 수 있는 함수의 인스턴스를 가리킵니다. 예를 들어 클로저를 다른 함수의 인수로 전달할 수 있습니다.

클로저는 클로저 **외부에 정의된 변수의 값에 접근하고, 값을 바꿀 수 있다.**

자바 8의 람다와 익명 클래스는 메서드의 인수로 전달될 수 있으므로, **자신의 외부 영역의 변수에 접근할 수 있다**는 점에서 클로저와 비슷한 동작을 수행한다고 할 수 있습니다.

다만 람다가 정의된 메서드의 지역 변숫값은 final이여야 하기 때문에, 둘다 **람다가 정의된 메서드의 지역 변수의 값은 바꿀 수 없습니다.**

지역 변수 값은 스택에 존재하므로 자신을 정의한 스레드와 생존을 같이 해야하며, 따라서 지역 변수는 final이어야 합니다. 가변 지역 변수를 새로운 스레드에서 캡처할 수 있다면 안전하지 않은 동작을 수행할 가능성이 생깁니다.

<br>
<hr>
<br>

## ■ 6. 메소드 참조

메소드 참조를 이용하면 기존의 메소드 정의를 재활용해서 람다처럼 전달할 수 있습니다.

```java
// 기존 코드
inventory.sort((Apple a1, Apple a2) -> a.getWeight().compareTo(a2.getWeight()));

// 메소드 참조와 java.util.Comparator.comparing 이용
inventory.sort(comparing(Apple::getWeight));
```

<br>

## □ 요약

메소드 참조는 특정 메소드만을 호출하는 람다의 축약형이라고 생각할 수 있습니다. 메소드명을 참조하면 가독성을 높일 수 있다는 장점도 있습니다.

메소드 참조는 **메소드명 앞에 구분자(::)를 붙이는 방식**으로 활용합니다.

ex |

[ **_(Apple a) -> a.getWeight() == Apple::getWeight_** ] ==> Apple 클래스에 정의된 getWeight의 메소드 참조

실제로 메소드를 호출하는 것은 아니므로 괄호는 필요 없습니다.

<br>

### **▶ 메소드 참조 만들기**

메소드 참조의 유형은 다음 세 가지로 나눌 수 있습니다.

1. 정적 메소드 참조

   Ex | Integer의 parseInt 메소드는 **Integer::parseInt**로 표현

2. 다양한 형식의 인스턴스 메소드 참조

   Ex | String의 length 메소드는 **String::length**로 표현

3. 기존 객체의 인스턴스 메소드 참조

   Ex | Transaction 객체를 할당받은 expensiveTransaction 지역 변수가 있고, Transaction 객체에는 getValue 메소드가 잇다면 이를 **expensiveTransaction::getValue**라고 표현

세 번째 유형의 메소드 참조는 람다 표현식에서 현존하는 외부 객체의 메소드를 호출할 때 사용합니다. 이는 비공개 헬퍼 메소드를 정의한 상황에서 유용하게 활용할 수 있습니다.

```java
// 기존
private boolean isValidName(String string) {
  return Character.isUpperCase(string.charAt(0));
}
// 메소드 참조
filter(words, this::isValidName);
```

<br>

## □ 생성자 참조

ClassName::new처럼 클래스명과 new 키워드를 이용해 기존 생성자의 참조를 만들 수 있습니다. 이는 **정적 메소드의 참조를 만드는 방법**과 비슷합니다.

```java
// 메소드 참조
Supplier<Apple> c1 = Apple::new;
Apple a1 = c1.get();
// 람다 표현식으로 디폴트 생성자를 가진 Apple 만듦
Supplier<Apple> c1 = () -> new Apple();
Apple a1 = c1.get();
```

다음 코드에서 Integer를 포함하는 리스트의 각 요소를 우리가 정의했던 map같은 메소드를 이용해 Apple 생성자로 전달합니다. 결과적으로 다양한 무게를 포함하는 사과 리스트가 만들어집니다.

```java
List<Integer> weights = Arrays.asList(7,3,4,10);
List<Apple> apples = map(weights, Apple::new);

public List<Apple> map(List<Integer> list, Function<Integer, Apple> f) { // map메소드로 생성자 참조 전달
  List<Apple> result = new ArrayList<>();
  for (Integer i : list) {
    result.add(f.apply(i));
  }
  return result;
}
```

인스턴스화 하지 않고도 생성자에 접근할 수 있는 기능을 다양한 상황에 응용할 수 있습니다.

정리하면, 원하는 시그니처를 갖는 함수형 인터페이스가 제공되지 않으면 직접 인터페이스를 구현해 사용하면 됩니다.

<br>
<hr>
<br>

## ■ 7. 람다, 메소드 참조 활용하기

지금까지 동작 파라미터화, 익명 클래스, 람다 표현식, 메소드 참조를 배웠습니다. 이 모든 것을 동원해 다음과 같은 코드로 만드는 것이 목표입니다.

```java
inventory.sort(comparing(Apple::getWeight));
```

<br>

### **▶ 1단계 : 메소드 참조 만들기**

자바 8의 List API에서 sort 메소드를 제공하기에 구현할 필요는 없지만, 어떻게 sort 메소드에 정렬 전략을 전달할 수 있을까요?

sort메소드는 다음과 같은 시그니처를 갖습니다.

```java
void sort(Comparator(? super E) c)
```

이 코드는 Comparator 객체를 인수로 받아 두 사과를 비교합니다. 객체 안에 동작을 포함시켜 sort에 전달된 정렬 전략에 따라 sort의 동작이 달라지게 합니다.

```java
// 1단계 코드
public class AppleComparator implements Comparator<Apple> {
  public inti compare(Apple a, Apple b) {
    return a.getWeight().compareTo(b.getWeight());
  }
}
inventory.sort(new AppleComparator());
```

<br>

### **▶ 2단계 : 익명 클래스 사용**

위 코드에서 Comparator를 한 번만 사용하므로 구현하는 것보다는 **익명클래스**를 이용하는 것이 효율적입니다.

```java
// 2단계 코드
inventory.sort(new Comparator<Apple>() {
  public int compare(Apple a, Apple b) {
    return a.getWeight().compareTo(b.getWeight());
  }
});
```

<br>

### **▶ 3단계 : 람다 표현식 사용**

장황한 코드를 람다표현식을 이용해 바꿀 수 있습니다. 함수형 인터페이스를 기대하는 곳 어디에서나 람다표현식을 사용할 수 있음을 배웠습니다.

함수형 인터페이스가 정의한 **추상 메소드의 시그니처 (함수 디스크립터)는 람다 표현식의 시그니처를 정의**합니다.

위 코드를 개선하면 아래와 같습니다.

```java
inventory.sort((Apple a, Apple b) -> a1.getWeight().compareTo(b1.getWeight));
```

여기서 자바 컴파일러는 **람다의 파라미터 형식을 추론**한다고 했으므로 코드를 아래처럼 더 줄일 수 있습니다.

```java
inventory.sort((a, b) -> a1.getWeight().compareTo(b1.getWeight));
```

**Comparator**는 Comparable 키를 추출하여 Comparator 객체로 만드는 **Function 함수를 인수로 받는 정적 메소드 comparing**을 포함합니다.

이 메소드를 사용해서 가독성을 향상시킬 수 있습니다.

```java
Comparator<Apple> c = Comparator.comparing((Apple a) -> a.getWeight());
```

이제 코드를 아래처럼 간소화 할 수 있습니다.

```java
// 3단계 코드
import static java.util.Comparator.comparing;
inventory.sort(comparing(apple -> apple.getWeight()));
```

<br>

### **▶ 4단계 : 메소드 참조 사용**

**메소드 참조**를 이용하면 람다 표현식의 인수를 더 깔끔하게 전달할 수 있습니다.

```java
// 4단계 코드
import static java.util.Comparator.comparing;
inventory.sort(comparing(Apple::getWeight));
```

<br>
<hr>
<br>

## ■ 8. 람다 표현식을 조합할 수 있는 유용한 메소드

앞서 자바 8 API의 몇몇 함수형 인터페이스와 유틸리티 메소드를 확인했습니다. 이 말은 즉, 간단한 여러 개의 람다 표현식을 조합해서 복잡한 람다 표현식을 만들 수 있다는 것입니다.

**두 프레디케이트를 조합하거나, 두 함수를 조합하는 일이 가능**합니다. 이런 일이 가능한 이유는

함수형 인터페이스에서 **디폴트 메소드**를 정의하기 때문입니다!
<br>

### **▶ 1. Comparator 조합**

정적메소드 Comparator.comparingi을 이용해 비교에 사용할 키를 추출하는 Function 기반의 Comparator를 반환할 수 있습니다.

```java
Comparator<Apple> c = Comparator.comparing(Apple::getWeight);
```

<br>

#### **> 역정렬**

내림차순 정렬을 하고싶으면 다른 Comparator인스턴스를 만들 필요 없이 인터페이스 자체에서 주어진 **reverse라는 디폴트 메소드**를 이용하면 됩니다.

```java
inventory.sort(comparing(Apple::getWeight).reversed()); // 무게 내림차순 정렬
```

<br>

### **> Comparator 연결**

위 코드에서 만약 무게가 같은 두 사과는 원산지 국가별로 사과를 정렬하고 싶으면 어떻게 해야할까요?

이럴 떈 비교 결과를 더 다듬을 수 있는 두 번째 Comparator를 만들 수 있습니다.

```java
inventory.sort(comparing(Apple::getWeight).reversed().thenComparing(Apple::getCountry)); // 무게 내림차순 정렬
```

<br>

### **▶ 2. Comparator 조합**

Predicate 인터페이스는 복잡한 프레디케이트를 만들 수 있도록 negate, and, or 세 가지 메소드를 제공합니다.

예를 들어 '빨간 색이 아닌 사과'처럼 특정 프레디케이트를 반전시킬 때 negate를 사용할 수 있습니다.

```java
// negate 이용 (기존 객체 결과 반전)
Predicate<Apple> notRedApple = redApple.negate();

// and 이용
Predicate<Apple> redAndHeavyApple = redApple.and(apple -> apple.getWeight() > 150);

// or 이용
Predicate<Apple> redAndHeavyAppleOrGreen =
  redApple.and(apple -> apple.getWeight() > 150)
    .or(apple -> GREEN.equals(a.getColor()));
```

and와 or은 작성순이 곧 우선순위입니다.

<br>

### **▶ 3. Function 조합**

Function 인터페이스는 Function 인스턴스를 반환하는 andThen, compose 두 가지 디폴트 메소드를 제공합니다.

<br>

#### **> andThen 메소드**

주어진 함수를 먼저 적용한 결과를 다른 함수의 입력으로 전달하는 함수를 반환합니다. (파이프랑 비슷)
andThen은 앞의 함수를 먼저 실행한 후, 괄호 안 함수를 실행합니다.

Ex |

**f.andThen(b)** == **g(f(x))**

<br>

#### **> compose 메소드**

이 메소드는 andThen 메소드와 순서가 반대라고 생각하면 됩니다. 인수로 주어진 함수를 먼저 실행한 다음 그 결과를 외부 함수의 인수로 제공합니다.

Ex |

**f.compose(g)** == **f(g(x))**

<br>
<hr>
<br>

## ■ 9. 마치며

- 람다 표현식은 익명 함수의 일종이다. 이름은 없지만 파라미터 리스트, 바디, 반환 형식을 가지며 예외를 던질 수 있다.
- 람다 표현식으로 간결한 코드를 구현할 수 있다.
- 함수형 이넡페이스는 하나의 추상 메소드만을 정의하는 인터페이스다.
- 함수형 인터페이스를 기대하는 곳에서만 람다 표현식을 이용할 수 있다.
- 람다 표현식을 이용해서 함수형 인터페이스의 추상 메소드를 즉석으로 제공할 수 있으며 람다 표현식 전체가 함수형 인터페이스의 인스턴스로 취급된다.
- java.util.function 패키지는 Predicate<T>, Function<T,R>, Supplier<T>, COnsumer<T> 등을 포함해 자주 사용하는 다양한 함수형 인터페이스를 제공한다.
- 자바 8은 Predicate<T>와 Function<T,R> 같은 제네릭 함수형 인터페이스와 관련한 박싱 동작을 피할 수 있는 IntPredicate, IntToLongFunction 등과 같은 기본형 특화 인터페이스도 제공한다.
- 실행 어라운드 패턴을 람다와 활용하면 유연성과 재사용성을 추가로 얻을 수 있다.
- 람다 표현식의 기대 형식을 대상 형식이라 한다.
- 메소드 참조를 이용하면 기존의 메소드 구현을 재사용하고 직접 전달할 수 있다.
- Comparator, Predicate, Function 같은 함수형 인터페이스는 람다 표현식을 조합할 수 있는 다양한 디폴트 메소드를 제공한다.
