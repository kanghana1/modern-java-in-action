# Chapter 1 - 자바 8,9,10,11: 무슨일이 일어나고 있는가?


이 장에서 살펴볼 내용은 크게 아래와 같습니다.
* 자바가 거듭 변화하는 이유
* 컴퓨팅 환경의 변화
* 자바에 부여되는 시대적 변화 요구
* 자바 8과 자바 9의 새로운 핵심 기능 소개

하나씩 내용을 살펴보겠습니다.


## Java8
자바 역사를 통틀어 Java8에서 가장 큰 변화가 일어났습니다. 자바8을 통해 자연어에 더 가깝게 간단한 방식으로 코드를 구현할 수 있게 되었습니다.

멀티코어 CPU 대중화와 같은 하드웨어적인 변화가 일어났지만, 자바8 등장 이전, 멀티 코어의 나머지 코어를 활용하려면 스레드를 사용하는 것이 좋다고 누군가 조언했을 수도 있다고 합니다.

하지만, 스레드를 사용하면 관리하기 어렵고 많은 문제가 발생할 수 있는 단점이 존재합니다. 자바 8은 이러한 병렬 실행 환경을 쉽게 관리하고 에러가 덜 발생하는 방향으로 진화하려 노력했습니다.

자바 8의 새로운 기법을 이용하려면 몇가지 규칙을 지켜야 하며, 그 내용을 이 책에서 설명한다고 합니다.

> 자바 8은 **간결한 코드, 멀티코어 프로세서의 쉬운 활용**이라는 두 가지 요구사항을 기반으로 합니다.

이 두가지 목적성을 가지고 새로 나온 기술 3가지를 요약하자면 아래와 같습니다.

1. 스트림 API
2. 메서드에 코드를 전달하는 기법
3. 인터페이스의 디폴트 메서드

자바8은 데이터베이스 질의 언어에서 표현식을 처리하는 것처럼 **병렬 연산을 지원하는 스트림**이라는 새로운 API를 제공합니다.

- 데이터베이스 질의 언어에서 고수준 언어로 원하는 동작을 표현하면
- 구현(자바에서는 스트림 라이브러리가 이 역할 수행)에서 최적의 저수준 실행 방법을 선택하는 방식으로 동작합니다.
- 즉, 스트림을 이용하면 에러를 자주 일으키며 멀티코어 CPU를 이용하는 것 보다 비용이 훨씬 비싼 키워드 synchronized를 사용하지 않아도 됩니다.


이 스트림 API 덕분에 다른 두 가지 기능, 즉 **메서드에 코드를 전달하는 간결 기법**, 인터페이스의 **디폴트 메서드**가 존재 할 수 있음을 알 수 있습니다.

메서드에 코드를 전달하는 간결 기법(메서드 참조와 람다)을 이용하면 동작 파라미터화를 구현할 수 있습니다.

<br>

⏩ 자바 8은, ```함수형 프로그래밍```에서 위력을 발휘합니다.

## Java8의 변화
프로그래밍 언어는 마치 생태계와 닮았습니다. 새로운 언어가 등장하면서 진화하지 않은 기존 언어는 사장됩니다.

자바는 많은 유용한 라이브러리를 포함하는 잘 설계된 객체지향 언어로 시작했으며, 처음부터 스레드와 락을 이용한 소소한 동시성도 지원했습니다. 

코드를 JVM바이트 코드로 컴파일 하는 특징이 자바를 인터넷 애플릿 프로그램의 주요 언어가 되도록 하였고, JVM의 최신 업데이트 덕분에 경쟁언어는 JVM에서 더 부드럽게 실행될수 있었고 자바와 상호동작 할 수 있게 되었습니다. 또한 자바는 임데디드 컴퓨팅 분야를 성공적으로 장악하고 있습니다.

<br>

>하지만 프로그래밍 언어 생태계에 큰 바람이 불었습니다. 프로그래머는 빅데이터라는 도전에 직면하면서 멀티코어 컴퓨터나 컴퓨팅 클러스터를 이용해서 빅데이터를 효과적으로 처리할 필요성이 커졌습니다.

<br>

### 1. 스트림 처리
스트림이란 한 번에 한 개씩 만들어지는 연속적인 데이터 항목들의 모임입니다.

* 프로그램은 입력 스트림에서 데이터를 한개씩 읽어 들이며 마찬가지로 출력 스트림으로 데이터를 한 개씩 기록한다.
* 즉, 어떤 프로그램의 출력 스트림은 다른 프로그램의 입력 스트림이 될 수 있다.

유닉스의 경우, 파이프 라인을 통해 명령을 연결하여 병렬로 처리할 수 있다. 
```c
// 파일의 단어를 소문자로 바꾼 다음, 사전순으로 단어를 정렬했을 때 마지막 세 단어 출력
cat file1 file2 | tr "[A-Z]" "[a-z]" | sort | tail -3 
```
cat, tr이 완료되지 않은 시점에서 sort가 행을 처리하기 시작할 수 있다. 기계적인 예로, 여러 자동차 공장은 여러 자동차로 구성된 스트림을 받아 처리하는데, 각각의 작업장에서는 수리 한 뒤 물리적인 순서로 다음 작업장에 넘겨주지만, 각각의 작업장에서는 동시에 작업을 처리합니다.

* 스트림 API는 이처럼 조립 라인처럼 어떤 항목을 연속으로 제공하는 어떤 기능이라고 생각할 수 있으며
* 이 스트림 API는 파이프라인을 만드는 데 필요한 많은 메소드를 제공합니다.

#### 스트림 API의 핵심
1. 기존에 한 번에 한 항목을 처리했지만, 자바8에서는 작업을 고수준으로 추상화해서 일련의 스트림으로 만들어서 처리가 가능합니다.
2. 스트림 파이프라인을 이용해서 입력 부분을 여러 CPU 코어에 쉽게 할당할 수 있습니다.
3. 즉, 스레드라는 복잡한 작업을 사용하지 않으면서도 **공짜로** 병렬성을 얻을 수 있습니다.
   
요약하자면, 기존에 컬렉션 API를 사용할 때는 for루프로 각 요소에 대한 작업을 반복하였고 이는 외부반복 이였으나, 스트림 API는 라이브러리 내부에서 모든 작업을 처리 할 수 있어 루프를 신경쓰지 않아도 되며, 이는 **내부 루프**라고 부릅니다.

### 2. 동작 파라미터화로 메서드에 코드 전달하기
코드의 일부를 API로 전달하는 기능입니다. 자바 8이전의 자바에서는 메서드를 다른 메서드로 전달할 방법이 없었습니다. 자바 8에서는 메서드를 다른 메서드의 인수로 넘겨주는 기능을 제공합니다.

이러한 기능을 이론적으로 동작 파라미터화(Behavior parameterization)이라고 부릅니다. 이게 왜 중요한가? 라고 한다면 그 이유는 스트림API는 연산의 동작을 파라미터화할 수 있는 코드를 전달한다는 사상에 기초하기 때문입니다.

### 3.병렬성과 공유 가변 데이터
세 번째 프로그래밍 개념은 '병렬성을 공짜로 얻을 수 있다'라는 말에서 시작됩니다. 병렬성을 공짜로 얻는 대신, 스트림 메서드로 전달하는 코드의 동작 방식을 조금 바꿔야 합니다.

다른 코드와 동시에 실행하더라도 안전하게 실행할 수 있는 코드를 만들려면 공유된 가변데이터에 접근하지 않아야 합니다. 
> 이러한 함수를 순수(pure), 부작용 없는(side-effect-free), 상태 없는(stateless) 함수 라고 합니다.

하지만 공유된 변수나 객체가 있으면 병렬성에 문제가 발생합니다. 기존처럼 synchronized를 이용하여 공유된 가변 데이터를 보호하는 규칙을 사용할 수도 있지만, 자바 8을 이용하면 성능에 악영향을 미치지 않고 기존 자바 스레드 API보다 쉽게 병렬성을 활용할 수 있습니다.

공유되지 않은 가변 데이터, 메서드, 함수 코드를 다른 메서드로 전달하는 두가지 기능은 **함수형 프로그래밍** 패러다임의 핵심사항입니다.

반면, **명령형 프로그래밍**(imperative programming)에서는 일련의 가변 상태로 프로그램을 정의합니다.

공유되지 않은 가변 데이터 요구사항이란 인수를 결과로 변환하는 기능과 관련되며, 이 요구사항은 수학적인 함수처럼 정해진 기능만 수행함을 뜻합니다.


### 자바가 진화해야 하는 이유
자바는 제너릭(generic)이 나타나고, 컴파일시 더 많은 에러 검출, 리스트 유형을 알 수 있어 가독성이 좋아짐, Iterator대신 for-each루프의 사용 등의 변화를 겪어 왔습니다.

>기존의 값을 변화시키는 데 집중했던 고전적인 객체지향에서 벗어나 함수형 프로그래밍으로 다가섰다는 것이 자바 8의 가장 큰 변화이다.

전통적인 객체지향 프로그래밍과 함수형 프로그래밍은 극단적으로 생각하였을때 완전 상극이다. 자바8에서는 함수형 프로그래밍을 도입함으로써 두 가지 프로그래밍 패러다임의 장점을 모두 활용할 수 있게 되었습니다.

* 언어는 하드웨어나 프로그래머 기대의 변화에 부응하는 방향으로 변해야 합니다.
  
자바의 위치가 견고하지만 코볼과 같은 언어의 선례를 떠올리면 자바가 영원히 지배적인 위치를 유지할 수 있는 것은 아닐 수 있다는 말 입니다.

## 1.3 자바 함수
프로그래밍 언어에서 함수(function)이라는 용어는 메서드 특히 정적 메서드와 같은 의미로 사용된다.

자바 8에서는 함수를 새로운 값의 형식으로 추가했다. 이는 멀티코어에서 병렬 프로그래밍을 활용할 수 있는 스트림과 연계될 수 있도록 함수를 만들었기 떄문이다. 먼저 함수를 값처럼 취급한다고 했는데 이 특징이 어떤 장점을 제공하는지 살펴보겠습니다.

자바 프로그램에서 조작할 수 있는 값은
* int, double등의 기본값
* 객체 (엄밀히 따지면 객체의 참조)
  * 객체 참조는 클래스의 instance를 가리킴
  * 심지어 배열도 객체다

<br>

**그런데 왜 함수가 필요한가?**
프로그래밍 언어의 핵심은 값을 바꾸는 것입니다. 역사적으로 그리고 전통적으로 프로그래밍 언어에서는 이 값을 일급값 (first-class) 혹은 일급시민 이라고 부릅니다.

자바 프로그래밍 언어의 다양한 구조체가 값을 표현하는데 도움이 될 수 있지만, 프로그램을 실행하는 동안 이러한 모든 구조체를 자유롭게 전달할 수는 없습니다. 이렇게 전달할 수 없는 구조체는 이급 시민입니다. 위에서 언급한 값은 모두 일급 자바 시민이지만 메서드, 클래스 등은 이급 자바시민에 해당합니다.

인스턴스화한 결과가 값으로 귀결되는 클래스를 정의할 때 메서드를 아주 유용하게 활용할 수 있지만 여전히 메서드와 클래스는 그 자체로 값이 될 수 없습니다. 하지만 메서드를 일급 시민으로 만들면 프로그래밍에 유용하게 활용할 수 있습니다.

따라서 자바 8 설계자들은 이급 시민을 일급 시민으로 바꿀 수 있는 기능을 추가했습니다.

### 1.3.1 메서드와 람다를 일급 시민으로
메서드를 일급값으로 사용하면 프로그래머가 활용할 수 있는 도우가 다양해지면서 프로그래밍이 수월해집니다.

그래서 자바 8 설계자들은 메서드를 값으로 취급할 수 있게, 그래서 프로그래머들이 더 쉽게 프로그램을 구현할 수 있는 환경이 제공되도록 하였습니다.

#### 첫번째, 메서드 참조 (method reference)

아래, 디렉토리에서 모든 숨겨진 파일을 필터링 해주는 코드가 있습니다.

```java
File[] hiddenFiles = new File(".").listFiles(new FileFilter() {
  public boolean accept(File file){
    return file.isHidden(); //숨겨진 파일 핕터링
  }
});
```
세 행의 코드가 있지만 각 행이 무슨 작업을 하는지 투명하지 않습니다. File 클래스에 이미 isHidden이라는 메서드가 있는데 굳이 FileFilter로 isHidden을 복잡하게 감싼 다음에 FileFilter를 인스턴스화 해야할까요? 자바 8 이전에는 달리 방법이 없었습니다.

하지만 자바 8에서는 다음처럼 한줄로 코드를 구현할 수 있습니다.
```java
File[] hiddenFiles = new File(".").listFiles(File::isHidden);
```
이미 isHidden이라는 **함수**가 준비되어 있으므로 자바 8의 **메서드 참조** ::('이 메서드를 값으로 사용하라'는 의미) 를 이용하여 listFiles에 직접 전달 할 수 있습니다. 기존에 비해 문제 자체를 더 직접적으로 설명한다는 점이 자바 8의 코드의 장점입니다.

자바 8에서는 더 이상 메서드가 이급값이 아닌 일급값 입니다. 기존의 객체 참조(object reference)(new로 객체 참조를 생성함)를 이용해서 객체를 이리저리 주고 받았던 것 처럼 자바 8에서는 `File::isHidden`을 이용해서 메서드 참조를 만들어 전달할 수 있게 되었습니다.

#### 람다 : 익명 함수(Anonymous function)
자바 8에서는 메서드를 일급값으로 취급할 뿐 아니라 **람다**를 포함하여 함수도 값으로 취급할 수 있다. 예를 들자면 `(int x) -> x+1` 즉, 'x라는 인수로 호출하면 x+1을 반환'하는 동작을 수행하도록 코드를 구현할 수 있습니다.

직접 메서드를 정의할 수도 있지만, 이용할 수 있는 편리한 클래스나 메서드가 없을 때 새로운 람다 문법을 이용하면 더 간결하게 코드를 구현할 수 있다는 장점이 있습니다.

* 람다 문법 형식으로 구현된 프로그램을 **함수형 프로그래밍**, 즉 '함수를 일급값으로 넘겨주는 프로그램을 구현한다'라고 합니다.


### 1.3.2 코드 넘겨주기 : 예제
Apple 클래스와 getColor 메서드가 있고, Apples 리스트를 포함하는 변수 inventory가 있다고 가정하여, 모든 녹색 사과를 선택해서 리스트를 반환하는 프로그램을 구현하려고 합니다. 이처럼 특정 항목을 선택해서 반환하는 동작을 **필터(filter)**라고 합니다.

* 자바 8 이전
```java
public static List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if ("green".equals(apple.getColor())) {
        result.add(apple);
      }
    }
    return result;
}
```
사과를 무게로 필터링하는 경우
```java
public static List<Apple> filterHeavyApples(List<Apple> inventory) {
	List<Apple> result = new ArrayList<>();
	for (Apple apple: inventory) {
		if (apple.getWeight() > 150){
			result.add(apple);
		}
	}
	return result;
}
```
소프트웨어공학적인 면에서 복사&붙여넣기의 단점 -> 어떤 코드에 버그가 있다면 복사&붙여넣기한 모든 코드를 고쳐야한다는 점입니다. 위 예제에서 두 메서드는 if문 한 줄의 코드만 다릅니다. 이러한 위험에서 안전하지 않습니다.

자바 8에서는, 코드를 인수로 넘겨줄 수 있으므로 filter 메서드를 중복으로 구현할 필요가 없습니다. 자바 8에서는 다음과 같이 구현할 수 있습니다.
```java
public static boolean isGreenApple(Apple apple){
	return GREEN.equals(apple.getColor());
}
public static boolean isHeavyApple(Apple apple){
	return apple.getWeight() > 150;
}
public interface Predicate<T>{
	boolean test(T, t); //명확히 하기 위해 추가함 (보통 java.util.function에서 임포트 함)
}

static List<Apple> filterApples(List <Apple> inventory, Predicate<Apple> p){
	List<Apple> result = new ArrayList<>();
	for (Apple apple : inventory){
		if (p.test(apple)){
			result.add(apple);
		}
	}
	return result;
}
```
다음과 같이 호출할 수 있습니다.
```java
filterApples(inventory, Apple::isGreenApple);
filterApples(inventory, Apple::isHeavyAplle);
```

* 프리디케이트란 무엇인가?

  * filterApples는 Apple::isGreenApple 메서드를 Predicate<Apple>이라는 타입의 파라미터로 받습니다. 인수로 값을 받아 true나 false를 반환하는 함수를 predicate라고 합니다. 
  
  * Function<Apple, boolean>같이 코드를 구현할 수 있지만 Predicate<Apple>을 사용하는 것이 더 표준적인 방식입니다.(boolean을 Boolean으로 변환하는 과정이 없으므로 더 효율적이기도 합니다)

> 여기서 핵심은, 자바 8에서는 메서드를 전달할 수 있다는 사실입니다.

### 1.3.3 메서드 전달에서 람다로
메서드를 값으로 전달하는 기능은 유용한 기능입니다. 하지만 한두 번만 사용할 메서드를 매번 정의하는 것을 귀찮은 일입니다. 자바 8에서는 이 문제를 (익명 함수 또는 람다라는) 새로운 개념을 이용하여 코드를 구현할 수 있습니다.

```java
filterApples(inventory, (Apple a) -> GREEN.equals(a.getColor()));
filterApples(inventory, (Apple a) -> a.getWeight() > 150);
filterApples(inventory, (Apple a) -> a.getWeight() > 150 || GREEN.equals(a.getColor()));
```
위와 같이 따로 정의를 구현할 필요없이 구현할 수 있습니다.

하지만 람다가 몇 줄 이상으로 길어진다면 익명 람다보다는 코드가 수행하는 일을 잘 설명하는 이름을 가진 메서드를 정의하고 메서드 참조를 이용하는것이 좋습니다. **코드의 명확성이 우선**이기 때문입니다.

병렬성이라는 문제 때문에, filter 그리고 일반적인 라이브러리 메서드를 추가하는 방향으로 발전했을 수도 있지만 그 대신 자바 8은 filter와 비슷한 동작을 수행하는 연산집합을 포함하는 새로운 스트림 API(컬렉션과 비슷하며 함수형 프로그래머에게 더 익숙한 API)를 제공합니다. 또한 컬렌션과 스트림간에 변환할 수 있는 메서드(map,reduce 등)도 제공합니다.

## 1.4 스트림
거의 모든 자바 애플리케이션은 컬렉션을 만들고 활용합니다. 하지만 그게 모든 문제를 해결해주지 않습니다.

스트림 API를 이용하면 컬렉션 API와는 상당히 다른 방식으로 데이터를 처리할 수 있습니다. 컬렉션에서는 for-each 루프를 이용하여 반복 과정을 직접 처리해야하며 이런 방식은 **외부 반복**(external iteration)이라고 합니다. 반면 스트림 API를 이용하면 루프를 신경쓸 필요가 없으며 라이브러리 내부에서 모든 데이터가 처리되고, 이런 방식을 **내부 반복**(internal iteration)이라고 합니다.

컬렉션을 이용하여 많은 요소를 가진 목록을 반복한다면 오랜 시간이 걸릴 수 있습니다. 거대한 리스트의 경우 단일 CPU로는 거대한 데이터를 처리하기 어려울 것입니다. 멀티코어 환경에서는 서로 다른 CPU 코어에 작업을 각각 할당하여 처리 시간을 줄일 수 있으며 자바 8에서는 이런 컴퓨터를 더 잘 활용할 수 있는 새로운 프로그래밍 스타일을 제공합니다.

### 1.4.1 멀티스레딩은 어렵다
이전 자바 버전에서 제공하는 스레드 API로 멀티스레딩 코드를 구현해서 병렬성을 이용하기는 쉽지 않았습니다. 멀티스레딩 환경에서 각각의 스레드는 동시에 공유된 데이터에 접근하고, 데이터를 갱신할 수 있으나, 잘 제어하지 못하면 원치 않는 방식으로 데이터가 바뀔 수 있습니다.

자바 8은 스트림API(java.util.stream)로 두가지 문제를 해결했습니다.
* 컬렉션을 처리하면서 발생하는 모호함과 반복적인 코드 문제
* 멀티코어 활용 어려움

라이브러리에서 반복되는 패턴을 제공한다면 좋을 것이라는 아이디어는 하나의 변화 동기가 되었습니다. 스트림 API는 자주 반복되는 패턴으로 주어진 조건에 따라 데이터를 필터링(filtering)하거나, 데이터를 추출(extracting), 데이터를 그룹화(grouping)하는 등의 기능을 제공합니다.

이러한 동작들을 쉽게 병렬화할 수 있다는 점도 변화의 동기가 되었습니다. 두 CPU를 가진 환경에서 리스트 앞 뒤부분을 각각 담당하도록 요청할 수 있으며 (이 과정을 **포킹 단계**forking step 라고 한다.) 마지막으로 하나의 CPU가 두 결과를 정리하도록 합니다. 스트림은 **스트림 내의 요소를 쉽게 병렬로 처리할 수 있는 환경을 제공**한다는 것이 핵심입니다.

* 컬렉션은 어떻게 데이터를 저장하고 접근할지에 중점
* 스트림은 데이터에 어떤 계산을 할지에 중점

다음은 순차 처리 방식과 병렬 처리 방식의 코드입니다.
```java
List<Apple> heavyApples = inventory.**stream()**.filter((Apple a)-> a.getWeight() > 150).collect(toList());

List<Apple> heavyApples = inventory.**parallelStream()**.filter((Apple a) -> a.getWeight() > 150).collect(toList());
```

## 1.5 디폴트 메서드와 자바 모듈
자바 변화 과정에서 자바 8 개발자들이 겪는 어려움 중 하나는 기존 인터페이스의 변경입니다. 인터페이스를 업데이트 하려면 해당 인터페이스를 구현하는 모든 클래스도 업데이트 해야하기 때문에 사실상 불가능 했기 때문입니다. 자바 8과 9는 이 문제를 다르게 해결합니다.

자바 9는 패키지 모음을 포함하는 **모듈**을 정의할 수 있으며 이 덕분에 JAR같은 컴포넌트에 구조를 적용할 수 있으며 문서화와 모듈 확인 작업이 용이해졌습니다.

자바 8에서는 인터페이스를 **쉽게 바꿀 수 있도록** 디폴트 메서드를 지원합니다. 디폴트 메서드는 특정 프로그램을 구현하는 데 도움을 주는 기능이 아니라 미래에 프로그램이 쉽게 변화할 수 있는 환경을 제공하는 기능이다.

```java
List<Apple> heavyApples = inventory.**stream()**.filter((Apple a)-> a.getWeight() > 150).collect(toList());

List<Apple> heavyApples = inventory.**parallelStream()**.filter((Apple a) -> a.getWeight() > 150).collect(toList());
```
자바 8 이전에는 List<T>가 stream이나 parallelStream 메서드를 지원하지 않았다는 것이 문제였습니다. 이 해결책은 Collection 인터페이스에 stream 메서드를 추가하고 ArrayList 클래스에 메서드를 구현하는 것 입니다. 

하지만 이미 컬렉션 API의 인터페이스를 구현하는 많은 컬렉션 프레임워크가 존재합니다. 인터페이스에 새로운 메서드를 추가한다면 인터페이스를 구현하는 모든 클래스는 새로 추가된 메서드를 구현해야 하기에 현실적으로 어렵습니다.

자바8은 기존의 구현을 고치지 않기 위해 고민하였고, 결과적으로 자바 8은 구현 클래스에서 구현하지 않아도 되는 메서드를 인터페이스에 추가할 수 있는 기능을 제공합니다. 메서드 본문(bodies)는 클래스 구현이 아니라 인터페이스의 일부로 포함됩니다. 그래서 이를 디폴트 메서드라고 부릅니다.

이를 이용하면 기존의 코드를 건드리지 않고도 원래의 인터페이스 설계를 자유롭게 확장할 수 있습니다.(default 라는 새로운 키워드를 지원)
9장에서 악명 높은 **다이아몬드 상속 문제**를 피할 수 있는 방법도 알아보도록 하겠습니다.

## 1.6 함수형 프로그래밍에서 가져온 다른 유용한 아이디어
일반적으로 함수형 언어도 프로그램을 돕는 여러 장치를 제공합다. 일례로 명시적으로 서술형의 데이터 형식을 이용해 null을 회피하는 기법이 있습니다.

자바 8에서는 NullPointerException을 피할 수 있는 `Optional<T>` 클래스를 제공합니다. 이 클래스는 값을 갖거나 갖지 않을 수 있는 컨테이너 객체입니다. `Optional<T>` 는 값이 없는 상황을 어떻게 처리할지 명시적으 로 구현하는 메서드를 포함하고 있다. 11장에서 더 자세히 설명합니다.

또한 구조적 패턴 매칭 기법도 있습니다. 자바에서는 if-then-else나 switch문을 이용하지만 다른 언어에서는 패턴 매칭으로 더 정확한 비교를 구현할 수 있다는 사실을 증명했습니다. 아쉽게도 자바 8은 패턴 매칭을 완벽하게 지원하지 않는습니다. 현재는 자바 개선안으로 제안된 상태입니다.

* 패턴 매칭은 Switch를 확장한 것으로 데이터 형식 분류와 분석을 한 번에 수행할 수 있다
* 객체지향 설계에서 클래스 패밀리를 방문할 때 방문자 패턴을 이용해서 각 객체를 방문한 다음에 원하는 작업을 수행한다.

## 1.7 마치며
1. 언어 생태계의 모든 언어는 변화해서 살아남거나 그대로 머물면서 사라지게 된다. 지금은 자바의 위치가 견고하지만 코볼과 같은 언어의 선례를 떠올리면 자바가 영원히 지배적인 위치를 유지할 수 있는 것은 아닐 수 있다.
2. 자바 8은 프로그램을 더 효과적이고 간결하게 구현할 수 있는 새로운 개념과 기능을 제공한다.
3. 기존의 자바 프로그래밍 기법으로는, 멀티코어 프로세서를 온전히 활용하기 어렵다.
4. 함수는 일급 값이다. 메서드를 어떻게 함수형값으로 넘겨주는, 익명 함수(람다)를 어떻게 구현하는지 기억하자
5. 자바 8의 스트림 개념 중 일부는 컬렉션에서 가져온 것이다, 스트림과 컬렉션을 적절하게 활용하면 스트림의 인수를 병렬로 처리할 수 있으며 더 가독성이 좋은 코드를 구현할 수있다.
6. 기존 자바 기능으로는 대규모 컴포넌트 기반 프로그래밍 그리고 진화하는 시스템의 인터페이스를 적절하게 대응하기 어려웠다, 디폴트 메서드를 이용해 기존 인터페이스를 구현하는 클래스를 바꾸지 않고도 인터페이스를 변경할 수 있다.
7. 함수형 프로그래밍에서 NULL 처리 방법과 패턴 매칭 화룡 등 흥미로운 기법을 발견했다.