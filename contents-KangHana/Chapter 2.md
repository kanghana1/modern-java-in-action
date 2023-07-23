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
