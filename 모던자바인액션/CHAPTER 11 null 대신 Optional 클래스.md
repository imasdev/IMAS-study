# CHAPTER 11 null 대신 Optional 클래스

발표자: 이승민
스터디일정: 2022/05/26 오후 6:00
주차별: 6주차
참여자: 권윤옥, 남채린, 이수환, 이승민, 장현준

자바 개발자로 NullPointerException을 안 겪은 사람이 있을까.. 👹

## 값이 없는 상황을 어떻게 처리할까?

```java
if(car != null){
	if(car.getCompany() != null){
		if(car.getCompany().getBranch() != null){
			// doSomeThing();
			//... 벌써 어지럽다 그죠?
		}
	}
}
```

- 에러의 근원이다
- 코드를 어지럽힌다.
- 아무 의미가 없다.
- 자바 철학에 위배된다.(자바는 모든 주소값 즉 포인터를 숨겼지만 유일하게 살아남은 좀비 바로 null이다.)
- 그냥 null 보면 화가나!

## Optional 클래스 소개

null 때문에 발생하는 다양한 문제가 있습니다…
그래서 자바8에서는 Optional이 등장!

![값를 Optional로 감싼다. 화려한 Optional이 나를 감싸네~](CHAPTER%2011%20null%20%E1%84%83%E1%85%A2%E1%84%89%E1%85%B5%E1%86%AB%20Optional%20%E1%84%8F%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A2%E1%84%89%E1%85%B3%20f07cdc70c1d145dca0a54015a1dcf8fd/Untitled.png)

값를 Optional로 감싼다. 화려한 Optional이 나를 감싸네~

😎 실제로 어떻게 사용하는지 알아보자!

## Optional 완벽 정복

```java
/* Optional 객체 만들기 */
Optional<Car> optCar = Optional.empty();

// 정적 팩토리 메서드로 Optional 만들기1
Optional<Car> optCar = Optional.of(car);

// 정적 팩토리 메서드로 Optional 만들기2
Optional<Car> optCar = Optional.ofNullalbe(car);

//두개는 뭐가 다를까요?

//또 jpa를 이용하면 Optional로 객체를 받아오죠.
```

Optional에서 어떻게 값을 가져올까요? → get을 이용해서 가져옵니다. 그러면 null이랑 뭐가 다른거야? Optional에서 제공하는 다양한 기능을 알게 되면 어떤게 다른지 알게 됩니다.

---

```java
Optional<Car> optCar = Optional.ofNullable(car);
Optional<String> optCarNumber = optCar.map(Car::getOptCarNumber);

// Optional이 비어 있으면 아무일도 일어나지 않는다.
Optional<Person> optPerson = Optional.ofNullable(person);
```

```java
public class Person {

    private Optional<Car> car;

    public Optional<Car> getCar() {
        return car;
    }
}

public class Car {

    private Optional<Insurance> insurance;

    public Optional<Insurance> getInsurance() {
        return insurance;
    }
}

public class Insurance {

    private String name;

    public String getName() {
        return name;
    }
}

//여기서 컴파일 에러 나요 ㅜㅜ
Optional<String> name = optPerson.map(Person::getCar)**//Optional<Opional<Car>>**
																 **.map(Car::getInsurance)**
																 .map(Insurance::getName);
```

```java
String name = optPerson.**flatMap**(Person::getCar)
											 .**flatMap**(Car::getInsurance)
											 .map(Insurance::getName);
											 .orElse("Unknown");
```

flatMap연산으로 Optional을 평준화(평탄화)한다. 이론적으로 두 Optional을 합치는 기능을 수행하면서 둘중 하나라도 null이면 빈 Optional을 생성하는 연산이다.

```
**도메인 모델에 Optional을 사용하지 말자**

자바 언어 아키텍트인 브라이언 고츠는 Optional의 용도가 선택형 반환값을 지원하는 것이라고
명확하게 못박았다.

Optional클래스는 필드 형식을 사용할 것을 가정하지 않았으므로
Serializable 인터페이스를 구현하지 않는다.
그러므로 직렬화모델을 사용하는 도구, 프레임워크에서 문제가 생길 수 있다.
다음 예제처럼 Optional로 값을 반환받을 수 있는 메서드를 추가하는 방식을 권장한다.
```

```java
public class Persion {
	private Car car;
	public Optional<Car> **getCarAsOptional**(){
		return Optional.ofNullable(car);
	}
}
```

```java
public Set<String> getCarInsuranceNames(List<Person> persons) {
        return persons.stream()
                      **.map(Person::getCarAsOptional)
                      .map(optCar -> optCar.flatMap(Car::getInsuranceAsOptional))
                      .map(optInsurance -> optInsurance.map(Insurance::getName))**
                      .flatMap(Optional::stream)
                      .collect(toSet());
    }
```

또 우리가 자주 사용하는 get(), orElse() orElseGet(), orElseThrow(), ifPresent(), ifPresentOrElse()등등이 있다.

```java
public String sumString(String a, String b){
    return a+" "+b;
}

public Optional<String> sumOptString(){
    Optional<String> nicky = Optional.of("이승민");
    Optional<String> lucky = Optional.of("운이좋네");

    return nicky.flatMap(n -> lucky.map(l -> sumString(n,l)));
}
```

```java
// 조건문으로 단순 null 체크
Insurance insurance = ...;
if(insurance != null && "CambridgeInsurance".equal(insurance.getName())) {
    System.out.println("ok");
}

// Optional 객체에 filter 메서드 사용
Optional<Insurance> optInsurance = ...;
optInsurance.filter(ins -> "CambridgeInsurance".equals(ins.getName()))
            .ifPresent(x -> System.out.println("ok"););
```

```java
Map<Object> map = new HashMap();
map.get("key"); // null

Optional.ofNullable(map.get("key"));
```

```java
public static Optional<Integer> stringToInt(String s) {
    try{
        return Optional.of(Integer.parseInt(s));
    } catch (NumberFormatException e) {
        reutn Optional.empty(); // 예외 발생시 빈 Optional 반환
    }
}
```

stringToInt를 앞으로 사용하면 try/catch 로직을 사용하지 않아도 됩니다.

- **기본형 Optional을 사용하지 말아야 하는 이유**

스트림처럼 Optional도 기본형 특화 클래스 OptionalInt, OptionalLong, OptionalDouble 등을 제공한다.

하지만 Optional의 최대 요소 수는 한개 이므로 기본형 특화 클래스로 성능을 개선할 수 없다.

또한 기본형특화 클래스에는 map, filter 등의 메서드를 지원하지 않으므로 사용을 권장하지 않는다