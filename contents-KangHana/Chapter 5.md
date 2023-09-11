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

스트림의 요소를 선택하는 방법 즉, 프레디케이트 필터링 방법과 고유 요소만 필터링하는 방법을 설명할 예정입니다.

<br>

### ▷ 프레디케이트로 필터링

스트림 인터페이스는 filter 메소드를 지원합니다. filter 메소드는 **프레디케이트(불리언을 반환하는 함수)** 를 인수로 받아서 프레디케이트와 일치하는 모든 요소를 포함하는 스트림을 반환합니다.

예시는 다음과 같습니다.

```java
Lsit<Dish> vegetarianMenu = menu.stream()
                                .filter(Dish::isVegetarin) // 프레디케이트를 인수로 받음
                                .collect(toList());
```

<br>

### ▷ 고유 요소 필터링

스트림은 고유 요소로 이루어진 스트림을 반환하는 distinct 메소드도 지원합니다. 이떄 고유 여부는 스트림에서 만든 객체의 hashCode, equals로 결정됩니다.

```java
List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
numbers.stream()
        .filter(i -> i % 2 == 0)
        .distinct() // 필터링 된 요소들을 중복없이 뽑아냄 (고유 요소)
        .forEach(System.out::println);
```

<br>
<hr>
<br>

## ■ 2. 스트림 슬라이싱

<br>

스트림의 요소를 선택하고 스킵하는 메소드가 있습니다. 프레디케이트를 이용하는 방법, 스트림의 처음 몇 개의 요소를 무시하는 방법, 특정 크기로 스트림을 줄이는 방법 등 다양한 방법을 이용해 효율적으로 작업을 수행할 수 있습니다.

<br>

### ▷ 프레디케이트를 이용한 슬라이싱

자바 9는 스트림의 요소를 효과적으로 선택할 수 있도록 takeWhile, dropWhile 두 가지 새로운 메소드를 지원합니다.

<br>

### **TAKEWHILE** 활용

takeWhile연산을 이용하면 조건에 맞지 않는 요소가 들어왔을 떄, 반복 작업을 중단하는 역할을 하는 연산입니다. 이는 **이미 정렬되어있는 리스트에서만 사용**할 수 있습니다!

<br>

다음과 같은 특별한 요리 목록을 갖고 있다고 가정해봅시다.

```java
List<Dish> specialMenu = Arrays.asList(
  new Dish("seasonal fruit", true, 120, Dish.Type.OTHER),
  new Dish("prawns", false, 300, Dish.Type.FISH),
  new Dish("rice", true, 350, Dish.Type.OTHER),
  new Dish("chicken", false, 400, Dish.Type.MEAT)
);
```

위 목록에서 칼로리 부분은 이미 정렬이 되어있습니다. 리스트가 이미 정렬되어 있다는 사실을 이용하여 300칼로리보다 크거나 같은 요리가 나왔을 떄 takeWhile연산을 이용하여 반복 작업을 중단할 수 있습니다.

```java
List<Dish> slicedMenu1 = specialMenu.stream()
                                    .takeWhile(dish -> dish.getCalories() < 300)
                                    .collect(toList());
```

<br>

### **DROPWHILE** 활용

나머지 요소를 선택할때, 즉 300 칼로리보다 큰 요소를 탐색할 때에는 dropWhile을 이용해주면 됩니다.

```java
List<Dish> slicedMenu2 = specialMenu.stream()
                                    .dropWhile(dish -> dish.getCalories() < 300)
                                    .collect(toList());
```

dropWhile은 takeWhile과 정반대의 작업을 수행합니다. dorpWhile은 프레디케이트가 처음으로 거짓이 되는 지점까지 발견된 요소를 버립니다. 거짓이 되면 그 지점에서 _작업을 중단_ 하고, **남은 모든 요소를 반환**합니다.

이는 무한한 남은 요소를 가진 무한 스트림에서도 동작합니다!

<br>

### ▷ 스트림 축소

<br>

### **limit(n)**

limit은 스트림 크기를 지정해주는 메소드입니다. 스트림은 주어진 값 이하의 크기를 갖게 됩니다.

```java
List<Dish> dishes = specialMenu.stream()
                                .filter(dish -> dish.getCalories() > 300)
                                .limit(3)
                                .collect(toList());
```

프레디케이트와 일치하는 처음 세 요소를 선택한 후 즉시 결과를 반환합니다. 소스가 정렬되어 있지 않았다면 limit의 결과도 정렬되지 않은 상태로 반환됩니다.

<br>

### ▷ 요소 건너뛰기

<br>

### **skip(n)**

skip은 처음 n개 요소를 제외한 스트림을 반환하는 메소드입니다. n개 이하의 요소를 포함하는 스트림에 skip(n)을 호출하면 빈 스트림이 반환됩니다.

```java
List<Dish> dishes = specialMenu.stream()
                                .filter(dish -> dish.getCalories() > 300)
                                .skip(2)
                                .collect(toList());
```

<br>
<hr>
<br>

## ■ 3. 매핑

<br>

특정 객체에서 특정 데이터를 선택하는 작업은 데이터 처리 과정에서 자주 수행됩니다. 스트림 API의 map과 flatMap 메소드는 특정 데이터를 선택하는 기능을 제공합니다.

<br>

### □ 스트림의 각 요소에 함수 적용하기

스트림은 **함수를 인수로** 받는 **map 메소드**를 지원합니다. 인수로 제공된 함수는 각 요소에 적용되고 함수를 적용한 결과가 새로운 요소로 매핑됩니다.

```java
List<String> dishNames = menu.stream()
                            .map(Dish::getName)
                            .collect(toList());
```

getName은 문자열을 반환하므로 map 메소드의 출력 스트림은 Stream<String> 형식을 갖습니다.
각 요리명의 길이를 알고싶으면 다음처럼 다른 map 메소드를 연결할 수도 있습니다.

```java
List<Integer> dishNameLengths = menu.stream()
                                    .map(Dish::getName)
                                    .map(String::length)
                                    .collect(toList());
```

<br>

### □ 스트림 평면화

위에서 했던 것들을 응용하여 리스트에서 고유 문자로 이루어진 리스트를 반환해봅시다. 예를 들어 ["Hello", "World"] 리스트가 있다면 결과로 ["H","e","l","o","W","r","d"] 를 포함하는 리스트가 반환되어야 합니다.

<br>

### **- 잘못된 풀이**

리스트에 있는 각 단어를 문자로 매핑한 다음 distinct로 중복된 문자를 필터링해 풀 수 있을거라고 생각했을수도 있습니다. (제가 그랬습니다) 즉, 코드로 작성하면 다음과 같습니다.

```java
words.stream()
      .map(word -> word.split(""))
      .distinct()
      .collect(toList());
```

하지만 위 코드는 각 단어의 Stirng[]을 반환한다는 문제점이 있습니다. 즉, map 메소드가 반환한 스트림의 형식은 Stream<String[]> 입니다. 반면 우리가 원하는 반환 형식은 **Stream<String.>** 입니다.

<br>

또 다른 경우로는 map 과 Arrays.stream을 활용한 경우가 있습니다. 다음 코드처럼 문자열을 받아 스트림을 만드는 Arrays.stream 메소드를 활용합니다.

```java
String[] arrayOfwords = {"Goodbye", "World"};
Stream<String> streamOfwords = Arrays.stream(arrayOfwords);

words.stream()
      .map(word -> word.split("")) // 각 단어를 개별 문자열 배열로 변환
      .map(Arrays::stream) // 각 배열을 별도의 스트림으로 생성
      .distinct()
      .collect(toList());
```

이렇게 써도 결국 스트림 리스트(List<Stream<String>>)이 만들어지므로 문제는 해결되지 않습니다.

문제를 해결하기 위해서는 각 단어를 개별 문자열로 이루어진 배열로 만든 후 각 배열을 별도의 스트림으로 만들어야 합니다.

<br>

### **-flatMap 사용**

flatMap을 사용하면 다음처럼 문제를 해결할 수 있습니다.

```java
List<String> uniqueCharacters = words.stream()
                                      .map(word -> word.split("")) // 배열변환
                                      .flatMap(Arrays::stream) // 생성된 스트림을 하나의 스트림으로 평면화
                                      .distinct()
                                      .collect(toList())
```

**flatMap**은 각 배열을 스트림이 아니라 **_스트림의 콘텐츠로 매핑_** 합니다. 즉, 하나의 평면화된 스트림을 반환합니다.

요약하면 flatMap 메소드는 스트림의 각 값을 다른 스트림으로 만든 후, 모든 스트림을 하나의 스트림으로 만드는 기능을 수행합니다.

<br>
<hr>
<br>

## ■ 4. 검색과 매칭

<br>

특성 속성이 데이터 집합에 있는지 여부를 검색하는 데이터 처리도 자주 사용됩니다. 스트림 API는 allMatch, anyMatch, noneMatch, findFirst, findAny 등 다양한 유틸리티 메소드를 제공합니다.

<br>

### ▷ 프레디케이트가 적어도 한 요소와 일치하는지 확인 \_ **anyMatch** 메소드

적어도 한 요소와 일치하는지 확인할 때에는 anyMatch 메소드를 활용합니다.

```java
if (menu.stream().anyMatch(Dish::isVegetarian)) {
  System.out.println("The menu is (somewhat) vegetarian friendly!");
}
```

anyMatch는 **불리언을 반환**합니다.

<br>

### ▷ 프레디케이트가 모든 요소와 일치하는지 확인

<br>

### **allMatch** 메소드

스트림의 모든 요소가 주어진 프레디케이트와 일치하는지 검사합니다. 예를 들어 메뉴가 건강식(모든 요리가 1000칼로리 이하면 건강식)인지 확인할 수 있습니다.

```java
boolean isHealthy = menu.stream()
                        .allMatch(dish -> dish.getCalories() < 1000);
```

<br>

### **noneMatch** 메소드

noneMatch는 allMatch와 반대되는 연산을 수행합니다. 즉, noneMatch는 주어진 프레디케이트와 일치하는 요소가 없는지 확인합니다. 위 예제를 noneMatch를 이용해 다시 구현할 수 있습니다.

```java
boolean isHealthy = menu.stream()
                        .noneMatch(dish -> dish.getCalories() >= 1000);
```

anyMatch, allMatch, noneMatch 세 메소드는 스트림 **쇼트서킷 기법**, 즉 자바의 &&, ||와 같은 연산을 활용합니다.

> **쇼트서킷**
>
> <br>
>
> 모든 스트림의 요소를 처리하지 않고도 결과를 반환할 수 있는 상황을 쇼트서킷이라고 합니다. 즉, 원하는 요소를 찾았으면 즉시 결과를 반환할 수 있습니다.
>
> 마찬가지로 스트림의 모든 요소를 처리할 필요 없이 주어진 크기의 스트림을 생성하는 limit도 쇼트서킷 연산입니다.
> <br>
>
> \_

<br>

### ▷ 요소 검색

findAny 메소드는 현재 스트림에서 임의의 요소를 반환합니다. 이 메소드를 다른 스트림 연산과 연결해서 사용할 수 있습니다.

**스트림 파이프라인**은 내부적으로 **_단일 과정으로 실행할 수 있도록 최적화_** 됩니다. 즉, 쇼트서킷을 이용해서 결과를 찾는 즉시 실행을 종료합니다.

```java
Optional<Dish> dish =
    menu.stream()
        .filter(Dish::isVegetarian)
        .findAny();
```

위 코드에서 Optional은 무엇일까요?

<br>

### **Optional** 이란?

Optional<T.> 클래스는 값의 존재나 부재 여부를 표현하는 컨테이너 클래스입니다. findAny처럼 null요소를 반환할 수 있는 메소드가 쉽게 에러를 발생시킬 수 있기 때문에 만들어졌다고 합니다.

즉, Optional은 값이 존재하는지 확인하고, 값이 없을 때 어떻게 처리할지 강제하는 기능을 제공합니다.

- isPresent() : Optional이 값을 포함하면 true 반환, 아니면 false 반환
- ifPresent(Consumer<T.> block) : 값이 있으면 주어진 블록을 실행합니다. 이때 Consumer 함수형 인터페이스는 void를 반환하는 람다를 전달할 수 있습니다.
- T get() : 값이 존재하면 값을 반환, 없으면 NoSuchElementException을 일으킵니다.
- T orElse(T other) : 값이 있으면 값을 반환하고 없으면 기본값을 반환합니다.

Optional을 사용하면 값이 null인지 검사할 필요가 없습니다.

### ▷ 첫번째 요소 찾기

리스트나 정렬된 연속 데이터로부터 생성된스트림처럼 일부 스트림에는 **논리적인 아이템 순서가 정해져 있을 수도** 있습니다.

이런 스트림에서 첫 번째 요소를 찾고싶으면 findFirst를 이용할 수 있습니다.

```java
List<Integer> someNumbers = Arrays.asList(1,2,3,4,5);
Optional<Integer> firstSquareDivisibleByThree = someNumbers.stream()
                                                            .map(n -> n * n)
                                                            .filter(n -> n % 3 == 0)
                                                            .findFirst();
```

> **findFirst**와 **findAny**는 언제 활용할까 ?
>
> <br>
>
> 이 둘이 모두 필요한 이유는 **병렬성** 때문입니다. 병렬실행에서는 첫 번째 요소를 찾기 쉽지 않기 때문에, 요소의 반환 순서가 상관없다면 병렬 스트림에서는 제약이 적은 findAny를 사용합니다.
>
> <br>
> _

<br>
<hr>
<br>

## ■ 5. 리듀싱

<br>
