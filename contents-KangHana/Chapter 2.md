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
