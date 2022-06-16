# CHAPTER 9 리팩터링, 테스팅, 디버깅

발표자: 권윤옥
스터디일정: 2022/05/19 오후 6:00
주차별: 5주차
참여자: 권윤옥, 남채린, 이수환, 이승민, 장현준

# 9.1 가독성과 유연성을 개선하는 리팩터링

코드 가독성을 개선한다는 것은 우리가 구현한 코드를 다른 사람이 쉽게 이해하고 유지보수할 수 있게 만드는 것을 의미함.

이 장에선 람다, 메서드 참조, 스트림을 활용해서 코드 가독성을 개선할 수 있는 리팩터링 예제를 소개함

## 익명클래스를 람다 표현식으로 리팩터링

```java
Runnable r1 = new Runnable() {
    @Override
    public void run(){
        System.out.println("hello");
    }
};

Runnable r2 = () -> System.out.println("hello");
```

### But, 모든 익명 클래스를 람다 표현식으로 변환할 수 있는 것은 아님

1. 익명 클래스에서 사용한 `this`와 `super`는 람다표현식에서 다른 의미를 가짐 → 블로깅을 통해 예제 확인하기
2. 익명 클래스는 감싸고 있는 클래스의 변수를 가릴 수 있다(섀도 변수). 하지만 람다 표현식으로는 변수를 가릴 수 없다. (컴파일이 되지않음)

![Untitled](CHAPTER%209%20%E1%84%85%E1%85%B5%E1%84%91%E1%85%A2%E1%86%A8%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC,%20%E1%84%90%E1%85%A6%E1%84%89%E1%85%B3%E1%84%90%E1%85%B5%E1%86%BC,%20%E1%84%83%E1%85%B5%E1%84%87%E1%85%A5%E1%84%80%E1%85%B5%E1%86%BC%203194f18852274eae9ef68fb565f4d6eb/Untitled.png)

1. 익명 클래스를 람다 표현식으로 바꾸면 콘텍스트 오버로딩에 따른 모호함이 초래될 수 있다 
    
    다음과 같이 Runnable과 같은 시그니처를 갖는 Task라는 함수형 인터페이스를 선언했을 때 문제가 발생함
    
    ```java
    public interface Task
    {
        public void execute();
    }
    
    public static void doSomething(Runnable r)
    {
        r.run();
    }
    public static void doSomething(Task t)
    {
        t.execute();
    }
    ```
    

![Untitled](CHAPTER%209%20%E1%84%85%E1%85%B5%E1%84%91%E1%85%A2%E1%86%A8%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC,%20%E1%84%90%E1%85%A6%E1%84%89%E1%85%B3%E1%84%90%E1%85%B5%E1%86%BC,%20%E1%84%83%E1%85%B5%E1%84%87%E1%85%A5%E1%84%80%E1%85%B5%E1%86%BC%203194f18852274eae9ef68fb565f4d6eb/Untitled%201.png)

→ 이는 명시적 형변환(타입 캐스팅)를 통해 해결할 수 있음

```java
doSomething( (Task)() -> System.out.println("danger!!");
```

이런 부분은 인텔리제이의 리팩터링 기능을 이용하면 자동으로 해결된다고 한다!! 👍

## 람다 표현식을 메서드 참조로 리팩터링

람다 표현식 대신 메서드 참조의 메서드명으로 코드의 의도를 명확하게 알릴 수 있기 때문에 추천!

```java
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream()
        .collect(
                groupingBy(dish -> {
                    if (dish.getCalories() <= 400) return CaloricLevel.DIET;
                    else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
                    else return CaloricLevel.FAT;
}));
```

위와 같은 칼로리 수준으로 요리를 그룹화하는 코드가 있을때, groupingBy에 전달한 람다 표현식을 별도의 메서드로 추출하여 전달하면 코드가 간결해진다.

```java
public class Dish{
	
		public CaloricLevel getCaloricLevel()
	  {
	      if (this.calories <= 400)
	      {
	          return CaloricLevel.DIET;
	      }
	      else if (this.calories <= 700)
	      {
	          return CaloricLevel.NORMAL;
	      }
	      else
	      {
	          return CaloricLevel.FAT;
	      }
	  }
}

Map<CaloricLevel, List<Dish>> dishesByCaloricLevel2 = 
			menu.stream().collect(groupingBy(Dish::getCaloricLevel));
```

`comparing`과 `maxBy` 같은 정적 헬퍼 메서드를 활용하는 것도 좋음

- comparing : 함수형 인터페이스 `Comparator`의 정적 메서드
- maxBy : `Collectors` API 에서 제공하는 정적 메서드

```java
inventory.sort(
		(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));

inventory.sort(comparing(Apple::getWeight));
```

Collectors API가 제공하며 메서드 참조와 함께 사용할 수 있는 내장 헬퍼 메서드를 활용 (예: sum, maximum)

```java
int totalCalories = menu.stream().map(Dish::getCalories)
																 .reduce(0, (c1, c2) -> c1 + c2);

int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
```

아래와 같이 내장 컬렉터를 사용해서 자신이 어떤 동작을 수행하는지 메서드 이름으로 설명이 가능한 코드를 작성할 수 있음.

## 명령형 데이터 처리를 스트림으로 리팩터링

명령형 코드는 필터링과 추출로 엉킨 코드로 이는 전체 구현을 자세히 살펴본 이후에야 코드의 의도를 이해할 수 있다. 

```java
List<String> dishNames = new ArrayList<>();
for(Dish dish : menu){
		if(dish.getCalories() > 300){
				dishNames.add(dish.getName());
		}
}
```

스트림 API를 이용하면 문제를 더 직접적으로 기술할 수 있고 쉽게 병렬화가 가능함

```java
menu.parallelStream()
		.filter(d -> d.getCalories() > 300)
		.map(Dish::getName)
		.collect(toList());
```

## 코드 유연성 개선

### 조건부 연기 실행

클라이언트 코드에서 객체 상태를 자주 확인하거나, 객체의 일부 메서드를 호출하는 상황이라면 내부적으로 객체의 상태를 확인한 다음에 메서드를 호출하도록 새로운 메서드를 구현하는 것이 좋다. (왜냐하면 코드 가독성이 좋아지고 캡슐화도 강화되기 때문)

🤔 조건부 연기 실행 패턴과 실행 어라운드 패턴을 자주 사용한다고 책에 기재되어 있었지만 필자는 체감이 가지 않아서 패스

# 9.2 람다로 객체지향 디자인 패턴 리팩터링하기

### 디자인 패턴(design pattern)

다양한 패턴을 유형별로 정리한 것이며, 공통적인 소프트웨어 문제를 설계할 때 재사용할 수 있는 검증된 청사진을 제공함.

## 전략(Strategy)패턴

한 유형의 알고리즘을 보유한 상태에서 런타임에 적절한 알고리즘을 선택하는 기법, 다양한 기준을 갖는 입력값을 검증하거나, 다양한 파싱 방법을 사용하거나, 입력 형식을 설정하는 등 다양한 시나리오에 활용 가능

![출처: [https://velog.io/@hero6027/Strategy전략-Pattern](https://velog.io/@hero6027/Strategy%EC%A0%84%EB%9E%B5-Pattern)](CHAPTER%209%20%E1%84%85%E1%85%B5%E1%84%91%E1%85%A2%E1%86%A8%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC,%20%E1%84%90%E1%85%A6%E1%84%89%E1%85%B3%E1%84%90%E1%85%B5%E1%86%BC,%20%E1%84%83%E1%85%B5%E1%84%87%E1%85%A5%E1%84%80%E1%85%B5%E1%86%BC%203194f18852274eae9ef68fb565f4d6eb/Untitled%202.png)

출처: [https://velog.io/@hero6027/Strategy전략-Pattern](https://velog.io/@hero6027/Strategy%EC%A0%84%EB%9E%B5-Pattern)

### 전략 패턴의 구성

- 알고리즘을 나타내는 인터페이스(Strategy 인터페이스)
- 다양한 알고리즘을 나타내는 한 개 이상의 인터페이스 구현(ImplA, ImplB)
- 전략 객체를 사용하는 한 개 이상의 클라이언트

```java
public interface ValidationStrategyInterface
{
    boolean execute(String s);
}

public class IsAllLowerCase implements ValidationStrategyInterface
{
    @Override
    public boolean execute(String s)
    {
        return s.matches("[a-z]+");
    }
}

public class IsNumeric implements ValidationStrategyInterface
{
    @Override
    public boolean execute(String s)
    {
        return s.matches("\\d+");
    }
}

public class Validator
{
    private final ValidationStrategyInterface strategy;

    public Validator(ValidationStrategyInterface v)
    {
        this.strategy = v;
    }

    public boolean validate(String s)
    {
        return strategy.execute(s);
    }
}
```

위와 같이 구현한 검증 전략 클래스를 다음과 같이 사용할 수 있을 것이다.

```java
public static void main(String[] args)
{
    Validator numericValidator = new Validator(new IsNumeric());
    boolean b1 = numericValidator.validate("aaaa");

    Validator lowerCaseValidator = new Validator(new IsAllLowerCase());
    boolean b2 = lowerCaseValidator.validate("bbbb");
}
```

### 람다 표현식 사용한 리팩토링

```java
Validator numericValidatorLambda = new Validator((String s) -> s.matches("[a-z]+"));
boolean b3 = numericValidatorLambda.validate("aaaa");

Validator lowerCaseValidatorLambda = new Validator((String s) -> s.matches("\\d+"));
boolean b4 = lowerCaseValidatorLambda.validate("bbbb");
```

람다 표현식을 사용하면 전략 디자인 패턴에서 발생하는 자잘한 코드를 제거할 수 있음. 람다표현식으로 전략 디자인 패턴을 대신할 수 있기 때문에 람다 표현식 사용 추천!!

## 템플릿 메서드(Template method) 패턴

템플릿 메소드 패턴이란 특정 작업을 처리하는 일부분을 서브 클래스로 캡슐화하여 전체적인 구조는 바꾸지 않으면서 특정 단계에서 수행하는 내용을 바꾸는 패턴

알고리즘의 구조를 메소드에 정의하고, 하위 클래스에서 알고리즘 구조의 변경없이 알고리즘을 재정의 하는 패턴이다. 알고리즘이 단계별로 나누어 지거나, 같은 역할을 하는 메소드이지만 여러곳에서 다른형태로 사용이 필요한 경우 유용한 패턴이다.

```java
abstract class OnlineBanking
{
    public void processCustomer(int id)
    {
        Customer c = DataBase.getCustomerWithId(id);
        makeCustomerHappy(c);
    }

    abstract void makeCustomerHappy(Customer c);
}
```

위의 코드에서 알 수 있듯이 각 지점별로 OnlineBanking 클래스를 상속받아 원하는 대로 makeCustomerHappy 메서드를 구현하면 됨

### 람다 표현식 사용

processCustomer 메서드에 makeCustomerHappy 메서드와 같은 시그니처( T → void )를 갖는 인수를 추가한다. 

```java
abstract class OnlineBanking
{
    public void processCustomer(int id, Consumer<Customer> makeCustomerHappy)
    {
        Customer c = DataBase.getCustomerWithId(id);
        makeCustomerHappy.accept(c);
    }
//    abstract void makeCustomerHappy(Customer c);
}
```

OnlineBanking 클래스를 상속받지 않고 람다 표현식을 전달해서 다양한 동작을 추가할 수 있음

```java
new OnlineBanking().processCustomer(1337, (Customer c) -> 
		System.out.println(c + "You've got bonus!!"));
```

## 옵저버(Observer) 패턴

어떤 이벤트가 발생했을 때 한 객체(주제)가 다른 객체 리스트(옵저버)에 자동으로 알림을 보내야 하는 상황에서 사용되는 패턴

![출처: [https://m.blog.naver.com/sule47/220907636623](https://m.blog.naver.com/sule47/220907636623)](CHAPTER%209%20%E1%84%85%E1%85%B5%E1%84%91%E1%85%A2%E1%86%A8%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC,%20%E1%84%90%E1%85%A6%E1%84%89%E1%85%B3%E1%84%90%E1%85%B5%E1%86%BC,%20%E1%84%83%E1%85%B5%E1%84%87%E1%85%A5%E1%84%80%E1%85%B5%E1%86%BC%203194f18852274eae9ef68fb565f4d6eb/Untitled%203.png)

출처: [https://m.blog.naver.com/sule47/220907636623](https://m.blog.naver.com/sule47/220907636623)

```java
public interface Observer
{
    void notify(String tweet);
}

public class NYTimes implements Observer
{
    @Override
    public void notify(String tweet)
    {
        if (tweet != null && tweet.contains("money"))
        {
            System.out.println("Breaking news in NY! " + tweet);
        }
    }
}

public class Guardian implements Observer
{
    @Override
    public void notify(String tweet)
    {
        if (tweet != null && tweet.contains("queen"))
        {
            System.out.println("Yet more news from London... " + tweet);
        }
    }
}

public class Lemonde implements Observer
{
    @Override
    public void notify(String tweet)
    {
        if (tweet != null && tweet.contains("wine"))
        {
            System.out.println("Today cheese, wine and news! " + tweet);
        }
    }
}
```

```java
public interface Subject
{
    void registerObserver(Observer o);
    void notifyObservers(String tweet);
}

public class Feed implements Subject
{
    private final List<Observer> observers = new ArrayList<>();

    @Override
    public void registerObserver(Observer o)
    {
        this.observers.add(o);
    }

    @Override
    public void notifyObservers(String tweet)
    {
        observers.forEach(o -> o.notify(tweet));
    }
}
```

주제와 옵저버를 연결하는 애플리케이션은 다음과 같이 만들 수 있다.

```java
public static void main(String[] args)
{
    Feed f = new Feed();
    f.registerObserver(new NYTimes());
    f.registerObserver(new Guardian());
    f.registerObserver(new Lemonde());
    f.notifyObservers("The queen said her favorite book is Modern Java In Action!");
}
```

### 람다 표현식 사용

Observer 인터페이스를 구현하는 모든 클래스(NYTimes, Guardian, Lemonde)가 하나의 메서드 notify를 구현하고 있음. 따라서 람다 표현식을 직접 전달해서 실행할 동작을 지정할 수 있음

```java
f.registerObserver((String tweet) -> {
    if (tweet != null && tweet.contains("money"))
    {
        System.out.println("Breaking news in NY! " + tweet);
    }
});
```

그러나, 옵저버가 상태를 가지며, 여러 메서드를 정의하는 등의 복잡한 로직이 구현되어 있다면 람다 표현식보다 기존의 클래스 구현방식을 고수하는 것이 바람직할 수 있음

## 의무 체인(Chain of Responsibility) 패턴

한 객체가 어떤 작업을 처리한 다음에 다른 객체로 결과를 전달하고, 다른 객체도 해야 할 작업을 처리한 다음에 또 다른 객체로 전달하는 형식의 패턴

![출처: [https://leejaedoo.github.io/refactoring/](https://leejaedoo.github.io/refactoring/)](CHAPTER%209%20%E1%84%85%E1%85%B5%E1%84%91%E1%85%A2%E1%86%A8%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC,%20%E1%84%90%E1%85%A6%E1%84%89%E1%85%B3%E1%84%90%E1%85%B5%E1%86%BC,%20%E1%84%83%E1%85%B5%E1%84%87%E1%85%A5%E1%84%80%E1%85%B5%E1%86%BC%203194f18852274eae9ef68fb565f4d6eb/Untitled%204.png)

출처: [https://leejaedoo.github.io/refactoring/](https://leejaedoo.github.io/refactoring/)

```java
public abstract class ProcessingObject<T>
{
    protected ProcessingObject<T> successor;

    public void setSuccessor(ProcessingObject<T> successor)
    {
        this.successor = successor;
    }

    public T handle(T input)
    {
        T r = handleWork(input);
        if (successor != null)
        {
            return successor.handle(r);
        }
        return r;
    }
    abstract protected T handleWork(T input);
}
```

위와 같은 작업 처리 객체가 있다고 가정했을 때, 텍스트를 처리하는 두 작업 처리 객체를 다음과 같이 만들 수 있다.

```java
public class HeaderTextProcessing extends ProcessingObject<String>
{
    @Override
    protected String handleWork(String text)
    {
        return "From Raoul, Mario, and Alan: " + text;
    }
}

public class SpellCheckerProcessing extends ProcessingObject<String>
{
    @Override
    protected String handleWork(String text)
    {
        return text.replaceAll("labda", "lambda");
    }
}
```

```java
public static void main(String[] args)
{
    ProcessingObject<String> p1 = new HeaderTextProcessing();
    ProcessingObject<String> p2 = new SpellCheckerProcessing();

    p1.setSuccessor(p2);    // 두 작업 처리 객체를 연결
    String result = p1.handle("Aren't labdas really good?!");
    System.out.println(result);
}
```

### 람다 표현식 사용

위의 패턴은 함수 체인과 비슷한 형식을 갖고 있어 람다 표현식을 조합하는 방법을 활용할 수 있음

```java
UnaryOperator<String> headerProcessing = (String text) -> 
						"From Raoul, Mario, and Alan: " + text;
UnaryOperator<String> spellCheckerProcessing = (String text) -> 
						text.replaceAll("labda", "lambda");

Function<String, String> pipeline = headerProcessing.andThen(spellCheckerProcessing);
String resultOfLambda = pipeline.apply("Aren't labdas really good?!");
System.out.println(resultOfLambda);
```

## 팩토리(Factory) 패턴

인스턴스화 로직을 클라이언트에 노출하지 않고 객체를 만들 때 사용하는 패턴

```java
public class ProductFactory
{
    public static Product createProduct(String name)
    {
        switch (name)
        {
            case "loan":
                return new Loan();
            case "stock":
                return new Stock();
            case "bond":
                return new Bond();
            default:
                throw new RuntimeException("No such product " + name);
        }
    }
}
```

Loan, Stock, Bond 모두 Product의 서브형식이며, 위의 코드는 생성자와 설정을 외부로 노출하지 않으면서 클라이언트가 단순하게 상품을 생성할 수 있다는 장점이 있음.

```java
Product p = ProductFactory.createProduct("loan");
```

### 람다 표현식 사용

생성자를 메서드 참조처럼 접근하여 다음과 같이 ProductFactory를 바꿀 수 있음 

```java
public class ProductFactory
{
    final static Map<String, Supplier<Product>> map= new HashMap<>();
    static {
        map.put("loan", Loan::new);
        map.put("stock", Stock::new);
        map.put("bond", Bond::new);
    }

		public static Product createProduct(String name)
    {
        Supplier<Product> productSupplier = map.get(name);
        if (productSupplier != null)
        {
            return productSupplier.get();
        }
        throw new RuntimeException("No such product " + name);
    }
}
```

위의 예시에서는 생성자에 전달하는 파라미터가 없기 때문에 깔끔하게 구현이 가능했지만, 생성자에 인수를 여러개 전달해야하는 상황에서는 이 기법을 적용하기 어렵다. 

# 9.3 람다 테스팅

사실 가장 중요한 것은 제대로 작동하는 코드를 구현하는 것이지 깔끔한 코드를 구현하는 것이 아니기 때문에 검증이 필요함! 하여, 단위 테스트를 진행하자!

```java
public class Point
{
    private final int x;
    private final int y;

    public Point(int x, int y)
    {
        this.x = x;
        this.y = y;
    }

    public int getX()
    {
        return x;
    }

    public int getY()
    {
        return y;
    }

    public Point moveRightBy(int x)
    {
        return new Point(this.x + x, this.y);
    }
}
```

위와 같은 클래스를 생성했을 때, 다음과 같은 단위테스트를 작성할 수 있음

```java
@Test
void testMoveRightBy()
{
    // given
    Point p1 = new Point(5, 5);

    // when
    Point p2 = p1.moveRightBy(10);

    // then
    assertEquals(15, p2.getX());
		assertEquals(5, p2.getY());
}
```

## 보이는 람다 표현식의 동작 테스팅

람다는 익명이므로 테스트 코드 이름을 호출할 수 없지만, 필요에 따라 람다를 필드에 저장해서 로직을 테스트할 수 있음

```java
public final static Comparator<Point> compareByXAndThenY = Comparator.comparing(Point::getX)
                                                                     .thenComparing(Point::getY);
```

위와 같이 람다식을 필드에 저장하면 다음과 같이 테스트 코드를 작성할 수 있음

```java
@Test
public void testComparingTwoPoints(){
		Point p1 = new Point(10, 15);
		Point p2 = new Point(10, 20);
		int result = Point.compareByXAndThenY(p1, p2);
		assertTrue(result < 0);
}
```

## 람다를 사용하는 메서드의 동작에 집중

람다 표현식을 사용하는 메서드의 동작을 테스트함으로써 람다를 공개하지 않으면서도 람다 표현식을 검증

```java
public static List<Point> moveAllPointsRightBy(List<Point> points, int x)
{
    return points.stream()
                 .map(p -> new Point(p.getX() + x, p.getY()))
                 .collect(toList());
}
```

위와 같은 람다표현식이 사용된 메서드를 생성했을 경우 아래와 같이 테스트 코드를 작성하여 메서드의 동작을 확인할 수 있음

```java
@Test
public void testMoveAllPointsRightBy(){
		List<Point> points = Arrays.asList(
				new Point(5, 5),
				new Point(10, 5)
		);
		List<Point> expectedPoints = Arrays.asList(
				new Point(15, 5),
				new Point(20, 5)
		);
		List<Point> newPoints = Point.moveAllPointsRightBy(points, 10);
		assertEquals(expectedPoints, newPoints);
 
}
```

## 복잡한 람다를 개별 메서드로 분할하기

람다 표현식을 새로운 일반 메서드로 선언하여 메서드 참조로 바꾸어 일반 메서드를 테스트 하듯이 진행할 수 있다고 하는데 이 부분은 위에 언급한 내용과 동일한 것으로 보임

# 9.4 디버깅

문제가 발생한 코드를 디버깅할 때 개발자는 스택 트레이스와 로깅을 확인해야 하는데, 람다에서는 이를 확인하고 이해하기 매우 까다롭다.

필자는 이에 대해 이 부분은 미래의 자바 컴파일러가 개선해야 할 부분이라고 하여 자바 8 이후에 나온 버전에서 개선된 적이 있는지 확인해보았으나 아직까지 개선된 바가 없는 것으로 보임

대신 스트림의 중간 연산 단계별로 각각 어떤 결과를 도출하는지 확인하는 방법이 있는데, 이는 `peek()` 이라는 스트림 연산을 활용하면 된다.

```java
List<Integer> numbers = Arrays.asList(2, 3, 4, 5);

List<Integer> result = numbers.stream()
															.peek(x -> System.out.println("from stream: " + x))
															.map(x -> x + 17)
															.peek(x -> System.out.println("after map: " + x))
															.filter(x -> x % 2 ==0 )
															.peek(x -> System.out.println("after filter: " + x))
															.limit(3)
															.peek(x -> System.out.println("after limit: " + x))
															.collect(toList());
```