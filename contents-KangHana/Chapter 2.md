# Chapter 2 - 동작 파라미터화 코드 전달하기

제가 파악한 해당 챕터의 대주제는 다음과 같습니다.

- 요구사항에 대응하기 위한 동작 파라미터화
- 동작 파라미터화를 위한 방법들
  - 익명클래스
  - 람다표현식

지금부터 위 목차를 위주로 정리해 보려고 합니다.
<br>
<br>

## > Intro <

시시각각 벼화하는 요구사항에 최소의 비용으로 대응하며, 구현이 쉽고 유지보수도 용이한 그런 방법! 바로
**동작 파라미터화**입니다.

동작 파라미터화란, 아직 어떻게 실행할 것인지 결정하지 않은 코드 블록을 의미합니다. 이 코드블록은 프로그램에서의 호출, 즉 실행이 나중으로 미뤄진다는 특징이 있습니다.

결과적으로 코드블록에 따라 **_메소드의 동작이 파라미터화_** 된다는 것입니다!
물론 동작 파라미터화를 추가하려면 코드가 늘어난다는 단점이 있지만, 자바8은 람다 표현식으로 이 문제를 해결할 수 있습니다.

<br>
<hr>
<br>

## ■ 1. 변화하는 요구사항에 대응하기

이 책에서는 예제를 통해 유연한 코드를 만드는 사례를 들어 설명했습니다.

<span style="color : #A9D0F5"> **>> 예제 \_ 녹색사과만 필터링하는 기능 구현하기**</span>

<br>

## □ 첫번째 시도 \_ 녹색 사과 필터링

사과 색을 정의하는 Color num이 존재한다고 가정합니다.

enum Color { RED, GREEN }

```java
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

위 코드는 초록색 사과를 선택하는 코드입니다. 그런데 이 상황에서 빨간색 사과도 필터링하고 싶어졌다면 어떻게 해야할까요>

물론 위 메소드를 복사해 조건문을 바꾸면 되지만, 그런 방법으로 하면 더 다양한 색을 필터링 하고싶어지는 등의 다른 변화에 적절하게 대응할 수 없게 됩니다.

이를 해결하는 방법은 **거의 비슷한 코드가 반복 존재한다면 그 코드를 <span style="color: pink">추상화</span>** 하는 것입니다!

<br>

## □ 두번째 시도 \_ 색을 파라미터화

코드를 반복 사용하지 않고 빨간색 사과를 필터링하는 프로그램을 구현하는 방법중 하나는 색을 파라미터화 하는 것입니다. 색을 파라미터화 할 수 있도록 메소드에 파라미터를 추가하면 좀 더 유연한 코드를 작성할 수 있습니다.

```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {
  List<Apple> result = new ArrayList<>();
  for (Apple apple : inventory) {
    if (apple.getColor().equals(color)) {
      result.add(apple);
    }
  }
  return result;
}
```

위처럼 코드를 구현하면 아래와 같이 구현한 메소드를 호출할 수 있습니다.

```java
List<Apple> greenApples = filterApplesByColor(inventory, GREEN);
List<Apple> redApples = filterApplesByColor(inventory, RED);
```

사과의 색 뿐만 아니라 무게에 대응할 수 있는 메소드를 구현할 수도 있습니다.

```java
public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight) {
  List<Apple> result = new ArrayList<>();
  for (Apple apple : inventory) {
    if (apple.getWeight() > weight) {
      result.add(apple);
    }
  }
  return result;
}
```

위 구현 코드도 괜찮지만, 색 필터링 코드와 무게 필터링 코드가 대부분 중복된다는 문제가 있습니다. 이는 소프트웨어 공학의 DRY(Don't Repeat Yourself) 원칙을 어기는 것입니다.

이를 해결하려면 메소드 전체 구현을 고쳐야합니다. 색과 무게를 filter라는 메소드로 합치는 방법도 있지만 이는 사용하면 안 된다고 합니다. 그 이유를 조금 뒤에 설명하겠습니다.

<br>

## □ 세번째 시도 \_ 가능한 모든 속성으로 필터링

아래는 모든 속성을 메소드 파라미터로 추가한 모습입니다. ( 좋지 않은 코드입니다. )

```java
public static List<Apple> filterApples(List<Apple> inventory,Color color, int weight, boolean flag) {
  List<Apple> result = new ArrayList<>();
  for (Apple apple : inventory) {
    if ((flag && apple.getColor().equals(color)) || (!flag && apple.getWeight() > weight)) {
      result.add(apple);
    }
  }
  return result;
}
```

그 후 아래처럼 메소드를 사용할 수 있습니다.

```java
List<Apple> greenApples = filterApplesByColor(inventory, GREEN, 0, true);
List<Apple> heavyApples = filterApplesByColor(inventory, null, 150, false);
```

이렇게 코드를 작성하면 앞으로 요구사항이 바뀌었을 때 유연하게 대응할 수도 없고, 코드의 의미를 한 눈에 파악하기도 힘들어집니다.

어떤 기준으로 사과를 필터링 할 것인지 효과적으로 전달하기 위해 아래부터는 동작 파라미터화를 이용해 유연성을 얻는 방법에 대해 작성할 예정입니다.

<br>
<hr>
<br>

## ■ 2. 동작 파라미터화

사과의 어떤 속성에 기초해 불리언값을 반환하는 방법을 써서 선택 조건을 결정하는 인터페이스를 정의할 수 있습니다. 이처럼 참 또는 거짓을 반환하는 함수를 **프레디케이트**라고 합니다.

```java
public interface ApplePredicate {
  boolean test (Apple apple);
}
```

그리고 이 인터페이스를 이용해 다양한 선택 조건을 대표하는 여러 버전의 ApplePredicate를 정의할 수 있습니다.

```java
public class AppleHeavyWeightPredicate implements ApplePredicate {
  public boolean test(Apple apple) {
    return apple.getWeight() > 150;
  }
}

public class AppleGreenColorPredicate implements ApplePredicate {
  public boolean test(Apple apple) {
    return GREEN.equals(apple.getColor());
  }
}
```

위 조건에 따라 filter 메소드가 다르게 동작할 것이라고 예상할 수 있습니다. 이를 전략 디자인 패턴이라고 부릅니다.

> 전략디자인 패턴이란, 실행중 알고리즘을 선택할 수 있게 하는 행위 소프트웨어 디자인 패턴입니다.
>
> 1. 특정 계열의 알고리즘을 정의합니다
> 2. 각 알고리즘을 캡슐화 하는 알고리즘 패밀리를 정의합니다.
> 3. 런타임에 알고리즘을 선택합니다 >> 이때 해당 계열 안에서 상호 교체가 가능합니다.
>
> \_ 출처 : 전략패턴 위키피디아

즉 위 예제에서 ApplePredicate가 알고리즘 패밀리이고, AppleHeavyWeightPredicate와 AppleGreenColorPredicate가 전략입니다.

ApplePredicate는 filterApples에서 객체를 받아 애플의 조건을 검사하도록 메소드를 고쳐 다양한 동작을 수행할 수 있게 해야합니다. 이렇게 **동작 파라미터화**, 즉 메소드가 다양한 동작을 받아서 내부적으로 다양한 동작을 수행할 수 있습니다.

filterApples 메소드가 ApplePredicate 객체를 인수로 받도록 고쳐서 **필터메소드 내부에서 컬렉션을 반복하는 로직**과 **컬렉션의 각 요소에 적용할 동작**을 **분리**할 수 있게 합니다.

<br>

## □ 네번째 시도 \_ 추상적 조건으로 필터링

다음은 ApplePredicate를 이용한 필터 메소드입니다.

```java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
  List<Apple> result = new ArrayList<>();
  for (Apple apple : inventory) {
    if (p.test(apple)) {  // 프레디케이트 객체로 사과 검사 조건을 캡슐화
      result.add(apple);
    }
  }
}
```

<br>

### **코드/동작 전달하기**

이로써 필요에 따라 다양한 프레디케이트를 만들어 메소드로 전달할 수 있게 되어 유연한 코드가 되었습니다. 필요에 따라 인터페이스를 이용해 클래스를 만들면 되므로 모든 변화에 대응할 수 있게 된 것입니다.

```java
public class AppleRedAndHeavyPredicate implements ApplePredicate {
  public boolean test(Apple apple) {
    return RED.equals(apple.getColor()) && apple.getWeight() > 150;
  }
}

List<Apple> redAndHeavyApples = filterApples(inventory, new AppleRedAndHeavyPredicate())
```

이렇게 filterApples 메소드의 동작을 파라미터화 했습니다.

여기에서 가장 중요한 구현은 test메소드 입니다. 메소드는 객체만 인수로 받기 떄문에 test 메소드를 ApplePredicate 개겣로 감싸 전달해야 하며, 객체를 이용해 불리언 표현식 등을 전달할 수 있으므로 코드를 전달할 수 있는 거나 다름 없습니다.

<br>

### **한 개의 파라미터, 다양한 동작**

<br>

> 동작 파라미터의 강점
>
> -> 컬렉션 탐색 로직과 각 항목에 적용할 동작을 **분리**할 수 있다는 것!

이러한 특징은 하나의 메소드가 다른 동작을 수행하도록 재활용 할 수 있게 해줍니다.

그러나 여러 클래스를 구현하여 인스턴스화 하는 과정이 복잡하게 느껴질 수 있습니다. 이를 어떻게 개선해야 할까요?

<br>
<hr>
<br>

## ■ 3. 복잡한 과정 간소화

filterApples 메소드에 동작을 전달하기 위해서는

- 인터페이스 구현
- 이를 이용해 클래스 구현
- 인스턴스화

위와 같은 과정을 거쳐야 합니다. 이는 상당히 번거로운 작업입니다.

이러한 복잡성을 해결하기 위해 자바는 클래스의 선언과 인스턴스화를 동시에 수행할 수 있도록 **익명 클래스**라는 기법을 제공합니다.

익명 클래스를 이용하면 코드의 양을 줄일 수 있지만 모든 것을 해결할 수는 없다고 합니다.

<br>

## □ 다섯번째 시도 \_ 익명 클래스 사용

**익명클래스**는 자바의 지역 클래스와 비슷한 개념입니다. 익명 클래스는 이름이 없는 클래스로 이를 이용하면 클래스 선언과 인스턴스화를 동시에 할 수 있습니다. 즉 **즉석에서 필요한 구현을 만들어 사용**할 수 있습니다.

<br>

다음은 익명 클래스를 이용해서 ApplePredicate를 구현하는 객체를 만드는 방법으로 구현한 코드입니다.

```java
List<Apple> redApples = filterApples(inventory, new ApplePredicate() { // filterApples 메소드의 동작을 직접 파라터화
  public boolean test(Apple apple) {
    return RED.equals(apple.getColor());
  }
});
```

이 방식은 GUI 애플리케이션에서 이벤트 핸들러 객체를 구현할 때 사용합니다.

즉석에서 구현할 수 있다는 장점이 있지만, 여전히 단점이 있습니다.

- 많은 공간을 차지한다
- 많은 프로그래머가 익명 클래스의 사용에 익숙하지 않다

익명클래스를 사용하면 코드가 장황해질 수밖에 없습니다. 장황한 코드는 구현과 유지보수에 시간이 오래 걸리는 큰 문제점이 있습니다.

또, 코드 조각을 전달하는 과정에서 결국 객체를 만들고 명시적으로 새로운 동작을 정의하는 메소드를 구현해야 한다는 점은 변하지 않습니다.

이런 문제를 람다 표현식을 사용해 정리할 수 있습니다.

 <br>

## □ 여섯번째 시도 \_ 람다 표현식 사용

람다 표현식을 사용하면 위 예제 코드를 다음처럼 간단하게 재구현 할 수 있습니다.

```java
List<Apple> result = filterApples(inventory, (Apple apple -> RED.equals(apple.getColor())));
```

<br>

> 동작파라미터화와 값파라미터화
>
> - 값 파라미터화
> - 동작 파라미터화 (클래스, 익명 클래스, 람다식)
>
> ---
>
> -> 유연함 : 동작파라미터화 >>> 값 파라미터화
> -> 동작파라미터화 내부에서 간결함 : 클래스 < 익명 클래스 < 람다
>
> .

 <br>

## □ 일곱번째 시도 \_ 리스트 형식으로 추상화

```java
public interface Predicate<T> {
  boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p) { // 형식 파라미터 T 등장
  List<T> result = new ArrayList<>();
  for (T e : list) {
    if (p.test(e)) {
      result.add(e);
    }
  }
  return result;
}
```

아래는 람다 표현식을 사용한 예제입니다.

```java
List<Apple> redApples = filter(inventory, (Apple apple) -> RED.equals(apple.getColor()));
List<Integer> evenNumbers = filter(numbers, (Integer i) -> i % 2 == 0);
```

이렇게 람다 표현식을 이용해 유연성과 간결함이라는 두 마리 토끼를 다 잡을 수 있습니다.

<br>
<hr>
<br>

## ■ 4. 실전 예제

동작 파라미터화 패턴

1. 동작을 캡슐화
2. 메소드로 전달
3. 메소드의 동작을 파라미터화

코드전달 개념을 확실히 하기 위해 아래 예제를 통해 정리하겠습니다.

<br>

## □ Comparator로 정렬하기

컬렉션 정렬은 반복되는 프로그래밍 작업입니다. 개발자는 변화하는 요구사항에 쉽게 대응할 수 있는 다양한 정렬 동작을 수행할 수 있는 코드를 짜야합니다.

자바 8의 List에 sort메소드가 포함되어 있습니다. java.util.Comparator 객체를 이용해 sort의 동작을 파라미터화 할 수 있습니다.

```java
public interface Comparator<T> {
  int compare(T o1, T o2);
}
```

Comparator를 구현해 sort 메소드의 동작을 다양화 할 수 있습니다. 바뀌는 요구사항에 맞는 Comparator를 만들어 sort 메소드에 전달할 수 있습니다.

이 또한 람다 표현식을 이용할 수 있습니다.

```java
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
```

<br>

## □ Runnable로 코드 블록 실행하기

**자바 스레드**를 이용하면 병렬로 코드 블록을 실행할 수 있습니다. 자바 8까지는 **Thread 생성자**에 객체만을 전달할 수 있었으므로, 보통 결과를 반환하지 않는 void run 메소드를 포함하는 익명 클래스가 Runnable 인터페이스를 구현하도록 하는 것이 일반적인 방법이었습니다.

자바에선 Runnable 인터페이스를 이용해서 실행할 코드 블록을 지정할 수 있습니다. 아래 코드에서 볼 수 있는 것처럼 코드 블록을 실행한 결과는 void입니다.

```java
// java.lang.Runnable
public interface Runnable {
  void run();
}

Thread t = new Thread(new Runnable() {
  public void run() {
    System.out.println("Hello world");
  }
});
```

Runnable을 이용해서 다양한 동작을 실행할 수 있습니다.
이 역시 람다로 구현이 가능합니다.

```java
Thread t = new Thread(() -> System.out.println("Hello world"));
```

<br>

## □ Runnable로 코드 블록 실행하기

자바 5부터 지원하는 ExecutorService 인터페이스는 태스크 제출과 실행 과정의 연관성을 끊어줍니다.

ExecutorService를 이용하면 태스크를 스레드 풀로 보내고 결과를 Future로 저장할 수 있다는 점이 스레드와 Runnable을 이용하는 방식과는 다릅니다.

지금은 Callable 인터페이스를 이용해 결과를 반환하는 태스크를 만든다는 사실만 알아두면 됩니다.

<br>
<hr>
<br>

## ■ 5. 마치며

- 동작 파라미터화에서는 메소드 내부적으로 다양한 동작을 수행할 수 있도록 코드를 메소드 인수로 전달한다.
- 동작 파라미터화를 이용하면 변화하는 요구사항에 더 잘 대응할 수 있는 코드를 구현할 수 있으며 나중에 엔지니어링 비용을 줄일 수 있다.
- 코드 전달 기법을 이용하면 동작을 메소드의 인수로 전달할 수 있다. 하지만 자바 8 이전에는 코드를 지저분하게 구현해야 했다. 익명 클래스로도 어느정도 코드를 깔끔하게 만들 수 있지만 자바 8에서는 인터페이스를 상속받아 여러 클래스를 구현해야 하는 수고를 없앨 수 있는 방법을 제공한다.
- 자바 API의 많은 메소드는 정렬, 스레드. GUI 처리 등을 포함한 다양한 동작으로 파라미터화 할 수 있다.
