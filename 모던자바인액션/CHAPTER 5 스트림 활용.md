# CHAPTER 5 스트림 활용

발표자: 이승민
스터디일정: 2022/05/04 오전 8:30
주차별: 3주차
참여자: 권윤옥, 남채린, 이승민, 장현준

## 시작하기전에

우리는 지난 시간에 동작의 파라미터화, 람다식, 스트림api 기초에 대해 알아봤습니다.

- 스트림을 통하여 외부반복을 내부반복으로 작업 할 수 있는 방법
- 스트림은 중간 연산과 최종 연산으로 나눌 수 있다.

## 5.1 필터링

스트림의 요소를 선택하는 방법, 프레디케이트(인수로 값을 받아 true나 false를 반환하는 함수) 필터링 방법

스트림 인터페이스는 filter를 지원한다.

```java
/**
 * Returns a stream consisting of the elements of this stream that match
 * the given predicate.
 *
 * <p>This is an <a href="package-summary.html#StreamOps">intermediate
 * operation</a>.
 *
 * @param predicate a <a href="package-summary.html#NonInterference">non-interfering</a>,
 *                  <a href="package-summary.html#Statelessness">stateless</a>
 *                  predicate to apply to each element to determine if it
 *                  should be included
 * @return the new stream
 */
Stream<T> filter(Predicate<? super T> predicate);
```

```java
List<Dish> vegetarianMenu = menu.stream()
																**.filter(Dish::isVegerarian)** //채식요리인지 확인
																.collect(toList());

// 고유 요소 필터링 고유 요소로 이루어진 스트림을 반환하는 메소드도 지원한다.
List<Integer> numbers = Arrays.asList(1,2,1,3,3,2,4);
numbers.stream()
			 **.distinct()**
			 .forEach(System.out::println);
```

## 5.2 스트림 슬라이싱

슬라이싱: 스트림의 요소를 선택하거나 스킵하는 방법

```java
// java9 takeWhile , dropWhile
List<Dish> sliceMenu1 = 
							sqecialMenu.stream()
												 **.takeWhile(dish -> dish.getCalories() < 320)**
												 .collect(toList());

List<Dish> sliceMenu2 = 
							sqecialMenu.stream()
												 **.dropWhile(dish -> dish.getCalories() < 320)**
												 .collect(toList());
```

```java
spceialMenu.stream()
					 .filter(dish -> dish.getCalories() > 300)
					 **.limit(3)**
					 .collect(toList());
// limit의 결과값은 정렬되어서 가져올까요?? 윤옥님 완전 정답!!

//요소 건너뛰기
menu.stream()
		.filter(d -> d.getCalories() > 300)
	  **.skip(2)** // ****처음2개를 건너뜀
		.collect(toList());

//n개 이하의 요소를 포함하는 스트림에 skip(n)을 호출하면 어떻게 될까요? : 빈 스트림을 반환합니다.
```

## 5.3 매핑

- 개인적으로 스트림api의 핵심 기능이라고 생각되는 기능입니다.(map, flatMap)

Map: 새로운 버전을 만든다는 개념에 가까우므로 변환에 가까운 **매핑**이라는 단어를 사용한다.

```java
menu.stream()
		**.map(Dish::getName)
  //.map(String::length)**
		.collect(toList());

words.stream()
		 **.map(String::length)**
		 .collect(toList()); //어떤게 나올까요? List<Integer>가 반환됩니다.
```

- 스트림 평면화

```java
List<String> words = Arrays.asList("hello","world");

List<String> oneWords = Arrays.asList("h","e".....); //중복제거도 하고 싶었음.

words.stream()
		 **.map(word -> word.split("")) // Stream<String[]>**
		 .distinct()
		 .collect(toList()); // X List<String[]>

words.stream()
		 **.map(word -> word.split("")) // Stream<String[]>
		 .map(Arrays::stream) // 각 배열을 별도의 스트림으로 생성 Stream<String>**
		 .distinct()
		 .collect(toList());

words.stream()
		 **.map(word -> word.split("")) // Stream<String[]>
		 .flatMap(Arrays::stream) // 각 스트림을 평면화**
		 .collect(toList());
```

## 5.4 검색과 매칭

- 스트림API는 allMatch, anyMatch, nonMatch, findFirst, findAny 등등 다양한 유틸리티 메서드를 제공한다.

anyMatch, allMatch, nonMatch 는 최종연산이고 boolean을 반환한다.

또한 **쇼트서킷**을 활용한다. ← 쇼트서킷이란? 

findFirst, findAny → Optional을 리턴합니다.

- 🧐 findFirst와 findAny는 언제 사용할까요?
    
    요소의 반환 순서가 상관없다면 병렬 스트림에서는 제약이 적은 findAny를 사용합니다.
    
    논리적인 아이템 순서가 정해져있을때 findFirst로 첫번째 요소를 찾습니다.
    

## 5.5 리듀싱

- 리듀싱 연산: 모든 스트림 요소를 처리해서 값으로 도출하는 연산

TMI: 이 과정이 마치 종이를 작은 조각이 될 때까지 반복해서 접는 것과 비슷하다는 의미로 폴드라고 불립니다.

```java
int sum = 0;
for(int x: numbers){
	sum += x;
}

int sum = numbers.stream().reduce(0, (a, b) -> a + b);

//자바8에서는 정적sum메서드를 제공한다.
int sum = numbers.stream().reduce(0, Inteager::sum);

// 초기값이 없는 경우
Optional<Integer> sum = numbers.stream().reduce((a, b) -> (a + b));

//최솟값 최댓값
Optional<Integer> max = numbers.stream().reduce(Inteager::max);
Optional<Integer> min = numbers.stream().reduce(Inteager::min);
```

- 💁 reduce의 장점 병렬화! 근데 어떻게?
    
    stream() → parallelStream()
    
    그러나 람다의 상태(인스턴스 변수 같은)가 바뀌지 말아야 하며, 연산이 어떤 순서로 실행되더라도 결과가 같아야만 하는 구조여야 한다.
    

6장에서는 더 복잡한 형식의 리듀스도 살펴본다. 난 몰?루

## 5.7 숫자형 스트림

🤔  리듀스 좋긴 한데 다른 방법이 없을까? 난 그냥 합계만 궁금할뿐인데.

```java
//승민: 자바8설계자야 .sum() 같은걸로 만들어주면 좋자나
//???: 응 sum 있어.
int calories = menu.stream()
									 .map(Dish::getCalories)
									 **.sum();** // sum , (Optional)max, (Optional)min

// 숫자 스트림으로 매핑
int calories = menu.stream()
									 **.mapToInt(Dish::getCalories) // mapToDouble, mapToLong**
									 .sum();

//OptionalInt, OptionalDouble, OptionalLong
OptionalInt optMax = menu.stream()
									 .mapToInt(Dish::getCalories)
									 .max();
int max = optMax.orElse(1);
```

그외에 숫자범위(rangeClosed)  객체스트림복원 boxed 등등이 있지만...

## 5.8 스트림 만들기

stream 메서드로 컬렉션에서 스트림을 얻을 수 있다.

```java
Stream<String> stream = Stream.of("Modern", "Java", "In", "Action");
Stream<String> emptyStream = Steam.empty();
Stream<String> nullableStream = Steam.ofNullable(
																				null, "Java");

// 배열로 스트림 만들기
int[] numbers = {2, 3, 5, 7, 11, 13};
int sum = **Arrays.stream(numbers)**.sum();
```

파일 스트림.. 함수로 무한 스트림 만들기 등등이 있지만..