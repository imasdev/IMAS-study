# CHAPTER 16 CompletableFuture : 안정적 비동기 프로그래밍

발표자: 이승민
스터디일정: 2022/06/16
주차별: 9주차

<aside>
💡 **지난시간에 우리는**
자바 Future API에 대해서 알아보았습니다.
자바의 동시성 지원은 계속 진화해 왔으며 앞으로도 그럴 것입니다.
자바8 CompletableFuture 클래스는 한 번의 비동기 연산을 표현 합니다.
컨네비이터로 비동기 연산을 조합함으로 Future를 이용할때 발생했던 기존의 블로킹 문제를 해결할 수 있습니다.(compose(), andThen())
플로API는 발행-구독 프로토콜, 역압력을 이용하면 자바의 리액티브 프로그래밍의 기초를 제공합니다.
역압력: `발행-구독` 프로토콜에서 이벤트 스트림의 **구독자(Subscriber)가 발행자(Publisher)가 이벤트를 제공하는 속도보다 느린 속도로 이벤트를 소비하면서 문제가 발생하지 않도록 보장하는 장치**다.

</aside>

---

# Future의 단순 활용

- 자바5부터는 미래의 어느 시점에 결과를 얻는 모델에 활용할 수 있도록 Future인터페이스를 제공한다.
- Future는 저수준의 스레드에 비해 직관적으로 이해하기 쉽다.

```java
ExecutorService executorService = Executors.newCachedThreadPool();

Future<Double> future = executorService.submit(new Callable<Double>() {
    @Override
    public Double call() throws Exception {
        return **doSomeLongComputation()**; //시간이 오래 걸리는 작업을 비동기적으로 실행한다.
    }
});
**doSomethingElse();** // 비동기 작업을 하는 동안 다른 작업을 수행한다.
try {
    Double result = future.get(1, TimeUnit.SECONDS); // 최대 1초동안 기다린다.
} catch (ExecutionException e) {
    // 계산 중 예외 발생
} catch (InterruptedException e2) {
    // 현재 스레드에서 대기 중 인터럽트 발생
} catch (TimeoutException e3) {
    // Future가 완료되지 전에 타임아웃 발생
}
```

오래걸리는 작업이 영원히 끝나지 않으면? 작업이 끝나지 않는 문제가 생기므로 [예제16-1] 처럼 get메서드에서 최대 타임아웃 시간을 설정하는게 좋다.

## Future의 제한

여러 future의 결과가 있을 때 이들의 의존성은 표현하기가 어렵습니다.
`A의 계산이 끝나면 결과를 다른 오래걸리는 계산 B로 전달해줘 B계산 끝나면 다른 결과랑 조합해줘`

해줘! ⇒  future: 몰?루

그래서 자바8에서 CompletableFuture 클래스를 제공합니다.

CompletableFuture는 Stream과 비슷한 패턴, 즉 람다 표현식과 파이프라이닝을 활용합니다.

따라서 Future와 CompletableFuture의 관계는 Collection과 Stream관계에 비유할 수 있습니다.

---

# 비동기 API 구현

```java
public class Shop{
	public double getPrice(String product){
		return calculatePrice(product);
	}
	
	public double calculatePrice(String product){
		Util.delay(); //의도적으로 1초 딜레이 시킴
		return random.nextDouble() * product.charAt(0) + produce.charAt(1);
	}
}

public class Util{
	public static void delay() {
		try{
			Thread.sleep(1000);
		}catch(InterruptedException e){
			log.error(e);
			throw new RuntimeException(e);
		}
	}
}
```

### 동기메서드를 비동기 메서드로 변환

```java
public Future<Double> getPriceAsync(String product){
	CompletableFuture<Double> futurePrice = new CompletableFuture<>();
	new Thread( () -> {
		double price = calculatePrice(price); // 다른 스레드에서 비동기적으로 계산을 수행함.
		futurePrice.complete(price); // **오래 시간이 걸리는 계산이 완료되면 Future에 값을 설정한다.**
	}).start();
	return futurePrice; // **계산결과가 완료되길 기다리지 않고 Future를 반환한다**.
}
```

비동기 API 사용

```java
Shop shop = new Shop("BestShop");

Future<Double> futurePrice = shop.getPriceAync("my favorite product");

doSomethingElse(); // 제품가격을 계산하는 동안 다른 상점 검색 등 다른 작업 수행

try{
	double price = futurePrice.get(); // 가격정보를 가져온다.
}catch(Exception e){
	throw new RuntimeException(e);
}
```

지금까지 개발한 코드는 아무 문제없이 잘 작동 될 겁니다.

`**그런데 가격을 계산하는 동안 에러가 발생한다면?**` get() 메서드가 반환될 때까지 영원히 기다리게 될 수도 있다.

문제점을 개선해보자!

```java
public Future<Double> getPriceAsync(String product){
	CompletableFuture<Double> futurePrice = new CompletableFuture<>();
	new Thread(() -> {
		try{
			double price = calculatePrice(price); // 다른 스레드에서 비동기적으로 계산을 수행함.
			futurePrice.complete(price); // **정상적으로** **계산이 완료되면 Future에 값을 설정한다.**
		}catch(Exception e){
			futurePrice.completeExceptionally(e); // **비정상적일때 에러를 포함시켜 future를 종료한다.**
		}
	}).start();
	return futurePrice;
}
```

지금까지 CompletableFuture를 직접 만들어서 사용했습니다.

- 솔직히 맘에 안들어 왜와이. Future만드는거랑 뭐가 다른거지? 비동기로 개발하는거 어렵고 복잡하고 비즈니스 코드를 확인도 잘 안되니깐 가독성도 떨어져.. 그냥 자바설계자 까불이 마려워!
    
    ![Untitled](CHAPTER%2016%20CompletableFuture%20%E1%84%8B%E1%85%A1%E1%86%AB%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%8C%E1%85%A5%E1%86%A8%20%E1%84%87%E1%85%B5%E1%84%83%E1%85%A9%E1%86%BC%E1%84%80%E1%85%B5%20%E1%84%91%E1%85%B3%E1%84%85%2023a9f494717d4dfbac22eb34ecf919b4/Untitled.png)
    
    자바설계자: 워워 진정해~ 그럴줄 알고 준비했어😂
    
    ```java
    public Future<Double> getPriceAsync(String product) {
    	return CompletableFuture.supplyAsync(() -> calculatePrice(product));
    }
    ```
    
    - 야이쉐끼야 에러처리는 어디다 팔아먹었냐?
        
        ![Untitled](CHAPTER%2016%20CompletableFuture%20%E1%84%8B%E1%85%A1%E1%86%AB%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%8C%E1%85%A5%E1%86%A8%20%E1%84%87%E1%85%B5%E1%84%83%E1%85%A9%E1%86%BC%E1%84%80%E1%85%B5%20%E1%84%91%E1%85%B3%E1%84%85%2023a9f494717d4dfbac22eb34ecf919b4/Untitled.png)
        
        자바설계자: 잠깐잠깐! 내부적으로 다 만들어놨어~ 너가 만든 `getPriceAsync` 이거랑 똑같은거야 편하지?
        
        ![Untitled](CHAPTER%2016%20CompletableFuture%20%E1%84%8B%E1%85%A1%E1%86%AB%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%8C%E1%85%A5%E1%86%A8%20%E1%84%87%E1%85%B5%E1%84%83%E1%85%A9%E1%86%BC%E1%84%80%E1%85%B5%20%E1%84%91%E1%85%B3%E1%84%85%2023a9f494717d4dfbac22eb34ecf919b4/Untitled%201.png)
        

---

# 비블록 코드 만들기

계속해서 최저가격 검색 애플리케이션을 개발해야 합니다. 다음과 같은 상점 리스트가 있다고 가정하자.

```java
List<Shop> shops = Arrays.asList(new Shop("무신사스탠다드"),
																 new Shop("탑텐"),
																 new Shop("나이키"),
																 new Shop("아디다스"));
																// ...
```

제품명을 입력하면 상점 이름과 제품가격 문자열 정보를 포한하는 List를 반환하는 메서드`findPrices`를 구현해야 한다.

```java
public List<String> findPrices(String product){
	return shop.stream()
						 .map(shop -> String.format("%s price is %.2f", shop.getName(), shop.getPrice(product)))
						 .collect(toList());
} // **대략** **4초가 걸림**
```

```java
public List<String> findPrices(String product){
	return shop.**parallelStream()**
						 .map(shop -> String.format("%s price is %.2f", shop.getName(), shop.getPrice(product)))
						 .collect(toList());
} // **대략 1.1초 걸림**
```

여기서 CompletableFuture를 이용하면 얼마나 더 빨라 질 수 있을까?

결과적으로 지금 이 상태로는 비슷할 수 있다.
다만 CompleatableFuture는 병렬 스트림 버전에 비해 작업에 이용할 수 있는 다양한 Executor를 지정할 수 있다는 장점이 있다.
따라서 Executor로 스레드 풀의 크기를 조절하는 등 애플리케이션에 맞는 최적화된 설정을 만들 수 있다.

```java
public List<String> findPrices(String product) {
    List<CompletableFuture<String>> priceFutures = shops.stream()
            .map(shop -> CompletableFuture.supplyAsync(() -> String.format("%s is %.2f", shop.getName(), shop.getPrice(product))))
            .collect(Collectors.toList());

    return priceFutures.stream()
            .map(CompletableFuture::join)
            .collect(Collectors.toList());
} **// 4개의 스레드일때 대략 2초 걸림**
```

- CompletableFuture 리스트를 만든 후 계산이 끝난 결과를 추출하는 예시이다. CompletableFuture의 join메서드를 활용했는데 Future의 get 메서드와 같은 의미를 갖는다.
- 다른점으로는 join 메서드는 예외를 발생시키지 않는다는 점이다.
- 두 map연산을 하나의 스트림 파이프라인으로 처리하지 않고 분리했다는것에 주목하자. 스트림 연산은 Lazy한 방식으로 동작하기 때문에 하나의 파이프 라인으로 처리했다면 순차적으로 실행되게 된다.

애플리케이션에서 사용하는 자원을 고려하여 풀에서 관리하는 최적의 스레드 수에 맞게 Executor를 만드는 것이 효율적일것이다.

스레드가 너무 많을수록 사용하지 않는 스레드가 많아지고 서버가 크래시 날 수 있으므로 적정한 갯수를 지정하는것이 중요하다.
[https://github.com/garlickim/study-note/blob/main/book/java-concurrency-in-practice/[week8] 스레드 풀 활용.md](https://github.com/garlickim/study-note/blob/main/book/java-concurrency-in-practice/%5Bweek8%5D%20%EC%8A%A4%EB%A0%88%EB%93%9C%20%ED%92%80%20%ED%99%9C%EC%9A%A9.md)

```java
private final Executor executor = 
	Executors.newFixedThreadPool(Math.min(shops.size(), 100), // 상점 수만큼의 스레드를 갖는 풀을 생성한다.(스레드 수의 범위는 0과100사이) 
		new ThreadFactory() {
	    @Override
	    public Thread newThread(Runnable r) {
	        Thread t = new Thread(r);
	        t.setDaemon(true); // 프로그램 종료를 방해하지 않는 데몬 스레드를 사용.
	        return t;
	    }
	});
```

데몬 스레드는 자바 프로그램이 종료될때 강제로 실행이 종료될 수 있다.

이제 새로운 Executor를 팩토리 메서드 supply의 두 번재 인수로 전달 할 수 있다.

```java
CompletableFuture.supplyAsync(() -> shop.getName()+" price is "+shop.getPrice(product), executor));
// 5개 검색, 9개 검색 : 대략 1초 400개 상점까지 같은 성능을 유지한다.
```

---

# 비동기 작업 파이프라인 만들기

계속해서 어플리케이션을 개발해보자! 모든 상점이 하나의 할인 서비스를 사용하기로 했다고 가정하자.

```java
public class Discount {
  public enum Code {
      NONE(0), SILVER(5), GOLD(10), PLATINUM(15), DIAMOND(20);
      
      private final int percentage;

      Code(int percentage) {
          this.percentage = percentage;
      }
  }
  // 생략 ...
}
```

getPrice에 discount 로직을 넣는다.

```java
public String getPrice(String product) {
    double price = calculatePrice(product); //계산하는서버 딜레이
    Discount.Code value = **Discount.Code.values()[
						new Random().nextInt(Discount.Code.values().length)]**;
    return String.format("$s %.2f $s", name, price, value);
}
```

### 할인 서비스 구현

할인코드와 연계된 할인율은 언제든 바뀔수 있으므로 매번 서버에서 정보를 얻어와야 한다.

```java
public class Discount {
  public enum Code {
		// 생략..
  }

  public static String applyDiscount(Quote quote) {
    return quote.getShopName() + "price is " + Discount.apply(quote,getPrice, quote.getDiscountCoe());
  }

  private static double apply(double price, Code code) {
    delay();// discount서비스의 응답 지연을 흉내낸다.
    return format(price * (100 - code.percentage) / 100);
  }
}
```

결과적으로 getPrice에서는 계산하는로직에서 딜레이 + 할인하는 로직에서 딜레이가 생긴다.

이제 findPrics메서드를 구현해보자!

```java
public List<String> findPrices(String product) {
  return shops.stream()
              .map(shop -> shop.getPrice(product))
              .map(Quote::parse)
              .map(Discount::applyDiscount)
              .collect(toList());
}

// 💁‍♂️ 5개의 상점이 있을때 findPrices()메소드의 걸리는 시간은?
```

```java
public static List<String> findPrices(String product) {
    List<CompletableFuture<String>> priceFutures = shopList.stream()
            .map(shop -> **CompletableFuture.supplyAsync**(
                    () -> shop.getPrice(product), executor)
            )
            .map(future -> future.**thenApply**(Quote::parse))
            .map(future -> future.**thenCompose**(
                    quote -> **CompletableFuture.supplyAsync**(
                            () -> Discount.applyDiscount(quote), executor
                    )
                )
            )
            .collect(Collectors.toList());
    
    return priceFutures.stream().map(**CompletableFuture::join**)
            .collect(Collectors.toList());
}
```

두개의 독립적인 비동기 작업을 합쳐야 하는 경우가 있습니다. 이런 경우 thenCombine 메서드를 활용하시면 됩니다.

```java
Future<Double> futurePriceInUSD = 
        CompletableFuture.supplyAsync(() -> shop.getPrice(prduct))
        .thenCombine(CompletableFuture.supplyAsync(() -> exchangeService.getRate(Money.EUR, Money.USD)), (price, rate) -> price * rate);
//.orTimeOUt(3, TimeUnit.SECONDS);
```

Future의 단점으로는 계산이 길어지는 경우 무한정 대기할 수도 있습니다. CompletableFuture에서는 이를 위해 orTimeout 메서드를 제공합니다.

```java
public class CompleteOnTimeOutMethodTest {
   public static void main(String args[]) throws InterruptedException {
      int a = 10;
      int b = 15;
      CompletableFuture.supplyAsync(() -> {
         try {
            TimeUnit.SECONDS.sleep(5);
         } catch(InterruptedException e) {
            e.printStackTrace();
         }
         return a + b;
      })
      **.completeOnTimeout(0, 4, TimeUnit.SECONDS)**
      .thenAccept(result -> System.out.println(result));
      TimeUnit.SECONDS.sleep(10);
   }
}
```

---

# CompleatableFuture의 종료에 대응하는 방법

CompletableFuture는 thenAccept라는 메서드를 제공합니다. 이 메서드는 연산 결과를 소비하는 Consumer를 인수로 받아 사용합니다.

```java
CompletableFuture[] futures = shopList.stream()
    .map(shop -> CompletableFuture.supplyAsync(
            () -> shop.getPrice(product), executor)
    )
    .map(future -> future.thenApply(Quote::parse))
    .map(future -> future.thenCompose(
            quote -> CompletableFuture.supplyAsync(
                    () -> Discount.applyDiscount(quote), executor
            )
        )
    )
    .map(f -> f.**thenAccept**(System.out::println))
    .toArray(size -> new CompletableFuture[size]);
```