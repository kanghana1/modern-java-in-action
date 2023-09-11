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
