# CHAPTER 8 컬렉션 API 개선

발표자: 남채린
스터디일정: 2022/05/19 오후 6:00
주차별: 5주차
참여자: 권윤옥, 남채린, 이수환, 이승민, 장현준

## 컬렉션 팩토리

자바8, 자바9에 추가된 새로운 컬렉션 API

### 리스트 팩토리 - List.of

```java
List<String> friends = List.of("Raphael", "Olivia", "Thibaut");
System.out.println(friends); //[Raphael, Olivia, Thibaut]
```

```java
List<String> friends = List.of("Raphael", "Olivia", "Thibaut");
friends.add("Chih-Chun");
```

변경할 수 없는 리스트가 만들어 졌기 때문! → 리스트를 바꿔야 하는 상황이라면 직접 리스트를 만들어야 한다.

### 집합 팩토리 - Set.of

```java
Set<String> friends = Set.of("Raphael", "Olivia", "Thibaut");
System.out.println(friends); //[Raphael, Olivia, Thibaut]
```

```java
Set<String> friends = Set.of("Raphael", "Olivia", "Olivia");
```

### 맵 팩토리 - Map.of

```java
Map<String, Integer> ageOfFriends = Map.of("Raphael", 30, "Olivia", 25, "Thibaut", 26);
System.out.println(friends); //{Olivia=25, Raphael=30, Thibaut=26}
```

열 개 이하의 키와 값 쌍을 가진 작은 맵을 만들때 유용

### 맵 팩토리 - Map.ofEntries

```java
import static java.util.Map.entry;
Map<String, Integer> ageOfFriends 
		= Map.ofEntries(entry("Raphael", 30), 
										entry("Olivia", 25), 
										entry("Thibaut", 26));
System.out.println(friends); //{Olivia=25, Raphael=30, Thibaut=26}
```

이 메서드는 키와 값을 감쌀 추가 객체 할당을 필요로 한다.

## 리스트 및 집합과 사용할 새로운 관용 패턴

### removeIf 메서드

```java
for (Transaction transaction : transactions) {
		if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
				transactions.remove(transaction);
		}
}
```

```java
for (Iterator<Transaction> iterator = transactions.iterator();
		 iterator.hasNext(); ) {
		 Transaction transaction = iterator.next();
		 if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
				transactions.remove(transaction);
		 }
}
```

```java
for (Iterator<Transaction> iterator = transactions.iterator();
		 iterator.hasNext(); ) {
		 Transaction transaction = iterator.next();
		 if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
				iterator.remove();
		 }
}
```

```java
transactions.removeIf(transaction ->
		Character.isDigit(transaction.getReferenceCode().charAt(0)));
```

### replaceAll 메서드

```java
referenceCodes.stream() //[a12, C14, b13]
							.map(code -> Character.toUpperCase(code.charAt(0)) + code.substring(1))
							.collect(Collectors.toList())
							.forEach(System.out::println); //A12, C14, B13
```

```java
for (ListIterator<String> iterator = referenceCodes.listIterator(); 
		iterator.hasNext(); ) {
		String code = iterator.next();
		iterator.set(Character.toUpperCase(code.charAt(0)) + code.substring(1));
}
```

```java
referenceCodes.replaceAll(code -> Charater.toUpperCase(code.charAt(0)) + code.subString(1));
```

## 맵과 사용할 새로운 관용 패턴

### forEach 메서드

```java
for(Map.Entry<String, Integer> entry: ageOfFriends.entrySet()) {
		String friend = entry.getKey();
		Integer age = entry.getValue();
		System.out.println(friend + " is " + age + " years old");
}
```

Map 인터페이스는 BiConsumer(키와 값을 인수로 받음)를 인수로 받는 **forEach** 메서드를 지원하므로 아래와 같이 코드를 간단하게 구현할 수 있다.

```java
ageOfFriends.forEach((friend, age) -> System.out.println(friend + " is " + 
		age + " years old"));
```

### 정렬 메서드

정렬은 반복과 관련한 오래된 고민거리임

아래의 새로운 유틸리티를 이용하면 맵의 항목을 값 또는 키를 기준으로 정렬할 수 있다.

- Entry.comparingByValue
- Entry.comparingByKey

```java
Map<String, String> favouriteMovies
				= Map.ofEntries(entry("Raphael", "Star Wars"),
												entry("Cristina", "Matrix"),
												entry("Olivia", "James Bond"));

favouriteMovies
		.entrySet()
		.stream()
		.sorted(Entry.comparingByKey())
		.forEachOrdered(System.out.println);

/*
Cristina=Matrix
Olivia=James Bond
Raphael=Star Wars
*/
```

### getOrDefault 메서드

- 요청한 키가 맵에 존재하지 않을 때 처리하는 방법
- 첫 번째 인수 : 키, 두 번째 인수 : 기본값
- 맵에 키가 존재하지 않으면 두 번째 인수로 받은 기본값을 반환한다.

```java
Map<String, String> favouriteMovies
		= Map.ofEntries(entry("Raphael", "Star Wars"),
										entry("Olivia", "James Bond"));

System.out.println(favouriteMovies.getOrDefault("Olivia", "Matrix"));
//James Bond

System.out.println(favouriteMovies.getOrDefault("Thibaut", "Matrix"));
//Matrix
```

### 계산 패턴

맵에 키가 존재하는지 여부에 따라서 어떤 동작을 실행하고 결과를 저장해야 할지

- computeIfAbsent : 제공된 키에 해당하는 값이 없으면(값이 없거나 널), 키를 이용해 새 값을 계산하고 맵에 추가한다.
- computeIfPresent : 제공된 키가 존재하면 새 값을 계산하고 맵에 추가한다.
- compute : 제공된 키로 새 값을 계산하고 맵에 저장한다.

```java
Map<String, byte[]> dataToHash = new HashMap<>();
MessageDigest messageDigest = MessageDigest.getInstance("SHA-256");

lines.forEach(line ->
		dataToHash.computeIfAbsent(line,
															 this::calculateDigest));

private byte[] calculateDigest(String key) {
		return messageDigest.digest(key.getBytes(StandardCharsets.UTF_8));
}
```

```java
String friend = "Raphael";
List<String> movies = friendsToMovies.get(friend);
if(movies == null) {
		movies = new ArrayList<>();
		friendsToMovies.put(friend, movies);
}
movies.add("Star Wars");

System.out.println(friendsToMovies); //{Raphael:[Star Wars]}
```

```java
friendsToMovies.computeIfAbsent("Raphael", name -> new ArrayList<>()).add("Star Wars")'
```

### 삭제 패턴

제공된 키에 해당하는 맵 항목을 제거하는 remove 메서드 

→ (자바8) 키가 특정한 값과 연관되었을 때만 항목을 제거하는 오버로드 버전 메서드를 제공

```java
String key = "Raphael";
String value = "Jack Reacher 2";

// 기존 코드
if (favouriteMovies.containsKey(key) 
			&& Objects.equals(favouriteMovies.get(key), value)) 
{
		favouriteMovies.remove(key);
		return true;
}
else {
		return false;
}

// 자바8 오버로드 버전 메서드 사용
favouriteMovies.remove(key, value);
```

### 교체 패턴

- replaceAll : BiFunction을 적용한 결과로 각 항목의 값을 교체한다. List의 replaceAll과 비슷한 동작
- Replace : 키가 존재하면 맵의 값을 바꾼다. 키가 특정 값으로 매핑되었을 때만 값을 교체하는 오버로드 버전도 있다.

```java
Map<String, String> favouriteMovies = new HashMap<>();
favouriteMovies.put("Raphael", "Star Wars");
favouriteMovies.put("Olivia", "james bond");
favouriteMovies.replaceAll((friend, movie) -> movie.toUpperCase());
System.out.println(favouriteMovies);

/*
{Olivia=JAMES BOND, Raphael=STAR WARS}
*/
```

### 합침

두 개의 맵에서 값을 합치거나 바꿔야 한다면 어떻게 해야할까? ⇒ merge 메서드 사용!

```java
Map<String, String> family = Map.ofEntries(
		entry("Teo", "Star Wars"), entry("Christina", "James Bond"));
Map<String, String> friends = Map.ofEntries(
		entry("Raphael", "Star Wars"));
Map<String, String> everyone = new HashMap<>(family);
everyone.putAll(friends);
System.out.println(everyone);

/*
{Christina=James Bond, Raphael=Star Wars, Teo=Star Wars}
*/
```

중복된 키가 없다면 위 코드는 잘 동작한다.

merge 메서드는 중복된 키를 어떻게 합칠지 결정하는 BiFunction을 인수로 받는다.

```java
Map<String, String> family = Map.ofEntries(
		entry("Teo", "Star Wars"), entry("Christina", "James Bond"));
Map<String, String> friends = Map.ofEntries(
		entry("Raphael", "Star Wars"), entry("Christina", "Matrix"));
```

```java
Map<String, String> everyone = new HashMap<>(family);
friends.forEach((k, v) ->
		everyone.merge(k, v, (movie1, movie2) -> movie1 + " & " + movie2));
System.out.println(everyone);

/*
{Raphael=Star Wars, Christina=James Bond & Matrix, Teo=Star Wars}
*/
```

<aside>
✅ ****BiFunction<T,U,R>****

Function<T, R> 과 동일하지만 U라는 타입을 하나 더 받습니다. 그래서 T, U 타입을 받아 R을 반환하는 함수 인터페이스 입니다.

```java
R apply(T t, U u)
```

</aside>

▶︎ merge 메서드는 널값과 관련된 복잡한 상황도 처리한다.

> 지정된 키와 연관된 값이 없거나 값이 널이면 [merge]는 키를 널이 아닌 값과 연결한다. 아니면 [merge]는 연결된 값을 주어진 매핑 함수의 [결과] 값으로 대치하거나 결과가 널이면 [항목]을 제거한다.
> 

```java
Map<String, Long> moviesToCount = new HashMap<>();
String movieName = "JamesBond";
long count = moviesToCount.get(movieName);
if(count == null){
		moviesToCount.put(movieName, 1);
}
else {
		moviesToCount.put(movieName, count + 1);
}
```

```java
moviesToCount.merge(movieName, 1L, (key, count) -> count + 1L);
```

merge의 두 번째 인수는 키와 연관된 기존 값에 합쳐질 널이 아닌 값 또는 값이 없거나 키에 널 값이 연관되어 있다면 이 값을 키와 연결하는데에 사용된다.