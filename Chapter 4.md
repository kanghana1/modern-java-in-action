# Chapter 4 - 스트림 소개

이번 챕터에서는 스트림에 대해 중점적으로 다룰 것입니다. 해당 챕터의 대주제는 다음과 같습니다.

- 스트림이란 ?
- 컬렉션과 스트림
- 내부 반복과 외부 반복
- 중간 연산과 최종 연산

<br>

만약 **컬렉션**이 없다면 무슨 일이 일어날까요? 거의 모든 자바 애플리케이션은 컬렉션을 만들고 처리하는 과정을 거칩니다. 컬렉션을 통해 데이터를 그룹화하고 처리할 수 있기 때문에, 대부분의 프로그래밍 작업에 이용합니다.

```sql
SELECT name FROM dishes WHERE calories < 400
```

다음과 같은 SQL 질의를 보면 알 수 있듯이, SQL에서는 '어떻게 필터링 할 것인지' 구현할 필요는 없습니다. 우리에게 필요한 것을 직접적으로 명시할 수 있기 때문에, 질의를 어떻게 구현할 것인지 명시할 필요가 없는 것입니다. 게다가 구현은 자동으로 제공됩니다. 컬렉션으로 이와 비슷한 기능을 만들 수 있지 않을까요?

많은 요소를 포함하는 커다락 컬렉션을 처리할 때, 성능을 높이려면 **멀티코어 아키텍처**를 활용해 병렬로 처리해야합니다.

하지만 이는 단순 반복 처리 코드에 비해 복잡하고 이때문에 디버깅이 어렵다는 단점이 있습니다.

<br>

이러한 문제들을 복합적으로 해결하기 위해 나온 답이 바로 _**스트림**_ 입니다.

<br>
<hr>
<br>

## ■ 1. 스트림이란 무엇인가?

<br>

**_스트림_** 은 자바 8 API에 새로 추가된 기능입니다. 스트림을 이용하면 선언형으로 컬렉션 데이터를 처리할 수 있습니다.

> 쉽게 스트림은 데이터 컬렉션 반복을 멋지게 처리하는 기능이라고 생각하면 됩니다.

<br>

스트림을 이용하면 멀티스레드 코드를 구현하지 않아도 데이터를 투명하게 병렬로 처리할 수 있습니다.

```java
// 기존 코드

List<Dish> lowCaloricDishes = new ArrayList<>();
for (Dish dish : menu) {
  if (dish.getCalories() < 400>) {
    lowCaloricDishes.add(dish);
  }
}
Collections.sort(lowCaloricDishes, new Comparator<Dish>() {
  public int compare(Dish d1, dish d2) {
    return Integer.compare(d1.getCalories(), d2.getCalories());
  }
});

List<String> lowCaloricDishesName = new ArrayList<>();
for (Dish dish : lowCaloricDishes) {
  lowCaloricDishesName.add(dish.getName());
}
```

위 코드에너는 lowCaloricDisihes라는 **'가비지 변수'** , 즉 _컨테이너 역할만 하는 중간 변수_ 를 사용했습니다.

다음은 최신 코드입니다.

```java
// Java 8
import static java.util.Comparator.comparing;
import static java.util.stream.Collectors.toList;
List<String> lowCaloricDisihesName = menu.stream()
                                          .filter(d -> d.getCalories() < 400)
                                          .sorted(comparing(Dish::getCalories)) // 칼로리로 요리 정렬
                                          .map(Dish::getName) // 요리명 추출
                                          .collect(toList()); // 요리명 리스트 저장
```

여기서 stream()을 parallelStream으로 바꾸면 멀티코어 아키텍처에서 병렬로 실행할 수 있습니다.

```java
import static java.util.Comparator.comparing;
import static java.util.stream.Collectors.toList;
List<String> lowCaloricDisihesName = menu.parallelStream()
                                          .filter(d -> d.getCalories() < 400)
                                          .sorted(comparing(Dish::getCalories))
                                          .map(Dish::getName)
                                          .collect(toList());
```

스트림을 사용하면 다음과 같은 소프트웨어공학적인 장점이 있습니다.

### - **선언형**으로 코드를 구현할 수 있습니다.

if문 등 제어블록을 사용해 구현을 지정할 필요 없이 동작 수행을 지정할 수 있습니다.

### - 여러 빌딩 블록 연산을 연결해 복잡한 데이터 처리 파이프라인을 만들 수 있습니다.

이를 통해 가독성과 명확성을 유지할 수 있습니다.

<br>

filter같은 연산은 고수준 빌딩 블록으로 이루어져 있습니다. 이들은 다음과 같은 장점이 있습니다.

- 특정 스레딩 모델에 제한되지 않음
- 자유롭게 어떤 상황에서든 사용 가능

<br>

결과적으로 자바 8의 스트림 API 특징을 다음처럼 요약할 수 있습니다.

- ### **선언형** : 더 **간결**하고 **가독성**이 좋아집니다.
- ### **조립**할 수 있음 : **유연성**이 좋아집니다.
- ### **병렬화** : 성능이 좋아집니다.

<br>
<hr>
<br>

## ■ 2. 스트림 시작하기

