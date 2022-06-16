# CHAPTER 6 스트림으로 데이터 수집

발표자: 장현준
스터디일정: 2022/05/12 오후 6:30
주차별: 4주차
참여자: 권윤옥, 남채린, 이승민, 장현준

<aside>
💡 1. Collectors 클래스로 컬렉션을 만들고 사용하기
2. 하나의 값으로 데이터 스트림 리듀스하기
3. 특별한 리듀싱 요약 연산
4. 데이터 그룹화의 분할 
5. 자신만의 커스텀 컬렉터 개발

</aside>

# 들어가기

1. 스트림 연산은 중간연산(filter, map)과 최종연산(count, findFirst, forEach, reduce)으로 구분할 수 있다.
중간연산은 한 스트림을 다른 스트림으로 변환하는 연산으로서, 여러 연산을 연결 가능하다. 또한 스트림 파이프라인을 구성하며, 스트림의 요소를 **소비** 하지 않는다. 하지만 최종연산은 스트림 파이프라인을 최적화하면서 계산 과정을 짧게 생략하기도 한다.
2. 이번 장에서는 reduce가 그랬던 것처럼 collect 역시 다양한 요소 누적 방식을 인수로 받아서 스트림을 최종 결과로 도출하는 리듀싱 연산을 수행할 수 있음을 설명한다.
3. 예시
    - 통화별로 트랜잭션을 그룹화한 다음에 해당 통화로 일어난 모든 트랜잭션 합계를 계산하시오(Map<Currency, Integer> 반환).
    - 트랜잭션을 비싼 트랜잭션과 저렴한 트랜잭션 두 그룹으로 분류하여라. (Map(Boolean, List<Transaction>>).
    - 트랜잭션을 도시 등 다수준으로 그룹화하여라. 그리고 각 트랜잭션이 비싼지 저렴한지 구분하여라. (Map<String, Map<Boolean, List<Transaction>>>)
    

**통화별로 트랜잭션을 그룹화한 코드**

```java
Map<Currency, List<Transaction>> transactionsByCurrencies = new HashMap<>(); // 그룹화한 트랜잭션을 저장할 맵을 생성한다.

for (Transaction transaction : transactions) {
	Currency currency = transaction.getCurrency(); // 트랜잭션의 통화를 추출한다.
	List<Transaction> transactionsForCurrency = transactionsByCurrencies.get(currency);
	if (transactionForCurrency == null) { // 만약 통화를 그룹화하는 맵에 항목이 없으면 항목을 만든다.
		transactionForCurrency = new ArrayList<>();
		transactionsByCurrencies.put(currency, transactionsForCurrency);
	}
	transactionsForCurrency.add(transactions); // 같은 통화를 가진 트랜잭션 리스트에 현재 탐색중인 트랜잭션을 추가한다.
}
```

```java
Map<Currency, List<Transaction>> transactionsByCurrencies = transactions
																																.stream()
																																.collect(groupingBy(Transaction::getCurrency));
```

→ `Transaction::getCurrency` 에서 값이 null이면 빈배열 반환하는지 체크하기

# 6.1 컬렉터란 무엇인가?

- 함수형 프로그래밍에서는 ‘무엇’을 원하는지 직접 명시할 수 있어서 어떤 방법으로 이를 얻을지는 신경 쓸 필요가 없다.( → 메서드를 넘긴다는 뜻인가...?) 이전 예제에서 collect 메서드로 Collector 인터페이스 구현을 전달했다.
- Collector 인터페이스 구현은 스트림의 요소를 어떤 식으로 도출할지 지정한다.

→ 이 부분의 대제목처럼 무엇인가?의 해답이 두드러지게 나오지 않음...☹️

## 6.1.1 고급 리듀싱 기능을 수행하는 컬렉터

- 스트림에 collect를 호출하면 스트림 요소에(컬렉터로 파라미터화된) 리듀싱 연산이 수행된다. 아래 그림은 내부적으로 **리듀싱 연산**이 일어나는 그림이다. collect에서는 리듀싱 연산을 이용해서 스트림의 각 요소를 방문하면서 컬렉터가 작업을 처리한다.

![스크린샷 2022-05-09 01.43.42.png](CHAPTER%206%20%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%89%E1%85%AE%E1%84%8C%E1%85%B5%E1%86%B8%20e54a11fbb30d40338331ed7c9fcc18fd/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-05-09_01.43.42.png)

## 6.1.2 미리 정의된 컬렉터

- 6장에서는 미리 정의된 컬렉터, 즉 groupingBy 같이 Collectors 클래스에서 제공하는 팩토리 메서드의 기능을 설명한다. Collectors에서 제공하는 메서드의 기능은 크게 3가지로 구분할 수 있다.
    - 스트림 요소를 하나의 값으로 리듀스하고 요약
    → 총합을 찾는 등의 다양한 계산을 수행할 때 유용하다.
    - 요소 그룹화
    → 다수준으로 그룹화하거나 각각의 결과 서브그룹에 추가로 리듀싱 연산을 적용할 수 있도록 다양한 컬렉터를 조합하는 방법이다.
    - 요소 분할
    → 한 개의 인수를 받아 불리언을 반환하는 함수, 즉 프레디케이트를 그룹화 함수로 사용한다.

```java
static <T,K> Collector<T,?,Map<K,List<T>>>	groupingBy(Function<? super T,? extends K> classifier)
→ Returns a Collector implementing a "group by" operation on input elements of type T, grouping elements according to a classification function, and returning the results in a Map.

static <T,K,A,D> Collector<T,?,Map<K,D>>	groupingBy(Function<? super T,? extends K> classifier, Collector<? super T,A,D> downstream)
→ Returns a Collector implementing a cascaded "group by" operation on input elements of type T, grouping elements according to a classification function, and then performing a reduction operation on the values associated with a given key using the specified downstream Collector.

static <T,K,D,A,M extends Map<K,D>>
→ Collector<T,?,M>	groupingBy(Function<? super T,? extends K> classifier, Supplier<M> mapFactory, Collector<? super T,A,D> downstream)
Returns a Collector implementing a cascaded "group by" operation on input elements of type T, grouping elements according to a classification function, and then performing a reduction operation on the values associated with a given key using the specified downstream Collector.
```

[java 8 Doc](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html)

# 6.2 리듀싱과 요약

1. 컬렉터로 스트림 항목을 컬렉션으로 재구성할 수 있다.

```java
long howManyDishes = menu.stream().collect(Collectors.counting()); 
long howManyDishes = menu.stream().count();
```

# 6.2.1 스트림값에서 최댓값과 최솟값 검색

- 다음은 칼로리로 요리를 비교하는 Comparator를 구현한 다음에 Collectors.maxBy로 전달하는 코드다.

```java
public class Dish {

  private final String name;
  private final boolean vegetarian;
  private final int calories;
  private final Type type;

  public Dish(String name, boolean vegetarian, int calories, Type type) {
    this.name = name;
    this.vegetarian = vegetarian;
    this.calories = calories;
    this.type = type;
  }

  public String getName() {
    return name;
  }

  public boolean isVegetarian() {
    return vegetarian;
  }

  public int getCalories() {
    return calories;
  }

  public Type getType() {
    return type;
  }

  @Override
  public String toString() {
    return name;
  }

  public enum Type {
    MEAT,
    FISH,
    OTHER
  }

  public static final List<Dish> menu = asList(
      new Dish("pork", false, 800, Dish.Type.MEAT),
      new Dish("beef", false, 700, Dish.Type.MEAT),
      new Dish("chicken", false, 400, Dish.Type.MEAT),
      new Dish("french fries", true, 530, Dish.Type.OTHER),
      new Dish("rice", true, 350, Dish.Type.OTHER),
      new Dish("season fruit", true, 120, Dish.Type.OTHER),
      new Dish("pizza", true, 550, Dish.Type.OTHER),
      new Dish("prawns", false, 400, Dish.Type.FISH),
      new Dish("salmon", false, 450, Dish.Type.FISH)
  );

  public static final Map<String, List<String>> dishTags = new HashMap<>();
  static {
    dishTags.put("pork", asList("greasy", "salty"));
    dishTags.put("beef", asList("salty", "roasted"));
    dishTags.put("chicken", asList("fried", "crisp"));
    dishTags.put("french fries", asList("greasy", "fried"));
    dishTags.put("rice", asList("light", "natural"));
    dishTags.put("season fruit", asList("fresh", "natural"));
    dishTags.put("pizza", asList("tasty", "salty"));
    dishTags.put("prawns", asList("tasty", "roasted"));
    dishTags.put("salmon", asList("delicious", "fresh"));
  }

}

List<Dish> menu = Dish.menu;
Comparator<Dish> dishComparator = Comparator.comparingInt(Dish::getCalories);
System.out.println(menu.stream().collect(maxBy(dishComparator)).get());

결과 : pork
```

→ 추가적으로 menu가 비어있다면 그 어떤 요리도 반환되지 않을것이다. 

- 위와 같이 객체의 숫자 필드의 합계 or 평균등을 반환하는 연산에도 리듀싱 기능이 사용된다. 이러한 연산을 **요약**이라 부른다.

# 6.2.2 요약 연산

1. summingInt 팩토리 메서드

```java
public static <T> Collector<T,?,Integer> summingInt(ToIntFunction<? super T> mapper)
Returns a Collector that produces the sum of a integer-valued function applied to the input elements. If no elements are present, the result is 0.
```

- 객체를 int로 매핑하는 함수를 인수로 받고있다. 또한 객체를 int로 매핑한 컬렉터를 반환한다.

### Example

`int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));`

![Untitled](CHAPTER%206%20%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%89%E1%85%AE%E1%84%8C%E1%85%B5%E1%86%B8%20e54a11fbb30d40338331ed7c9fcc18fd/Untitled.png)

[사진 출처](https://cornswrold.tistory.com/546)

```java
Integer collect = Dish.menu.stream().collect(summingInt(Dish::getCalories));
                                 ⏬
Integer collect = Dish.menu.stream().mapToInt(Dish::getCalories).sum(); // 인텔리제이 추천
System.out.println(collect);

결과 : 4300
```

1. Collectors.summingLong, Collectors.summingDouble 메서드는 같은 방식으로 동작하고 각각 long 또는 double 형식의 데이터로 요약한다는 점만 다름.
2. 평균값 계산도 가능함 (averagingInt, averagingLong, averagingDouble)
`example → menu.stream().collect(averagingInt(Dish::getCalories));`
3. 만약 두 개 이상의 연산을 한 번에 수행해야 할 때도 있다. 이때는 어떻게 해야할까?
`summarizingInt` 를 사용할 수 있다.

```java
IntSummaryStatistics collect = Dish.menu.stream().collect(summarizingInt(Dish::getCalories));
System.out.println(collect);

결과:
IntSummaryStatistics{count=9, sum=4300, min=120, average=477.777778, max=800}
```

# 6.2.3 문자열 연결

- 컬렉터에 joining 팩토리 메서드를 이용하면 스트림의 각 객체에 toString 메서드를 호출해서 추출한 모든 문자열을 하나의 문자열로 연결해서 반환한다.

```java
return carTripPathRepository.findAllPdopByCarIdAndOccurrenceDateTimeAndTripId(carId, BEFORE_RT_CREATION_DATE, WHEN_RT_CREATION_DATE, tripRealtimePacket.getTripId())
                                       .stream()
                                       .map(String::valueOf)
                                       .collect(Collectors.joining(", ","[","]"));

결과 : pdop	[14, 14, 14, 14, 16, 16, 15, 20, 19, 13, 12, 16]
```

# 6.2.4 범용 리듀싱 요약 연산

- 여태까지 본 모든 컬렉터는 reducing 팩토리 메서드로 정의할 수 있다. (범용 팩토리 메서드 대신 특화된 컬렉터를 사용하는 이유는 프로그래밍적 편의성 때문이다.)

```java
// Collectors.reducing 범용 팩토리 메서드 사용 
int totalCalories = menu.stream().collect(reducing(0, Dish::getCalories, (i, j) -> (i + j)); 
int totalCalories = menu.stream().collect(reducing(0, Dish::getCalories, Integer::sum); 

// 특화된 컬렉터 사용 
int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));

```

- 다음과 같이 인수를 한 개로 받을 수 도 있다.

```java
Optional<Dish> mostCaloriDish = menu.stream().collect(reducing(d1, d2) -> d1.getCalories() > d2.getCalories() ? d1 : d2));
```

→ 인수를 한 개만 받는 reducing은 시작값이 없으므로 빈 스트림이 넘겨졌을 때 시작값이 설정되지 않는 상황이 발생한다. 그래서 반환값이 `Optional`로 설정되어있다.

# Collect VS Reduce

```java
Stream 인터페이스의 collect와 reduce 메서드는 차이점이 뭐야..?
collect(toList()) 대신 reduce 를 사용하여 리스트를 반환할 수 있다.
// 1. reduce 
// 병렬 처리 시 각자 다른 쓰레드에서 실행한 결과를 마지막에 합치는 단계 
// 병렬 스트림에서만 동작 
List<Integer> numbers = stream.reduce( 
		new ArrayList<Integer>(), 
			(List<Integer> l, Integer e) -> { 
				l.add(e); 
				return l; 
			}, 
			(List<Integer> l1, List<Integer> l2) -> { 
				l1.addAll(l2); 
				return l1; 
			} 
	); 
// 2. 
collect List<Integer> numbers = stream.collect(Collectors.toList());
```

collect : 도출하려는 결과를 `누적하는 컨테이너`를 바꾸도록 설계가 되어있는 메서드

reduce : 두 값을 하나로 도출하는 `불변형 연산` 하는 메서드

### 결론

- 자신의 상황에 맞게 사용하자 😀

## 컬렉션 프레임워크 유연성 : 같은 연산도 다양한 방식으로 수행가능

- reducing 컬렉터를 사용한 이전 예제에서 람다 표현식 대신 Integer 클래스의 sum 메서드 참조를 이용하면 코드를 좀 더 단순화 가능하다.

```java
int totalCalories = menu.stream().collect(reducing(0, → 초기값
																			Dish::getCalories, → 변환함수
																			Integer::sum)); → 합계함수
```

![Untitled](CHAPTER%206%20%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%89%E1%85%AE%E1%84%8C%E1%85%B5%E1%86%B8%20e54a11fbb30d40338331ed7c9fcc18fd/Untitled%201.png)

```java
public static <T> Collector<T, ?, Long> counting() {
	return reducing(0L, e -> 1L, Long::sum);
}
```

- Collecotr에서의 정의

### 자신의 상황에 맞는 최적의 해법 선택

- 문제를 해결할 수 있느 다양한 해결 방법을 확인한 다음에 가장 일반적으로 문제에 특화된 해결책을 고르는 것이 바람직하다.

### Quiz

- 아래 joining 컬렉터를 reducing 컬렉터로 올바르게 바꾼 코드를 모두 선택하면 된다.

`String shorMenu = menu.stream().map(Dish::getName).collect(joining());`

1. String shortMenu = menu.stream().map(Dish::getName).collect(reducing( (s1, s2) → s1 + s2) ).get();
2. String shortMenu = menu.stream().collect(reducing( (d1, d2) → d1.getName() + d2.getName())).get();
3. String shortMenu = menu.stream().collect(reducing( “”, Dish::getName, (s1, s2) → s1 + s2));

# 6.3 그룹화

- 데이터 집합을 하나 이상의 특성으로 분류해서 그룹화하는 연산도 데이터베이스에서 많이 수행되는 작업.

```java
Map<Dish.Type, List<Dish>> dishesByType = menu.stream().collect(groupingBy(Dish::getType));
```

### 결과

```java
{FISH =[Prawns, salmon] , OTHER = [french fries, rice, season fruit], MEAT = [pork, breef]}
```

- 위 코드에서 groupingBy 메서드 기준으로 스트림이 그룹화되므로 이를 `분류함수` 라고 함.

![Untitled](CHAPTER%206%20%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%89%E1%85%AE%E1%84%8C%E1%85%B5%E1%86%B8%20e54a11fbb30d40338331ed7c9fcc18fd/Untitled%202.png)

### 조금 더 복잡한 로직으로는 그룹화를 할 수 없는가?

- 단순한 속성 접근자 대신 더 복잡한 분류 기준이 필요한 상황에서는 메서드 참조를 분류 함수로 사용 x
예를 들어 400칼로리 이하를 `diet` 로 400 ~ 700 칼로리를 `normal` 등등... 하지만 이러한 연산에 필요한 메서드가 없기 때문에 `분류함수` 를 사용할 수 없다. 그래서 밑에 코드와 같이 람다 표션식으로 필요한 로직을 구현 할 수 있다.

```java
public enum CaloricLevel { DIET, NORMAL, FAT }

Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream().collect( groupingBy dish -> {
																																		if (dish.getCalories() <= 400 ) return CaloricLevel.DIET;
																																	  else if ...
```

# 6.3.1 그룹화된 요소 조작

- 예를들어 500 칼로리가 넘는 요리만 필터 한다고 가정하자.

### 첫번 째 방법

- 그룹화를 하기 전에 프레디케이트로 필터를 적용해 문제 해결

```java
Map<Dish, List<Dish>> caloricDishesByType = menu.stream().filter(dish -> dish.getCalories() > 500).collect(groupingBy(Dish::getType));
```

### 결과

```java
{OTHER = [french fries, pizza], MEAT = [pork, beef]}
```

- 무엇이 문제인가?
- 해답
    
    우리의 필터 프레디케이트를 만족하는 FISH 종류 요리는 없으므로 결과 맵에서 해당 키 자체가 사라짐. 그래서 `Collector 형식의 두 번째 인수를 갖도록 groupingBy 팩터리 메서드를 오버로드해 이 문제를 해결하면 된다.` 
    
    ```java
    Map<Dish.Type, List<Dish>> caloricDishesByType = menu.stream()
                                                         .collect(groupingBy(Dish::getType, filtering(dish -> dish.getCaloies() > 500, toList()));
    ```
    
    ### 결과
    
    ```java
    {OTHER = [french fries, pizza], MEAT = [pork, beef], FISH = []}
    ```
    

### 두번 째 방법

- filtering 컬렉터와 같은 이유로 Collectors 클래스는 매핑 함수와 각 항목에 적용한 함수를 모으는 데 사용하는 또 다른 컬렉터를 인수로 받는 mapping 메서드를 제공한다.

```java
Map<Dish.Type, List<String>> dishNameByType = menu.stream()
																								  .collect(groupingBy(Dish::getType, mapping(Dish::getName, toList()));

```

### 예제

```java
Map<String, List<String>> dishTags = new HashMap<>();
dishTags.put("pork", asList("greasy", "salty"));
dishTags.put("chicken", asList("greasy", "salty"));
dishTags.put("beef", asList("greasy", "salty"));
dishTags.put("salmon", asList("greasy", "salty"));

// flatMapping 이용
Map<Dish.Type, Set<String>> dishNamesByType = 
															menu.strema()
														      .collect(groupingBy(Dish::getType, flatMapping(dish -> dishTags.get( dish.getName() ).stream(), toSet()))));
```

### 결과

```java
{MEAT=[salty, greasy, fried], FISH=[tasty, fresh, delicious], OTHER=[salty,greasy]}
```

# 6.3.2 다수준 그룹화

- Collectors.groupingBy를 이용해서 항목을 다수준으로 그룹화 할 수 있다. Collectors.groupingBy는 일반적인 분류 함수와 컬렉터를 인수로 받는다. 즉, 바깥쪽 groupingBy 메서드에 스트림의 항목을 분류할 두 번째 기준을 정의하는 내부 groupingBy를 전달해서 두 수준으로 스트림의 항목을 그룹화 가능

```java
Map<Dish.Type, Map<CaloricLevel, List<Dish>>> dishesByTypeCaloricLevel = menu.stream()
																																						 .collect(
																																						 groupingBy(Dish::getType, // 첫 번쨰 수준의 분류 함수
																																						  groupingBy(dish -> {     // 두번째 수준의 분류 함수
																																									 if(dish.getCalories() <= 400)
																																										 return CaloricLevel.DIET;
																																									 else if (dish.getCalories() <= 700)
																																									   return CaloricLevel.NORMAL;
																																									 else
																																										 return CaloricLevel.FAT;
																																						 })
																																					)
};	
```

### 결과

{

MEAT={DIET=[chicken], NORMAL=[beef], FAT=[pork]},

 FISH={DIET=[prawns], NORMAL=[salmon]}, 

OTHER={DIET=[rice, seasonal fruit], NORMAL=[french fries, pizza]}

}

![Untitled](CHAPTER%206%20%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%89%E1%85%AE%E1%84%8C%E1%85%B5%E1%86%B8%20e54a11fbb30d40338331ed7c9fcc18fd/Untitled%203.png)

- 보통 groupingBy의 연산을 **버킷** 개념으로 생각하면 쉽다. 첫 번째 groupingBy는 각 키의 버킷을 만든다. 그리고 준비된 각각의 버킷을 서브스트림 컬렉터로 채워가기를 반복하면서 n수준 그룹화를 달성한다.

# 6.3.3 서브그룹으로 데이터  수집

- 다음 쿼리는 무엇에 대한 기준으로 작성한 퀴리인가??

```java
Map<Dish.Type, Optional<Dish>> mostCaloricByType = 
	menu.stream()
			.collect(groupingBy(Dish::getType, 
													maxBy(comparingInt(Dish::getCalories))));
```

- 결과
    
    ```java
    {
    	FISH=Optional[salmon],
    	OTHER=Optional[pizza],
    	MEAT=Optional[pork]
    }
    ```
    

### 컬렉터 결과를 다른 형식에 적용하기

- 팩토리 메서드 Collectors.collectingAndThen으로 컬렉터가 반환한 결과를 다른 형식으로 활용할 수 있다.

```java
Map<Dish.Type, Dish> mostCaloricByType = 
		menu.stream()
				.collect(groupingBy(Dish::getType, // 분류 함수
								 collectingAndThen(
										maxBy(comparingInt(Dish::getCalories)), //감싸인 컬렉터
								 Optional::get))); //변환 함수
```

### 결과

```java
{
	FISH=salmon,
	OTHER=pizza,
	MEAT=pork
}
```

### 해설

- 팩토리 메서드 collectingAndThen은 적용할 컬렉터와 변환 함수를 인수로 받아 다른 컬렉터를 반환한다.
반환되는 컬렉터는 기존 컬렉터의 래퍼 역할을 하며 collect의 마지막 과정에서 변환 함수로 자신이 반환하는 값을 매핑한다.
- 이 예제에서는 maxBy로 만들어진 컬렉터가 감싸지는 컬렉터며 변환 함수 Optional::get으로 반환된 Optional에 포함된 값을 추출!!!

`리듀싱 컬렉터는 절대 Optional.empty()를 반환x`

![Untitled](CHAPTER%206%20%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%89%E1%85%AE%E1%84%8C%E1%85%B5%E1%86%B8%20e54a11fbb30d40338331ed7c9fcc18fd/Untitled%204.png)

- 컬렉터는 점선으로 표시.
- groupingBy는 가장 바깥쪽에 위치하면서 요리의 종류에 따라 메뉴 스트림을 세 개의 서브스트림으로 그룹화
- groupingBy 컬렉터는 collectingAndThen 컬렉터를 감싼다.
    - 두 번째 컬렉터는 그룹화 된 세 개의 서브스트림에 적용된다.
- collectingAndThen 컬렉터는 세번째 컬렉터 maxBy를 감싼다.
- 리듀싱 컬렉터가 서브스트림에 연산을 수행한 결과에 collectingAndThen의 Optional::get 변환 함수가 적용된다.
- groupingBy 컬렉터가 반환하는 맵의 분류 키에 대응하는 세 값이 각각의 요리 형식에서 가장 높은 칼로리임.

# 6.4 분할

- 분할은 **분할 함수** 라 불리는 프레디케이트를 분류 함수로 사용하는 특수한 그룹화 기능이다. 분할 함수는 불리언을 반환하므로 맵의 키 형식은 Boolean이다. 결과적으로 그룹화 맵은 최대 ( 참 or 거짓) 두 개의 그룹으로 분류된다.

### 예를 들어보자

- 채식주의자 친구를 초대했다고 하면 모든 요리는 채식요리와 채식이 아닌 요리로 구분해야함.

```java
Map<Boolean, List<Dish>> partitionedMenu =
							menu.strem().collect(paritioningBy(Dish::isVegetarian));

or

List<Dish> vegetDishes = 
		menu.stream().filter(Dish::isVegetarian).collect(toList());
```

# 6.4.1 분할의 장점

- 분할 함수가 반환하는 참, 거짓 두 가지 요소의 스트림 리스트를 모두 유지한다는 것이 분할의 장점이다.

```java
Map<Boolean, Map<Dish.Type, List<Dish>>> vegetarianDishesByType 
				= menu.stream()
						  .collect(partitioningBy(Dish::isVegetarian, // 분할 함수
																			groupingBy(Dish::getType))); // 두 번째 컬렉터
```

### 결과

```java
{
	false={FISH=[prawns, salmon], MEAT=[pork, beef, chicken]},
	true={OTHER=[french fries, rice, season fruit]}
}
```

### 채식요리와 아닌요리에서 칼로리가 높은 요리 구하기

```java
Map<Boolean, Dish> mostCaloricPartitionedByVegetarian
		= menu.stream().collect(
					paritioningBy(Dish::isVegetarian,
								collectingAndThen(maxBy(comparingInt(Dish::getCalories)), Optional::get)));
```

### 결과

```java
{false=pork, true=pizza}
```