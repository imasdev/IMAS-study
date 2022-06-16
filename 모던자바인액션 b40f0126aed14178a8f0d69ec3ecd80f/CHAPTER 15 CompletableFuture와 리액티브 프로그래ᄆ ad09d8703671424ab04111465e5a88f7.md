# CHAPTER 15 CompletableFuture와 리액티브 프로그래밍 컨셉의 기초

발표자: 장현준
스터디일정: 2022/06/09
주차별: 8주차
참여자: 남채린, 이수환, 이승민, 장현준

- 프로세스와 스레드
    
    # 프로세스와 스레드
    
    - 프로세스란 운영체제에서 실행 중인 하나의 어플리케이션을 의미한다. 프로그램을 실행하면 **운영체제로부터 실행에 필요한 메모리를 할당 받아 해당하는 코드를 실행하는 작업의 단위**를 프로세스라 한다.
    
    ### 그렇다면 스레드는 무엇일까?
    
    - **스레드**는 **프로세스 안에서 독립적으로 실행되는 흐름의 단위**를 말함 그리고 하나의 프로세스가 여러 스레드를 실행하는 것을 **멀티 스레드**라고 한다.
- Future class란?
    - 비동기적인 연산의 결과를 표현하는 클래스입니다. Future를 이용하면 멀티쓰레드 환경에서 처리된 어떤 데이터를 다른 쓰레드에 전달 할 수 있다.
    - Future 내부적으로 Thread-Safe하도록 구현되었기 때문에 `Synchronized block` 을 사용하지 않아도 된다.
    
    ### Future 예제 1
    
    ```java
    ExecutorService executor
            = Executors.newSingleThreadExecutor();
    
    Future<Integer> future = executor.submit(() -> {
        System.out.println(LocalTime.now() + " Starting runnable");
        Integer sum = 1 + 1;
        Thread.sleep(3000);
        return sum;
    });
    
    System.out.println(LocalTime.now() + " Waiting the task done");
    Integer result = future.get();
    System.out.println(LocalTime.now() + " Result : " + result);
    ```
    
    - 이 예제에서는 두개의 Thread가 생성되었고, Futrue를 이용하여 어떤 쓰레드에서 처리된 데이터를 다른 쓰레드로 전달한다.
    
    ### 결과
    
    ```java
    23:19:57.134359 Waiting the task done
    23:19:57.134802 Starting runnable
    23:20:00.163916 Result : 2
    ```
    
    - `ExecutorService` 는 Single Thread를 생성한다. `submit()` 으로 Callable을 전달하면, 인자로 전달된 Callable()을 수행한다.
    - executor.submit() 가 호출되었을 때 Future 객체는 리턴된다. 하지만 아직 값이 설정되지 않은 상태이다.
    - future.get()는 Future 객체에 어떤 값이 설정될 때까지 기다린다. `submit()` 에 전달된  Callable이 어떤 값을 리턴하면 그 값을 Future에 설정한다.
    
    ### Future 예제 2
    
    ```java
    CompletableFuture<Integer> future
            = new CompletableFuture<>();
    
    Executors.newCachedThreadPool().submit(() -> {
        System.out.println(LocalTime.now() + " Doing something");
        Integer sum = 1 + 1;
        Thread.sleep(3000);
        future.complete(sum);
        return null;
    });
    
    System.out.println(LocalTime.now() + " Waiting the task done");
    Integer result = future.get();
    System.out.println(LocalTime.now() + " Result : " + result);
    ```
    
    - `CompletableFuture<Integer>`으로 Future 객체를 생성!
    - `Executors.newCachedThreadPool()` 는 Thread pool을 생성하여 submit()으로 Callable을 전달하면 대기 중인 Thread로부터 실행된다.
    - `future.complete(data)`으로 다른 쓰레드로 전달한 데이터를 저장한다.
    - `future.get()`으로 데이터가 set될 때까지 기다린다.
    
    ### 결과
    
    ```java
    23:42:02.293044 Doing something
    23:42:02.292976 Waiting the task done
    23:42:05.318822 Result : 2
    ```
    
    ### Future 예제 3
    
    ```java
    ExecutorService executor
            = Executors.newSingleThreadExecutor();
    
    Future<Integer> future = executor.submit(() -> {
        System.out.println(LocalTime.now() + " Starting runnable");
        Integer sum = 1 + 1;
        Thread.sleep(4000);
        System.out.println(LocalTime.now() + " Exiting runnable");
        return sum;
    });
    
    System.out.println(LocalTime.now() + " Waiting the task done");
    Integer result = null;
    try {
        result = future.get(2000, TimeUnit.MILLISECONDS);
    } catch (TimeoutException e) {
        System.out.println(LocalTime.now() + " Timed out");
        result = 0;
    }
    System.out.println(LocalTime.now() + " Result : " + result);
    ```
    
    - Future 에 데이터가 set되지 않으면 `Future.get()` 를 호출한 Thread는 무한히 대기한다. 그럼 프로그램은 응답 x.
    - `Future.get()`에 TimeOut을 설정하여 일정 시간 내에 응답이 없으면 리턴하여 다음 작업을 처리하도록 만들 수 있다.
    
    ```java
    Future.get(long timeout, TimeUnit unit)
    ```
    
    ### 결과
    
    ```java
    11:55:06.693 Waiting the task done
    11:55:06.693 Starting runnable
    11:55:08.696 Timed out
    11:55:08.696 Result : 0
    11:55:10.696 Exiting runnable
    ```
    
    - get(2000, TimeUnit.MILLISECONDS)는 2000ms를 Timeout으로 설정한다는 의미이다.
    - Timeout이 발생하면 `TimeoutException` 예외가 발생함.
    

<aside>
💡 이 장의 내용
1. Thread, Future, 자바가 풍부한 동시성 API를 제공하도록 강요하는 진화의 힘
2. 비동기 API
3. 동시 컴퓨팅의 박스와 채널 뷰
4. CompletableFuture 콤비네이터로 박스를 동적으로 연결
5. 리액티브 프로그래밍용 자바 9 플로 API의 기초를 이루는 발행 구독 프로토콜
6. 리액티브 프로그래밍과 리액티브 시스템

</aside>

# 15.1 동시성을 구현하는 자바 지원의 변화

- Runnable, Thread → ExecutorService, Callable<T>, Future<T>, 제네릭  → RecursiveTask → 람다 → 분산 비동기 프로그래밍

# 15.1.1 스레드와 높은 수준의 추상화

- 자바 스트림을 사용하면 외부반복 대신 내부반복을 통해 쉽게 병렬성을 달성할 수 있음.

```java
sum = Arrays.stream(param).parallel().sum();
```

→ 스트림으로 추상화하는 것은 디자인 패턴을 적용하는 것과 비슷하지만 대신 쓸모 없는 코드가 라이브러리 내부로 구현되면서 복잡성도 줄어든다는 장점이 있음.

### 동시성이란?

![Untitled](CHAPTER%2015%20CompletableFuture%E1%84%8B%E1%85%AA%20%E1%84%85%E1%85%B5%E1%84%8B%E1%85%A2%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%20ad09d8703671424ab04111465e5a88f7/Untitled.png)

![Untitled](CHAPTER%2015%20CompletableFuture%E1%84%8B%E1%85%AA%20%E1%84%85%E1%85%B5%E1%84%8B%E1%85%A2%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%20ad09d8703671424ab04111465e5a88f7/Untitled%201.png)

- 하나의 CPU 에서 2개의 프로세스가 있다고 가정해보자. 이 둘은 엄청나게 짧은시간에 컨텍스트 위치가 일어나며 번갈아서 실행된다.

### 병렬성이란?

![Untitled](CHAPTER%2015%20CompletableFuture%E1%84%8B%E1%85%AA%20%E1%84%85%E1%85%B5%E1%84%8B%E1%85%A2%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%20ad09d8703671424ab04111465e5a88f7/Untitled%202.png)

![Untitled](CHAPTER%2015%20CompletableFuture%E1%84%8B%E1%85%AA%20%E1%84%85%E1%85%B5%E1%84%8B%E1%85%A2%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%20ad09d8703671424ab04111465e5a88f7/Untitled%203.png)

# 15.1.2 Executor와 스레드 풀

### 스레드의 문제

- 자바 스레드는 직접 운영체제 스레드에 접근한다. 운영체제 스레드는 비용이 비싸고, 스레드 숫자도 제한되어 있음. 주어진 프로그램에서 사용할 최적의 자바 스레드 개수는 사용할 수 있는 하드웨어 코어 개수에 따라 달라짐.

### 스레드 풀 그리고 스레드 풀이 더 좋은 이유

- 스레드풀을 사용하면 하드웨어에 맞는 수의 Task를 유지함과 동시에 수 천개의 Task를 스레드 풀에 아무 오버헤드 없이 제출가능하다. 큐의 크기 조정, 거부 정책, Task 종류에 따른 우선순위 등 다양한 설정 가능.

### 스레드 풀 그리고 스레드 풀이 나쁜이유

- n 스레드를 가진 스레드 풀은 오직 n 만큼의 스레드를 동시에 실행가능하다. sleep 중이거나 I/O block 중인 스레드가 있다면 작업 block 할 수 있는 Task는 스레드에 제출하지 말아야하지만 이를 항상 지키긴 어렵다는 단점이 있다.
- 모든 스레드 풀이 종료되기 전에 프로그램을 종료하면, 워커 스레드가 다음 Task 제출을 기다리며 종료되지 않을 수 있으므로 주의해야한다.

# 15.1.3 스레드의 다른 추상화 : 중첩되지 않은 메서드 호출

- 7장(병렬 스트림 처리와  포크/조인 프레임워크)에서 사용한 동시성에서는 한 개의 특별한 속성, Task or Thread 가 메서드 호출 안에서 시작되면 그 메서드 호출은 반환하지 않고 작업을 끝나길 기다렸다.
다음과 같이 Thread 생성과 Join()이 한 쌍처럼 중첩된 메서드 호출을 `엄격한 포크/조인`이라 부른다.

![Untitled](CHAPTER%2015%20CompletableFuture%E1%84%8B%E1%85%AA%20%E1%84%85%E1%85%B5%E1%84%8B%E1%85%A2%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%20ad09d8703671424ab04111465e5a88f7/Untitled%204.png)

- 시작된 Task를 내부 호출이 아니라 외부 호출에서 종료되도록 기다리는 더 여유로운 방식의 포크/조인을 사용해도 비교적 안전함.

![Untitled](CHAPTER%2015%20CompletableFuture%E1%84%8B%E1%85%AA%20%E1%84%85%E1%85%B5%E1%84%8B%E1%85%A2%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%20ad09d8703671424ab04111465e5a88f7/Untitled%205.png)

- 하지만 이번 15장 (CompletableFuture와 리액티브 프로그래밍 컨셉 기초)에서는 다음 그림과 같이 사용자의 메서드 호출에 의해 스레드가 생성되고 메서드를 벗어나 계속 실행되는 동시성 형태에 초점을 맞춘다!!!

![Untitled](CHAPTER%2015%20CompletableFuture%E1%84%8B%E1%85%AA%20%E1%84%85%E1%85%B5%E1%84%8B%E1%85%A2%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%20ad09d8703671424ab04111465e5a88f7/Untitled%206.png)

- 이렇게 메서드가 반환된 후에도 만들어진 태스크 실행이 계속되는 메서드를 `비동기 메서드`라고 지칭한다.

### 비동기 메서드 호출 시 주의사항

- 스레드 실행은 메서드를 호출한 다음의 코드와 동시에 실행되므로 데이터 경쟁 문제를 일으키지 않도록 조심해야한다.
- 기존 실행중이던 스레드가 종료되지 않은 상황에서 자바의 main()메서드가 반환되면 문제가 발생할 수 있음.
*→ 그렇다면 현재 우리 현장서비스 등록할 때는 등록이 되면 main 메서드가 반환되는게 아닌가??*

# 15.2 동기 API와 비동기 API

- 다음과 같은 시그니처를 갖는 f, g 두 메서드의 호출을 합하는 예제를 살펴보자

```java
int f(int x);
int g(int x);

int y = f(x);
int z = g(x);
System.out.println(y + z);
```

```java
class ThreadExample {
  public static void main(String[] args) throws InterruptedException {
    int x = 1337;
    Result result = new Result();
    
    Thread t1 = new Thread(() -> { result.left = f(x); });
    Thread t2 = new Thread(() -> { result.right = g(x); });
    t1.start();
    t2.start();
    t1.join();
    t2.join();
    System.out.println(result.left + result.right);
  }
  
  private static class Result {
    private int left;
    private int right;
  }
}
```

→ Runnable 대신 Future API 인터페이스를 사용해서 더 단순화 할 수 있음

```java
public class ExecutorServiceExample {
  pubvlic static void main(String[] args) throws ExecutionException, InterruptedException {
    int x = 1337;
    
    ExecutorService executorService = Executors.newFixedThreadPool(2);
    Future<Integer> y = executorService.submit(() -> f(x));
    Future<Integer> z = executorService.submit(() -> g(x));
    System.out.println(y.get() + z.get());
    
    executorService.shutdown();
  }
}
```

- 명시적 반복으로 병렬화를 수행하던 코드를 스트림을 이용해서 내부반복으로 바꾼 것 처럼, 비동기 API라는 기능으로 API를 바꿔서 해결가능하다.

# 15.2.2 Future 형식 API

# 15.2.2 리액티브 형식 API( With Callable )

- 위에서 코드로 설명하겠습니다.

# 15.2.3 잠자기 (그리고 기타 블로킹 동작)는 해로운 것으로 간주

- 스레드 풀에서 Sleep중인 Task는 다른 Task가 시작되지 못하게 막으므로 자원을 소비한다. 이상적으로는 절대 Task에서 기다리는 일을 만들지 말거나 코드에서 예외를 일으키는 방법으로 이를 처리가능하다. **(시간제한)
Task를 앞과 뒤 두 부분으로 나누고 블록되지 않을 때만 뒷부분을 자바가 스케줄링하도록 요청할 수 있다.**

### A코드

```java
work1();
Thread.sleep(10000);
work2();
```

### B코드

```java
public class ScheduledExecutorServiceExample {
  public static void main(String[] args) {
    ScheduledExecutorService scheduledExecutorService = Executor.newScheduledThreadPool(1);
    
    work1();
    scheduledExecutorService.schedule(ScheduledExecutorServiceExample::work2, 10, timeUnit.SECONDS);
    
    scheduledExecutorService.shutdown();
  }
  
  public static void work1() {
    ...
  }
  
  
  public static void work2() {
    ...
  }
}
```

- A 코드는 sleep하는 동안 스레드 자원을 점유하는 반면, B코드는 다른 작업이 실행될 수 있도록 허용함.

# 15.2.4 비동기 API에서 예외는 어떻게 처리할까?

- 비동기 API에서 호출된 메서드의 실제 바디는 별도의 스레드에서 호출되며 이때 발생하는 어떤 에러는 이미 호출자의 실행 범위와는 관계가 없는 상황이됨.
- Future를 구현한 CompletableFuture에서는 런타임 get()메서드에 예외를 처리할 수 있는 기능을 제공하며 예외에서 회복할 수 있도록 exceptionally() 같은 메서드도 제공됨

```java
CompletableFuture.supplyAsync(this::failingMSG)
								 .exceptionally(ex -> new Result(Status.FAILED))
								 .thenAccept(this::notify);
```

- 리액티브 형식의 비동기 API에서는 return 대신 기존 콜백이 호출되므로 예외가 발생했을 때 실행될 추가 콜백을 만들어 인터페이스를 바꿔야함.

# 15.3 박스와 채널 모델

- 박스와 채널 모델은 동시성을 설계하고 개념화하기 위한 모델이다. 박스와 채널 모델을 이용하면 생각과 코드를 구조화할 수 있으며, 시스템 구현의 추상화 수준을 높일 수 있다. 또한 병렬성을 직접 프로그래밍하는 관점을 콤비네이터를 이용해 내부적으로 작업을 처리하는 관점으로 바꿔준다.

![Untitled](CHAPTER%2015%20CompletableFuture%E1%84%8B%E1%85%AA%20%E1%84%85%E1%85%B5%E1%84%8B%E1%85%A2%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%20ad09d8703671424ab04111465e5a88f7/Untitled%207.png)

위 Task를 코드로 구현해보자

```java
int t = p(x);
System.out.println(r(q1(t), q2(t));
```

위에 방식은 q1, q2를 차례로 호출하여 H/W 병렬성 활용과는 거리가 멀다.

```java
int t = p(x);
Future<Integer> a1 = executorService.submit(() -> q1(t));
Future<Integer> a2 = executorService.submit(() -> q2(t));
System.out.println(r(a1.get(t), a2.get(t));
```

- 많은 Task가 get() 메서드를 호출해서 Future가 끝나기를 기다리게 되면 H/W의 병렬성을 제대로 활용하지 못하거나 데드락에 걸릴 수 있다.

# 15.4 CompletableFuture와 콤비네이터를 이용한 동시성

- CompletableFuture는 Future를 조합할 수 있는 기능이 있다.ComposableFuture가 아닌 CompletableFuture라 부르는 이유는 실행할 코드 없이 Future를 만들 수 있고, complete() 메서드를 이용해 다른 스레드가 완료한 후에 get()으로 값을 얻을 수 있도록 허용하기 때문이다.

```java
public class CFComplete {
  public static void main(String[] args) throws ExecutionException, InterruptedException {
    ExecutorService executorService = Executors.newFiexedthreadPool(10);
    int x = 1337;

    CompletableFuture<Integer> a = new CompletableFuture<>();
    executorService.submit(()-> a.complete(f(x)));
    int b = g(x);
    System.out.println(a.get()+b);

    executorService.shutdown();
  }
}
```

- f(x)와 g(x)를 동시에 실행해 합계를 구하는 위 코드에서, f(x)의 실행이 끝나지 않은 상황에서 get()을 기다리며 프로세싱 자원을 낭비할 수 있다.ComposableFuture<T>의 thenCombine 메서드를 사용하면 연산 결과를 효과적으로 더할 수 있다.

`ComposableFuture<V> thenCombine(CompletableFuture<U> other, Bifunction<T, U, V> fn)`💁🏻 이 메서드는 T, U 값을 받아 새로운 V값을 만든다.

```java
public class CFComplete {
  public static void main(String[] args) throws ExecutionException, InterruptedException {
    ExecutorService executorService = Executors.newFiexedthreadPool(10);
    int x = 1337;

    CompletableFuture<Integer> a = new CompletableFuture<>();
    CompletableFuture<Integer> b = new CompletableFuture<>();

    CompletableFuture<Integer> c = a.thenCombine(b, (y, z) -> y + z);
    executorService.submit(()-> a.complete(f(x)));
    executorService.submit(()-> b.complete(g(x)));

    System.out.println(c.get());
    executorService.shutdown();
  }
}
```

- 결과를 추가하는 세 번째 연산 c는 다른 두 작업이 끝날때까지 실행되지 않으므로 먼저 시작해서 블록되지 않는다.이전 버전의 y+z 연산은 g(x)를 실행한 스레드에서 수행되어 f(x)가 완료될 때까지 블록될 여지가 있었다.반면 thenCombine을 이용하면 f(x)와 g(x)가 끝난 다음에 덧셈 계산이 실행된다

# 15.5 발행-구독 그리고 리액티브 프로그래밍

- Future는 독립적 실행과 병렬성에 기반하므로, 한 번만 실행해 결과를 제공한다.
    
    반면 리액티브 프로그래밍은 시간이 흐르면서 여러 Future 같은 객체를 통해 여러 결과를 제공한다.
    
    또한 가장 최근의 결과에 대해 반응(react)하는 부분이 존재한다.
    

자바9에서는 java.util.concurrent.Flow 인터페이스에 발행-구독 모델을 적용해 리액티브 프로그래밍을 제공

플로 API를 간단히 정리하면 다음과 같다.

- 구독자가 구독할 수 있는 발행자
- 이 연결을 구독(subscription)이라 한다.
- 이 연결을 이용해 메시지(또는 이벤트)를 전송한다.

# 15.5.1 두 플로를 합치는 예제

- 두 정보 소스로부터 발생하는 이벤트를 합쳐서 다른 구독자가 볼 수 있도록 발행하는 예제를 살펴보자. 스프레드 시트의 셀 C3에  “=C1 + C2” 공식을 입력했을 때 제공되는 동작이라고 볼 수 있다.

```java
private class SimpleCell {
  private int value = 0;
  private String name;
  public SimpleCell(String name){
    this.name = name;
  }
}
...
SimpleCell c2 = new SimpleCell("c2");
SimpleCell c1 = new SimpleCell("c1");
```

- 통신할(?) 구독자를 인수로 받는 발행자 인터페이스와 구독자 인터페이스를 추가하자!

```java
interface Publisher<T> {
  void subscribe(Subscriber<? super T> subscriber);
}

interface Subscriber<T> {
  void onNext(T t);
}
```

### Cell은 publisher이며 동시에 Subscriber이다…?

```java
public class SimpleCell implements Publisher<Integer>, Subscriber<Integer> {
  private int value = 0;
  private String name;
  private List<Subscriber> subscribers = new ArrayList<>();

  public SimpleCell(String name) {
      this.name = name;
  }
    
  @Override
  public void subscribe(Flow.Subscriber<? super Integer> subscriber) {
    subscribers.add(subscriber);
  }
  
  private void notifyAllSubscribers() {
    subscribers.forEach(subscriber -> subscriber.onNext(this.value));
  }

  @Override
  public void onNext(Integer newValue) {
    this.value = newValue;
    System.out.println(this.name + ":" + this.value);
    notifyAllSubscribers();
  }
}
```

### 예제 실행

```java
SimpleCell c3 = new SimpleCell("c3");
SimpleCell c2 = new SimpleCell("c2");
SimpleCell c1 = new SimpleCell("c1");

c1.subscribe(c3);
c1.onNext(10);
c2.onNext(20);

//결과
//c1:10
//c2:20
//c3:10
```

# 15.5.2 역압력이 뭐야?

- 매 초마다 수천개의 메시지가 onNext로 전달된다면 빠르게 전달되는 이벤트를 아무 문제 없이 처리가 가능할까??? 이러한 상황을 압력(Pressure)이라 부른다.
- 이럴때는 정보의 흐름 속도를 제어하는 역압력 기법이 필요함. 역압력은 Subscriber가 Publisher로 정보를 요청할 때만 전달 할 수 있도록 한다

```java
void onSubscribe(Subscription subscription);
```

→ 우리로 치면 RPC호출???

# 15.6 리액티브 시스템 VS 리액티브 프로그래밍

- 리액티브 시스템은 런타임 환경이 변화에 대응하도록 전체 아키텍처가 설계된 프로그램을 가리킨다. 리액티브 시스템이 가져야 할 속성은 `반응성(responsive)`, `회복성(resilient)`, `탄력성(elastic)`의 세가지 속성으로 요약가능하다.이러한 속성을 구현하는 방법 중 하나로 리액티브 프로그래밍을 이용할 수 있다.