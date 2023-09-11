# Chapter 4 - 스트림 활용

이번 챕터에서는 스트림의 활용에에 대해 중점적으로 다룰 것입니다. 해당 챕터의 대주제는 다음과 같습니다.

- 필터링, 슬라이싱, 매칭
- 검색, 매칭, 리듀싱
- 특정 범위의 숫자와 같은 숫자 스트림 사용
- 다중 소스로부터 스트림 만들기
- 무한 스트림

<br>

지난 챕터에서는 스트림읕 통해 외부반복을 내부반복으로 바꾸는 방법을 알아보았습니다. 다음은 데이터 컬렉션 반복을 명시적으로 반복하는 **외부반복코드(for문 사용)** 입니다.

```java
List<Dish> vegetarianDishes = new ArrayList<>();
for (Dish d : menu) {
  if (d.isVegetarian()) {
    vegetarianDishes.add(d);
  }
}
```

명시적 반복 (for문을 사용한 외부반복) 대신 filter와 collect 연산을 지원하는 스틑림 API를 이용하여 데이터 컬렉션 반복을 내부적으로 처리할 수 있습니다.

다음처럼 filter 메소드에 필터링 연산을 인수로 넘기면 됩니다.

```java
import static java.util.stream.Collectors.toList;
List<Dish> vegetarianDishes = menu.stream()
                                  .filter(Dish::isVegetarian)
                                  .collect(toList());
```

데이터 처리 방법은 스트림 API가 관리하기 떄문에 편리하게 작업을 할 수 있습니다. 따라서 스트림 API 내부적으로 다양한 최적화가 이루어질 수 있습니다.

스트림 API는 내부 반복 뿐만 아니라 코드를 병렬로 실행할지 여부도 결정할 수 있습니다. 이런 일은 **순차적인 반복을 단일 스레드로 구현하는 외부 반복으로는 달성할 수 없습니다.**

<br>

해당 챕터에서는 스트림 API가 지원하는 다양한 연산을 살펴봅니다.

<br>
<hr>
<br>

## ■ 1. 필터링

<br>
