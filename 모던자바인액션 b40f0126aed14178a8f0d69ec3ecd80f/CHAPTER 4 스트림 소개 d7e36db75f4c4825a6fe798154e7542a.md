# CHAPTER 4 스트림 소개

발표자: 남채린
스터디일정: 2022/05/04 오전 8:30
주차별: 3주차
참여자: 권윤옥, 남채린, 이승민, 장현준

> SQL 질의 언어처럼 우리가 기대하는 것이 무엇인지 직접 표현할 수 없을까...?
> 
> 
> 많은 요소를 포함하는 커다란 컬렉션은 어떻게 처리해야 할까...?
> 

## 스트림이란 무엇인가?

스트림은 자바 8 API에 새로 추가된 기능으로 스트림을 이용하면 **선언형으로** 컬렉션 데이터를 처리할 수 있다.

```java
List<Dish> lowCaloricDishes = new ArrayList<>();
for(Dish dish:menu) { <-- 누적자로 요소 필터링
		if(dish.getCalories() < 400) {
				lowCaloricDishes.add(dish);
		}
}
Collectors.sort(lowCaloricDishes, new Comparator<Dish>() {
		public int compare(Dish dish1, Dish dish2) {
				return Integer.compare(dish1.getCalories(), dish2.getCalories());
		}
});
List<String> lowCaloricDishesName = new ArrayList<>();
for(Dish dish: lowCaloricDishes) {
		lowCaloricDishesName.add(dish.getName());
}
```

기존코드에서 `lowCaloricDishes`라는 가비지 변수를 사용했다. 이는 컨테이너 역할만 하는 중간 변수다.

```java
import static java.util.Comparator.comparing;
import static java.util.stream.Collectors.toList;

List<String> lowCaloricDishesName = 
						menu.stream()
								.filter(d -> d.getCalories() < 400)
								.sorted(comparing(Dish::getCalories))
								.map(Dish::getName)
								.collect(toList());
```

🤔 스트림이란? 데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소

```java
List<String> threeHighCaloricDishNames = 
		menu.stream()
				.filter(dish -> dish.getCalories(0 > 300)
				.map(Dish::getName)
				.limit(3)
				.collect(toList());
```

**데이터 소스**(요리 리스트_메뉴)는 **연속된 요소**를 스트림에 제공 

→ 스트림에 filter, map, limit, collect로 이어지는 일련의 **데이터 처리 연산**을 적용 

→ collect를 제외한 모든 연산은 서로 **파이프라인**을 형성할 수 있도록 스트림을 반환 

→ collect 연산으로 파이프라인을 처리해서 결과를 반환(스트림 반환 X, List를 반환)

### 스트림의 특징

- 파이프라이닝
    
    : 스트림 연산끼리 연결해서 커다란 파이프라인을 구성할 수 있도록 스트림 자신을 반환한다.
    
- 내부 반복
    
    : 반복자를 이용해서 명시적으로 반복하는 컬렉션과 달리 스트림은 내부 반복을 지원한다.
    

<aside>
💡 **스트림 API의 특징** 
• **선언형** : 더 간결과 가독성이 좋아진다.
• **조립할 수 있음** : 유연성이 좋아진다.
• **병렬화** : 성능이 좋아진다.

</aside>

## 컬렉션과 스트림

데이터를 언제 계산하느냐의 차이!

모든 값을 메모리에 저장 VS 요청할 때만 계산

![스트림과 컬렉션의 차이](CHAPTER%204%20%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%20%E1%84%89%E1%85%A9%E1%84%80%E1%85%A2%20d7e36db75f4c4825a6fe798154e7542a/3674F8C4-957F-492C-9285-63DE3728F2FE.jpeg)

스트림과 컬렉션의 차이

++ 스트림은 반복자와 마찬가지로 한 번만 탐색할 수 있다. 한 번 탐색한 요소를 다시 탐색하려면 초기 데이터 소스에서 새로운 스트림을 만들어야 한다.

- Quiz❓
    
    아래 코드를 실행할 경우 출력되는 값을 예상해보세요!
    
    ```java
    List<String> title = Arrays.asList("Java8", "In", "Action");
    Stream<String> s = title.stream();
    s.forEach(System.out::printIn);
    s.forEach(System.out::printIn);
    ```
    

## 내부 반복과 외부 반복

컬렉션과 스트림의 또다른 차이점인 데이터 반복 처리 방법

컬렉션 인터페이스를 사용하려면 사용자가 직접 요소를 반복해야 한다.(ex. for-each) ⇒ **외부 반복**

스트림 라이브러리는 함수에 어떤 작업을 수행할지만 지정하면 모든 것이 알아서 처리 ⇒ **내부 반복**

![9F186CB8-E66B-4FFC-9B92-9D2BAB2870F9.jpeg](CHAPTER%204%20%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%20%E1%84%89%E1%85%A9%E1%84%80%E1%85%A2%20d7e36db75f4c4825a6fe798154e7542a/9F186CB8-E66B-4FFC-9B92-9D2BAB2870F9.jpeg)

컬렉션은 외부적으로 반복, 명시적으로 컬렉션 항목을 하나씩 가져와서 처리

내부 반복을 이용하면 작업을 투명하게 병렬로 처리하거나 더 최적화된 다양한 순서로 처리 가능

## 중간 연산과 최종 연산

```java
List<String> names = menu.stream() 
												 .filter(dish -> dish.getCalories() > 300) <-- 중간 연산
												 .map(Dish::getName) <-- 중간 연산
												 .limit(3) <-- 중간 연산
												 .collect(toList()); <-- 스트림을 리스트로 변환
```

### 연결할 수 있는 스트림 연산을 **중간 연산**

: filter, map, limit, sorted 같은 중간 연산은 다른 스트림을 반환한다. 따라서 여러 중간 연산을 연결해서 질의를 만들 수 있다.

⭐️  중간 연산의 중요한 특징은 단말 연산을 스트림 파이프라인에 실행하기 전까지는 아무 연산도 수행하지 않는다는 것 = **lazy!!!**

### 스트림을 닫는 연산을 **최종 연산**

: 최종 연산은 스트림 파이프라인에서 결과를 도출한다. 최종 연산에 의해 List, Integer, void 등 스트림 이외의 결과가 반환된다.

![1DB13FA7-E203-40DB-AB86-2714DBBBCF29.jpeg](CHAPTER%204%20%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%20%E1%84%89%E1%85%A9%E1%84%80%E1%85%A2%20d7e36db75f4c4825a6fe798154e7542a/1DB13FA7-E203-40DB-AB86-2714DBBBCF29.jpeg)