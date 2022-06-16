# CHAPTER 2 동작 파라미터화 코드 전달하기

발표자: 장현준
스터디일정: 2022/04/28 오후 6:00
주차별: 2주차
참여자: 권윤옥, 남채린, 이승민, 장현준

<aside>
💡 1. 변화하는 요구사항에 대응
2. 동작 파라미터화
3. 익명 클래스
4. 람다 표현식 미리보기
5. 실전 예제 : Comparator, Runnable, GUI

</aside>

# 동작 파라미터화 코드 전달

- 요구사항은 항상 바뀐다. 특히 우리의 엔지니어링적인 비용이 가장 최소화될 수 있으면 좋으며, 새로 추가한 기능은 쉽게 구현할 수 있어야한다. 또한 장기적인 관점에서 유지보수가 쉬워야한다.
- **동작 파라미터화** 를 이용하면 자주 바뀌는 요구사항에 효과적으로 대응할 수 있다.
💁🏻 **동작 파라미터화** 란 아직은 어떻게 실행할 것인지 결정하지 않은 코드 블록이다.

### 동작 파라미터화의 예시(리스트)

1. 리스트의 모든 요소에 대해서 `어떤 동작` 을 수행할 수 있다.
2. 리스트 관련 작업을 끝낸 다음에 `어떤 다른 동작` 을 수행할 수 있다.
3. 에러가 발생하면 `정해진 어떤 다른 동작` 을 수행할 수 있다.

# 변화하는 요구사항에 대응하기

### 예시

- 기존의 농장 재고목록 애플리케이션에 리스트에 녹색사과만 필터링하는 기능을 추가한다고 하자.

## 첫 번째 시도 : 녹색 사과 필터링

```java
enum Color {RED, GREEN}

public static List<Apple> filterGreenApples(List<Apple> inventory) {
	List<Apple> result = new ArrayList<>(); //사과 누적 리스트
	for (Apple apple : inventory) {
		if (GREEN.equals(apple.getColor()) { // 녹색 사과만 선택
			result.add(apple);
		}
	}
}
```

- 만약 농부가 녹색말고 빨간사과도 필터링 해달라고 요구하면? 노랑? 파랑? ~~run해야지~~  메서드를 복사해서 `filterRedApple` 메서드를 만들거나 if문의 조건을 주는 선택을 할 수 있다.하지만 계속해서 변화하는 요구사항에 적절한 해결방법은 아니다.
- `거의 비슷한 코드가 반복 존재한다면 그 코드를 추상화한다.`

## 두 번째 시도 : 색을 파라미터화

```java
public static List<Apple> filterAppleByColor(List<Apple> inventory, Color color) {
	List<Apple> result = new ArrayList<>();
	for (Apple apple : inventory) {
		if (apple.getColor().equals(color)) {
			result.add(apple);
		}
	}
}
```

- 실제 사용
List<Apple> greenApples = filterAppleByColor(inventory, GREEN);
List<Apple> redApples = filterAppleByColor(inventory, RED);

- 다시 농부가 나타나서 **색 이외에도 가벼운 사과 무거운 사과로 구분할 수 있게 요구한다면??** ~~하...~~

```java
public static List<Apple> filterAppleByWeight(List<Apple> inventory, int weight) {
	for(Apple apple : inventory) {
		if ( apple.getWeigth() > weigth ) {
			...
		{
	}
}
```

- 이처럼 위에 3개의 코드도 괜찮은 해결책이라고 할 수 있다. 하지만 소프트웨어 공학의 DRY (don`t repeat yourself) 원칙을 어기게된다.

## 세 번째 시도 : 가능한 모든 속성으로 필터링

→ 굳이 된장과 똥을 찍어먹어봐야 하는 타입 코드 ( ~~그게 바로 나~~ )

```java
public static List<Apple> filterApples(List<Apple> inventory, Color color, int weight, boolean flag) {
	List<Apple> result = new ArrayList<>();
	for(){
		if((flag && apple.getColor().equals(color)) || (!flag && apple.getWeight() > weight)) {
				result.add(apple);	
		}
	}	
}

```

- 실제 사용
List<Apple> greenApples = filterApples(inventory, GREEN, 0, true);
List<Apple> heavyApples = filterApples(inventory, null, 150, false);

# 동작 파라미터화

### 프레디케이트(Predicate)

- 참 또는 거짓을 반환하는 함수를 프레디케이트 라고 한다. **선택 조건을 결정하는 인터페이스**를 정의하자.

```java
public interface ApplePredicate {
	boolean test (Apple apple);
}
```

### 다양한 선택 조건을 대표하는 여러 버전의 ApplePredicate를 정의할 수 있다.

```java
public class AppleHeavyWeigthPredicate implements ApplePredicate {
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

![Untitled](CHAPTER%202%20%E1%84%83%E1%85%A9%E1%86%BC%E1%84%8C%E1%85%A1%E1%86%A8%20%E1%84%91%E1%85%A1%E1%84%85%E1%85%A1%E1%84%86%E1%85%B5%E1%84%90%E1%85%A5%E1%84%92%E1%85%AA%20%E1%84%8F%E1%85%A9%E1%84%83%E1%85%B3%20%E1%84%8C%E1%85%A5%E1%86%AB%E1%84%83%E1%85%A1%E1%86%AF%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%208e452655982f4ec2a357b5295acdc549/Untitled.png)

- [전략 디자인 패턴](https://victorydntmd.tistory.com/292)
→ 간단히 말해서 객체가 할 수 있는 행위들 각각을 전략으로 만들어 놓고, 동적으로 행위의 수정이 필요한 경우 전략을 바꾸는 것만으로 행위의 수정이 가능하도록 만든 패턴입니다.
→ 각 알고리즘(전략이라 불리는) 을 캡슐화 하는 알고리즘 패밀리를 정의해둔 다음에 런타임에 알고리즘을 선택하는 기법이다. [WIKI](https://ko.wikipedia.org/wiki/%EC%A0%84%EB%9E%B5_%ED%8C%A8%ED%84%B4)
- 다시 본론으로 돌아와서 ApplePredicate가 알고리즘 패밀리이며, **AppleHeavyWeightPredicate**와 **AppleGreenColorPredicate가 전략임.**

### 어떻게 ApplePredicate에서 다양한 동작을 수행할 수 있을까?

- **filterApples**에서 **ApplePredicate**객체를 받아 애플의 조건을 검사하도록 메서드를 고쳐야한다.
→ 이렇게 **동작 파라미터화,** 즉 메서드가 다양한 동작(또는 전략)을 **받아서** 내부적으로 다양한 동작을 **수행** 할 수 있다.

## 네 번째 시도 : 추상적 조건으로 필터링

```java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
	List<Apple> result = new ArrayList<>();
	for (Apple apple : inventory) {
		if (p.test(apple)) { // 프레디케이트 객체로 사과 검사 조건을 캡슐화 했다.
			result.add(apple);
		}
	}
	return result;
}
```

## 💁🏻 QUIZ

- 농부가 150그램이 넘는 빨간 사과를 검색해달라고 부탁하면 어떻게 코드를 작성해야할까요?

```java
public class AppleRedAndHeavyPredicate implements ApplePredicate {
	public boolean test(Apple apple) {
		return RED.equals(apple.getColor()) && apple.getWeight() > 150;
	}
}

List<Apple> redAndHeavyApples = filterApples(inventory, new AppleRedAndHeavyPredicate());
```

## 한 개의 파라미터, 다양한 동작

```java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
	...
}

ApplePredicate p 매개변수에 여러가지 메서드를 넣었으며, 모두 다 다르게 동작하였다.
```

## 결론

- 지금까지 동작을 추상화해서 변화하는 요구사항에 대응할 수 있는 코드를 구현하는 방법을 살펴봤다. 하지만 여러 클래스를 구현해서 인스턴스화 하는 과정이 거추장스럽게 느껴진다.

# 복잡한 과정 간소화

- 현재 위의 예시에서 filterApples 메서드로 새로운 동작을 전달하려면 ApplePredicate 인터페이스를 구현하는 여러 클래스를 정의한 다음에 인스턴스화 해야함... 상당히 번거로운 작입이며 시간 낭비이다.

```java
static class AppleWeightPredicate implements ApplePredicate {

    @Override
    public boolean test(Apple apple) {
      return apple.getWeight() > 150;
    }

  }

  static class AppleColorPredicate implements ApplePredicate {

    @Override
    public boolean test(Apple apple) {
      return apple.getColor() == Color.GREEN;
    }

  }

public class FilteringApples {

  public static void main(String... args) {
    List<Apple> inventory = Arrays.asList(
        new Apple(80, Color.GREEN),
        new Apple(155, Color.GREEN),
        new Apple(120, Color.RED));

		List<Apple> heavyApples = filterApples(inventory, new AppleWeightPredicate());
		List<Apple> greenApples = filterApples(inventory, new AppleColorPredicate());
	}
	
	public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
	List<Apple> result = new ArrayList<>();
	for (Apple apple : inventory) {
		if (p.test(apple)) { // 프레디케이트 객체로 사과 검사 조건을 캡슐화 했다.
			result.add(apple);
		}
	}
	return result;
}
}
```

- 자바는 클래스의 선언과 인스턴스화를 동시에 수행할 수 있도록 **익명 클래스** 라는 기법을 제공한다. **익명 클래스** 를 이용하면 위와 같은 코드의 양을 줄일 수 있다.

### 익명클래스

- 자바의 지역 클래스(블록 내부에 선언된 클래스)와 비슷한 개념이다. 익명 클래스는 말 그대로 이름이 없는 클래스이다. 익명 클래스를 이용하면 클래스 선언과 인스턴스화를 동시에 할 수 있다.

## 다섯 번째 시동 : 익명 클래스 사용

```java
List<Apple> redApples = filterApple(inventory, new ApplePredicate() {
	public boolean test(Apple apple) {
		return RED.quals(apple.getColor());
  }
});
```

### 실전예시(GUI 프로그래밍)

```java
button.setOnAction(new EventHandler<ActionEvent>() {
	public void handle(ActionEvent event) {
		System.out.println("Hi");
	}
}
```

### 익명 클래스의 단점

1. 코드가 장황하다.
2. 익명 클래스의 사용에 익숙하지 않다.
    - 구현하고 유지보수하는 데 시간이 오래걸림

```java
public class MeaningOfThis {
	public final int value = 4;
	public void doIt() {
		int value = 6;
		Runnable r = new Runnable() {
			public final int value = 5;
			public void run() {
				int value = 10;
				System.out.println(this.value);
			}
		};
		r.run();
	}
	public static void main(String...args) {
		MeaningOfThis m = new MeaningOfThis();
		m.doIt(); // 이행의 결과는?
	}
}
```

## 여섯 번째 시도 : 람다 표현식 사용 (맛보기)

```java
List<Apple> result = 
				filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```

## 일곱 번째 시도 : 리스트 형식으로 추상화

```java
public interface Predicate<T> {
	boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p) {
	List<T> result = new ArrayList<>();
	for(T l : list) {
		if(p.test(l)) {
			result.add(e);
		}
	}
	return result;
}

//실제 사용
List<Apple> redApples = filter(inventory, (Apple apple) -> RED.equals(apple.getColor()));
List<Integer> numbers = filter(numbers, (Integer i) -> i % 2 == 0);
```

# 실전예제

## Comparator로 정렬하기

```java
// java.util.Comparator
public interface Comparator<T> {
	int compare(T o1, T o2);
}

//익명 클래스
inventory.sort(new Comparator<Banana>() {
	public int compare(Banana b1, Banana b2) {
		return b1.getWeight().compareTo(b2.getWeight());
	}
}

//람다 표현식
inventory.sort((Banana b1, Banana b2) -> b1.getWeight().compareTo(b2.getWeight()));
```

## Runnable로 코드 블록 실행하기

```java
// java.lang.Runnable
public interface Runnable {
	void run();
}

//익명 클래스
Thread t = new Thread(new Runnable() {
	public void run() {
		System.out.println("Thread");
	}
}

//람다 표현식
Thread t = new Thread(() -> System.out.println("Thread"));
```

# 마치며...

- 동작 파라미터화에서는 메서드 내부적으로 다양한 동작을 수행할 수 있도록 코드를 메서드 인수로 전달한다.
- 동작 파라미터화를 이용하면 변화하는 요구사항에 더 잘 대응할 수 있는 코드를 구현할 수 있으며 나중에 엔지니어링 비용을 줄 일 수 있다.
- 코드 전달 기법을 이용하면 동작을 메서드의 인수로 전달할 수 있다.