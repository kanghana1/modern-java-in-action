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

<br>

자바 8 컬렉션에는 스트림을 반환하는 stream 메소드가 추가됐습니다. stream 메소드 이외에도 숫자 범위나 I/O 자원에서 스트림 요소를 만드는 방법 등으로 스트림을 얻을 수 있습니다.
그렇다면 스트림이란 무엇일까요 ?

<br>

### 스트림이란..

데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소입니다.

- **연속된 요소** : 컬렉션과 마찬가지로 스트림은 특정 요소 형식으로 이루어진 연속된 값 집합의 인터페이스를 제공합니다.
  - **컬렉션 (주제: 데이터)**: 자료구조이므로 시공간의 복잡성과 관련된 요소 저장 및 접근 연산이 주를 이룸
  - **스트림 (주제 : 계산)** : filter, sorted, map처럼 표현 계산식이 주를 이룸
- **소스** : 스트림은 컬렉션, 배열, I/O 자원 등의 데이터 제공 소스로부터 데이터를 소비합니다. **_정렬된 컬렉션으로 스트림을 생성하면 정렬이 그대로 유지_** 됩니다. (즉, 리스트로 스트림을 만들면 스트림의 요소는 리스트의 요소와 같은 순서를 유지합니다)
- **데이터 처리 연산** : 스트림은 함수형 프로그래밍 언어에서 일반적으로 지원하는 연산과 데이터베이스와 비슷한 연산을 지원합니다. 예를 들어 filter, map, reduce, find, match, sort 등으로 데이터를 조작할 수 있습니다. 스트림 연산은 **_순차적으로, 병렬적으로 실행_** 할 수 있습니다.

또 스트림에는 다음과 같은 두 가지 중요 특징이 있습니다.

- **파이프라이닝** : 대부분의 스트림 연산은 **_스트림 연산끼리 연결해서 커다란 파이프라인을 구성할 수 있도록 스트림 자신을 반환_** 합니다. 그 덕에 게으름, 쇼트서킷같은 최적화도 얻을 수 있습니다. (쇼트서킷이란, &&, ||를 사용하는 것을 말합니다 )
- **내부반복** : 스트림에서는 컬렉션과 달리 내부반복을 지원합니다. ( 컬렉션에서는 반복자를 이용해 명시적으로 반복 )

<br>

```java
import static java.util.stream.Collectors.toList;
List<String> threeHighCaloricDishNames = menu.stream() // 메뉴에서 스트림 얻음
                                          .filter(dish -> dish.getCalories() > 300) // 파이프라인 연산만들기
                                          .map(Dish::getName)
                                          .limit(3)
                                          .collect(toList()); // 결과를 리스트로 저장
System.out.println(threeHighCaloricDishNames);
```

- 우선 요리 리스트를 포함하는 menu에 stream 메소드를 호출해 스트림 얻음 => **데이터 소스**는 요리 리스트(menu) \_ 뎅터 소스는 연속된 요소를 스트림에 제공
- 다음으로 스트림에 filter, map, limit, collect로 이어지는 데이터 처리 연산 적용
  - collect를 제외한 모든 연산 => 서로 파이프라인을 형성할 수 있도록 **스트림 반환**
  - collect => 파이프 라인을 처리해서 결과 반환 (예제에서는 리스트 반환)

collect가 호출되기 전까지는 메소드 호출이 저장되는 효과가 있습니다.

<br>

### 메소드들의 수행 작업

- filter
  - 람다를 인수로 받음
  - 스트림에서 특정 요소를 제외시킴
- map
  - 람다를 이용해 한 요소를 다른 요소로 변환하거나 정보 추출
  - 예제에서는 메소드 참조 Dish::getName을 전달
- limit
  - 요소 개수를 제한
- collect
  - 스트림을 다른 형식으로 변환
  - 다양한 변환 방법을 인수로 받음

스트림 라이브러리에서 필터링(filter), 추출(map), 축소(limit) 기능을 제공하므로 직접 기능을 구현할 필요가 없습니다.

<br>
<hr>
<br>

## ■ 3. 스트림과 컬렉션

<br>

자바의 기존 컬렉션과 새로운 스트림 모두 연속된 요소 형식의 값을 저장하는 자료구조 인터페이스를 제공합니다. 여기서 '연속된' 이라는 표현은 순서와 상관없이 아무 값에나 접속하는게 아닌, **순차적으로 값에 접근한다는 것을 의미**합니다.

<br>

### □ 컬렉션 vs 스트림

이 둘의 차이는 다음과 같습니다.

- 1 \_ 데이터를 언제 계산하느냐
  - ▷ **컬렉션**
    - 현재 자료구조가 포함하는 모든 값을 메모리에 저장하는 자료구조
    - 컬렉션의 모든 요소는 컬렉션에 추가하기 전에 계산되어야 함
    - 컬렉션에 요소를 추가하거나 삭제 가능
    - 적극적으로 생성(모든 값을 계산할 때까지 기다림)
  - ▷ **스트림**
    - 요청할 때만 요소를 계산하는 고정된 자료구조
    - 요소를 추가하거나 삭제 불가능
    - 사용자가 요청하는 값만 스트림에서 추출
    - 생산자와 소비자 관계 형성

여기서 스트림은 반복자와 마찬가지로 한 번만 탐색할 수 있습니다. 즉, 탐색된 스트림의 요소는 소비됩니다. 한 번 탐색한 요소를 다시 탐색하려면, **초기 데이터 소스에서 새로운 스트림을 만들어야**합니다.

- 2 \_ 데이터 반복 처리 방법
  - ▷ **컬렉션**
    - **_외부 반복_** 사용 \_ 사용자가 직접 요소를 반복하는것 (ex | foreach)
  - ▷ **스트림**
    - **_내부 반복_** 사용 \_ 반복을 알아서 처리하고 결과 스트림값을 어딘가에 저장해줌

```java
// 컬렉션 : for-each 루프를 이용하는 "외부반복"
List<String> names = new ArrayList<>();
for (Dish dish : menu) {
  names.add(dish.getName());
}

 // 컬렉션 : 내부적으로 숨겨졌던 반복자를 사용한 "외부반복"
List<String> names = new ArrayList<>();
Iterator<String> iterator = menus.iterator();
while (iterator.hasNext()) { // 명시적 반복
  Dish dish = iterator.next();
  names.add(dish.getName());
}

// 스트림 : "내부반복"
List<String> names = menu.stream()
                    .map(Dish::getName)
                    .collect(toList()); // 파이프라인 실행 (반복자 필요 X)
```

### □ 내부반복과 외부반복 차이

- **외부반복**
  - 명시적으로 컬렉션 항목을 "하나씩" 가져와서 처리
  - 병렬성을 스스로 관리해야함 (병렬성을 포기하거나 synchronized로 어렵게 처리)
- **내부반복**
  - 투명하게 병렬로 처리하거나, **_최적화_** 된 다양한 순서로 처리 가능
  - 병렬성 구현을 자동으로 선택 가능

스트림은 내부 반복을 사용하므로 반복 과정을 신경쓰지 않아도 됩니다.

<br>
<hr>
<br>

## ■ 4. 스트림 연산

<br>
