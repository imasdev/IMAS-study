# CHAPTER 17 리액티브 프로그래밍

발표자: 이수환
스터디일정: 2022/06/16
주차별: 9주차

# chapter 17. 리액티브 프로그래밍

**이장의 내용**

1. 리액티브 프로그래밍을 정의하고 리액티브 매니패스토를 확인함.
2. 애플리케이션 수준, 시스템 수준의 리액티브 프로그래밍
3. 리액티브 스트림, 자바9 플로 apif를 사용한 예제코드
4. 널리 사용되는 리액티브 라이브러리 RxJava 소개
5. 여러 리액티브 스트림을 변환하고 합치는 RxJava 동작 살펴보기
6. 리액티브 스트림의 동작을 시각적으로 문서화 하는 마블 다이어그램

**새로운 리액티브 프로그래밍 패러다임의 중요성이 증가하는 이유.**

<최근 대규모 애플리케이션의 3가지 변화>

1. 빅데이터 : 보통 빅데이터는 페타바이트 단위로 구성되며 매일 증가한다.
2. 다양한 환경 : 모바일 디바이스 에서 수천개의 멀티 코어 프로세스로 실행되는 클라우드 기반 클러스터에 

이르기 까지 다양한 환경의 애플리케이션이 배포된다.

1. 사용패턴 : 사용자는 24시간 1년 내내 항상 서비스를 이용할 수 있으며, 밀리초 단위의 응답시간을 기대한다.

예전 소프르웨어 아키텍처로는 오늘날의 이런 요구사항을 만족 시킬 수 없다.

**모바일기기와 사물인터넷(iot)** 같은 기기들이 더욱 늘어나면서 상황이 더욱 심화 될 것이다.

리액티브 프로그래밍 에서는 다양한 시스템과 소스에서 들어오는 데이터항목 스트림을 **비동기적** 으로

처리하고 합쳐서 이런 문제를 해결한다. 사용자에게 높은 응답을 제공하고, 고장, 정전같은 상태에

대처할 수 있으며, 무거운 작업을 하고 있는 상황에서도 가용성을 제공한다.

리액티브 애플리케이션과 시스템의 발전된 특징은 지금부터 살펴볼 **리액티브 매니페스토** 에 요약되어 있다.

# 17.1 리액티브 매니패스토

리액티브 매니패스토는 2013~2014년에 걸쳐 정의되었으며, 

리액티브 애플리케이션과 시스템 개발의 핵심 원칙을 공식적으로 정의한다.

---

**반응성 (responseive)**

리액티브 시스템은 빠르고 일정하고 예상가능한 반응시간을 제공해야 한다.

**회복성 (resilient)**

장애가 발생해도 시스템은 반응해야한다.

컴포넌트의 실행 복제, 여러 컴포넌트의 시간과 공간 분리 

(발송자와 수신자가 독립적인 생명주기를 가지며, 서로 다른프로세스에서 실행됨.)

컴포넌트 위임등 다양한 기법을 제공한다.

**탄력성 (elastic)**

무거운 작업 부하가 발생하면 자동으로 관련 컴포넌트에 할당된 자원수를 늘리거나 줄여서 대처 가능해야 한다.

**메시지주도(massage-driven)**

회복성과 탄력성을 지원하려면 시스템을 구성하는 컴포넌트들의 약한 결합, 고립, 위치투명성이 유지되도록

비동기 메세지 전달에 의존해야 한다.

---

**리액티브 프로그래밍에 대한 다른 개념정리**

Reactive Programming이란 데이터 흐름과 전달에 관한 프로그래밍 패러다임이다.

우리는 주로 알고리즘 문제와 같이 절차를 명시하여 순서대로 실행되는  Imperative Programming(명령형 프로

그래밍)을 한다. 반면 Reactive Programming이란 데이터의 흐름을 먼저 정의하고 데이터가 변경되었을 때 

연관된 작업이 실행된다. 즉 프로그래머가 어떠한 기능을 직접 정해서 실행하는 것이 아닌, 시스템에 이벤트가 

발생했을 때 알아서 처리되는 것이다. 기존의 프로그래밍 방식을 Pull 방식, Reactive 프로그래밍 방식을 Push 

방식이라고도 한다. Pull 방식은 데이터를 사용하는 곳(Consumer)에서 데이터를 직접 가져와서 사용한다면,

Push 방식은 데이터의 변화가 발생한 곳에서 새로운 데이터를 Consumer에게 전달한다.

따라서 Reactive 프로그래밍은 주변 환경과 끊임없이 상호작용을 한다. 다만 프로그램이 주도하는 것이 아니라 

환경이 변하면 이벤트를 받아 동작함으로써 상호작용한다.

![스크린샷 2022-06-13 오후 5.47.40.png](CHAPTER%2017%20%E1%84%85%E1%85%B5%E1%84%8B%E1%85%A2%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20288650f9ad21470c8c5e8b43ddc01ad1/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-13_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_5.47.40.png)

## 17.1.1 애플리케이션 수준의 리액티브

애플리케이션 수준 리액티브 프로그래밍의 주요 기능은 **비동기**로 작업을 수행할 수 있다는 점이다.

비동기로 작업시 최신 멀티코어 cpu의 사용률을 극대화 할수 있으며, 이 목표를 달성할 수 있도록

리액티브 프레임워크와 라이브러리는 스레드를 퓨처, 액터, 이벤트루프등과 공유하고

처리할 이벤트를 변환하고 관리한다.

이들 기술은 스레드보다 가벼울 뿐만 아니라 개발자에게 큰 이득을 제공한다.

개발자 입장에서는 비동기 애플리케이션의 추상수준을 높을 수 있으므로, 동기블록, 경쟁조건, 데드락

같은 멀티스레드 문제를 직접 처리할 필요가 없어지면서 비즈니스 요구사항을 구현하는데 더 집중할 수 있다.

## 17.1.2 시스템 수준의 리액티브

리액티브 시스템은 여러 애플리케이션이 한개의 일관적인, 회복할 수 있는 플랫폼을 구성할수 있게 해주고

이들 애플리케이션중 하나가 실패해도 전체 시스템은 계속 운영될 수 있도록 도와주는 아키텍처 이다.

리액티브 시스템의 주요속성으로 메세지 주도를 꼽을 수 있다.

수신자와 발신자가 메세지에 결합되지 않도록 메세지를 비동기로 처리하여 각 컴포넌트들의

결합도를 낮추어 각 컴포넌트에서 발생한 장애를 고립시킴으로서 회복성을 유지한다.

그리고 위치투명성을 이용하여 탄력성을 유지한다.

위치 투명성이란 리액티브 시스템의 모든 컴포넌트가 수신자의 위치에 상관없이 다른 모든 서비스와 통신할수 

있음을 의미한다. 위치 투명성 덕분에 시스템을 복제할 수 있으며, 현재 작업부하에 따라 자동으로 

애플리케이션을 확장 할 수 있다.

# 17.2 리액티브 스트림과 플로API

리액티브 프로그래밍은 리액티브 스트림을 사용하는 프로그래밍이다.

리액티브 스트림은 잠재적으로 무한의 비동기 데이터를 순서대로. 그리고 블록하지 않는 역압력을

전제해 처리하는 표준 기술이다.

역압력이란?

역압력은 발행-구독 프로토콜에서 이벤트 스트림의 발행자가 이벤트를 제공하는 속도보다

구독자의 이벤트 소비 속도가 느릴 경우 문제가 발생하지 않도록 보장하는 장치이다.

(컴포넌트가 불능이 되거나, 이벤트를 잃어버리는 등의 문제발생 방지)

이벤트 발생 속도를 늦추라고 알리거나, 소비가능한 이벤트의 양을 알리거나, 현재 이벤트를

처리하는데 얼마나 시간이 걸리는지를 업스트림 발행자에게 알리는 방식이다.

스트림의 비동기적 특성상 역압력 기능은 필수이다.

**리액티브 스트림을 제공하는 라이브러리들 (사용처)**

1. Java9 의 Flow, Akka스트림 (라이트밴드)
2. 리액터(Reactor) (피보탈)
3. RxJava (넷플릭스)
4. Vert.x (레드햇)

## 17.2.1 Flow 클래스 소개

자바9에서는 리액티브 프로그래밍을 제공하는 클래스 java.util.concurrent.Flow 를 추가했다.

이 클래스는 정적 컴포넌트 하나를 포함하고 있으며, 인스턴스화 할 수 없다.

발행 - 구독 모델을 지원할 수 있도록 flow클래스는 인터페이스 4개를 포함하고 있다.

1. Publisher
2. Subscriber
3. Subscription
4. Processor

Publisher는 항목을 발행.

Subscriber는 발행된 항목을 한개씩, 또는 여러개씩 소비한다.

Subscription이 이과정을 관리할 수 있도록 Flow 클래스는 관련 인터페이스와 정적 메서드를 제공한다.

Publisher 는 함수형 인터페이스로, Subscriber는 Publisher가 발행한 이벤트의 리스너로 자신을 등록한다.

Subscription 은 Publisher와 Subscriber 사이의 제어흐름, 역압력을 관리한다.

Flow.Publisher 인터페이스

```java
@FunctionalInterface
public interface Publisher<T> {
    void subscriber(Subscriber<? super T> s);
}
```

`subscriber(Subscriber s)` : 매개변수로 받은 Subscriber를 이 Publisher 에 구독시킴.

- Subscriber 인터페이스

```java
public interface Subscriber<T> {
    void onSubscribe(Subscription s);
    void onNext(T t);
    void onError(Throwable t);
    void onComplete();
}
```

이들 4종류의 이벤트는 다음 순서대로 실행된다.

onSubscriber → onNext * 다수 → onError 또는 onComplete

`onSubscriber` : Publisher 가 subscriber 를 구독 시킬때 onSubscribe 메소드로 Subscription 

객체를 전달

`onNext` : onNext 메소드로 다음요청

`onError` : 에러발생시 알림

`onComplete` : 종료시 알림

- Subscription 인터페이스

```java
public interface Subscription {
    void request(long n);
    void cancel();
}
```

Subscription 인터페이스는 메소드 2개를 정의한다.

`request` :  Publisher에게 주어진 개수의 이벤트를 처리할 준비가 되었음을 알린다.

`cancel` :  Subscription 을 취소. 더이상 Publisher에게 이벤트를 받지 않음을 통지한다.

**위 3개 인터페이스의 규칙 요약**

1. Publisher는 반드시 Subscription 의 request 에 정의된 개수 이하의 요소만
Subscriber 에게 전달해야한다.
2. Subscriber는 요소를 받아 처리할 수 있음을 Publisher 에 알려야 한다.
이러한 방식으로 Subscriber는 Publisher에 역압력을 행사할 수 있고, Subscriber가 관리할 수 없을 정도로 많은 요소를 받는 일을 피할 수 있다.
3. Publisher와 Subscriber 는 같은 Subscription 을 공유해야 하마 각각 고유한 역할을 수행해야 한다.

![스크린샷 2022-06-14 오전 9.25.29.png](CHAPTER%2017%20%E1%84%85%E1%85%B5%E1%84%8B%E1%85%A2%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20288650f9ad21470c8c5e8b43ddc01ad1/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-14_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_9.25.29.png)

Flow클래스의 네번째이자 마지막 멤버 Processor 인터페이스는 단지 Publisher 와 Subscribe를

상속받을 뿐 아무 메소드도 추가하지 않는다.

- Flow.Processor 인터페이스

```java
public interface Processor<T,R> extends Subscriber<T>, Publisher<R> {

}
```

이 인터페이스는 리액티브 스트림에서 처리하는 이벤트의 변환 단계를 나타낸다.

Processor가 에러를 수신하면 이로부터 회복하거나 즉시 onError 신호로 모든 Subscriber에 에러를 

전파할 수 있다. 마지막 Subscriber 가 Subscription 을 취소하면 Processor는 자신의 업스트림 

Subscription도 취소함으로 취소신호를 전파해야 한다.

## 17.2.2 첫번째 리액티브 애플리케이션 만들기

자바9 플로api 를 직접이용해 리액티브 애플리케이션을 개발하면서 지금까지 배운 네 개의 인터페이스가

어떻게 동작하는지 쉽게 확인할 수 있다. 리액티브 원칙을 적용해 온도를 보고하는 간단한 프로그램을 만들어본다.

1. TempInfo. 원격 온도계를 흉내낸다. (0~99 화씨온도를 임의로 만들어 연속적으로 보고
2. TempSubscriber. 레포트를 관찰하면서 각 도시에 설치된 센서에 보고한 온도 스트림을 출력한다.

먼저 현재 보고된 온도를 전달하는 간단한 클래스를 만든다.

- TempInfo : 현재 보고된 온도를 전달하는 자바 빈

```java
package com.kevin.Flow;
import lombok.Getter;
import java.util.Random;

@Getter
public class TempInfo {

    public static final Randomrandom= new Random();
    private final String town;
    private final int temp;

    public TempInfo(String town, int temp){
        this.town = town;
        this.temp = temp;
    }

    //정적 팩토리 메소드를 이용해 해당 도시의 TempInfo 인스턴스를 만든다.
    public static TempInfo fetch(String town){
        //10% 확률로 온도 가져오기 작업이 실패하도록 설정
        if(random.nextInt(10) == 0){
            throw new RuntimeException("Error!");
        }
        //0~99사이의 값을 임의로 반환한다.
        return new TempInfo(town,random.nextInt(100));
    }
}

```

다음은 Subscriber가 요청할 때마다 해당 도시의 온도를 전송하도록 Subscription을 구현한다.

- TempSubscription : Subscriber 에게 TempInfo 스트림을 전송하는 Subscription

```java
package com.kevin.Flow;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class TempSubscription implements Subscription {
    private final Subscriber<? super TempInfo> subscriber;
    private final String town;

    public TempSubscription (
			 Subscriber<? super TempInfo> subscriber,
			 String town
		){
        this.subscriber = subscriber;
        this.town = town;
    }

    @Override
    public void request(long n) {
        //subscriber 가 만든 요청을 n개 만큼 실행
        for(long i = 0L; i < n ; i++) {
            try {
                //현재 온도를 subscriber 로 전달
                subscriber.onNext(TempInfo.fetch(town));
            } catch (Exception e) {
                //온도 가져오기 실패 시 subscriber로 에러 전달
                subscriber.onError(e);
                break;
            }
        }
    }

    @Override
    public void cancel() {
        //구독이 취소되면 완료 신호를 subscriber 에게 전달.
        subscriber.onComplete();
    }
}

```

이번에는 새 요소를 얻을 때 마다 Subscription이 전달한 온도를 출력하고, 

새 레포트를 요청하는 Subscriber 클래스를 구현한다.

- TempSubscriber : 받은 온도를 출력하는 Subscriber

```java
package com.kevin.Flow;

public class TempSubscriber implements Subscriber<TempInfo> {

    private Subscription subscription;

    @Override
    public void onSubscribe(Subscription subscription) {
        //구독을 저장하고, 첫번째 요청을 전달
        this.subscription = subscription;
        subscription.request(1);
    }
    @Override
    public void onNext(TempInfo tempInfo) {
        //수신한 온도를 출력하고 다음 정보를 요청
        System.out.println(tempInfo.getTown()+"/"+tempInfo.getTemp());
        subscription.request(1);
    }
    @Override
    public void onError(Throwable t) {
        System.err.println(t.getMessage());
    }
    @Override
    public void onComplete() {
        System.out.println("Done!");
    }
}
```

다음은 리액티브 애플리케이션이 실제 동작할 수 있도록 Publisher를 만들고 TempSubscriber를 이용해

Publisher에 구독하도록 Main클래스를 만든다.

- Main : Publisher를 만들고 TempSubscriber 를 구독시킴

```java
package com.kevin.Flow;

public class Main {
    public static void main(String[] args) {
        //뉴욕으로 publisher를 만들고 TempSubscriber를 구독시킴.
        getTemperatures("New York").subscriber(new TempSubscriber());
    }

    
		//구독한 Subscriber 에게 TempSubscription 을 전송하는 Publisher를 반환하는 메소드
    //Publisher를 생성하면서 subscriber 를 구독시킴. 
		//구독시킬때 subscription 객체를 생성하여 subscriber 에 전달

		private static Publisher<TempInfo> getTemperatures(String town){
        return new Publisher<TempInfo>() {
            @Override
            public void subscriber(Subscriber<? super TempInfo> subscriber) {
                subscriber.onSubscribe(new TempSubscription(subscriber, town));
            }
        };
    }
}
```

```java
private static Publisher<TempInfo> getTemperatures(String town){
      //위 함수형인터페이스 구현 형식을 람다로
      return subscriber -> subscriber.onSubscribe(
              new TempSubscription(subscriber,town)
      );
  }
```

실행결과

![스크린샷 2022-06-15 오후 2.19.57.png](CHAPTER%2017%20%E1%84%85%E1%85%B5%E1%84%8B%E1%85%A2%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20288650f9ad21470c8c5e8b43ddc01ad1/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-15_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_2.19.57.png)

위에서 TempSubscription 은 뉴욕의 온도를 4번 성공적으로 전달했지만, 여섯번째에 에러가 발생했다.

Flow API의 세가지 인터페이스를 이용해 원하는 기능을 구현한것 같다.

그런데 코드에 문제가 없는걸까?

**Quiz. 1)** 

현재 TempInfo.fetch() 에서 10% 확률로 에러 발생시키는 부분을 삭제하고 실행하면 어떻게 될까?

- 
    
    정답) TempSubscriber 가 새로운 요소를 onNext 메소드로 받을 때마다 TempSubscription 으로
    
    새 요청을 보내면 TempSubscription의 request 메소드가 TempSubscriber 자신에게
    
    또 다른 요소를 보내는 문제가 있다. 이런 재귀호출은 스택오버플로우가 발생할 때 까지 반복해서 일어난다.
    
    ![스크린샷 2022-06-15 오후 2.25.51.png](CHAPTER%2017%20%E1%84%85%E1%85%B5%E1%84%8B%E1%85%A2%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20288650f9ad21470c8c5e8b43ddc01ad1/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-15_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_2.25.51.png)
    

이 문제를 어떻게 해결할까?

TempSubscription 에 Executor 를  에 추가하면 다른 스레드에서 TempSubscriber 로

새 요소를 전달하도록 할 수 있다.

```java
private static final ExecutorService executor = 
Executors.newSingleThreadExecutor();

@Override
public void request(long n) {
		executor.submit(() -> {
        for(long i = 0L; i < n ; i++) {
            try {
                subscriber.onNext(TempInfo.fetch(town));
            } catch (Exception e) {
                subscriber.onError(e);
                break;
            }
        }
    });
}
```

다시 실행하면 StackOverflowError 에러 없이 계속 출력되는 것을 확인할 수 있다.

## 17.2.3 Processor로 데이터 변환하기

앞에서 설명한 것처럼 Processor는 Subscriber이며 동시에 Publisher 다.

사실 Processor의 목적은 Publisher 를 구독한 다음 수신한 데이터를 가공해 다시 제공하는 것이다.

화씨로 제공된 데이터를 섭씨로 변환해 다시 방출하는 예제를 만들어 보자.

TempProcessor 클래스

```java
package com.kevin.Flow;

//TempInfo 를 받아서 다른 TempInfo로 반환하는 프로세서
public class TempProcessor implements Processor<TempInfo, TempInfo>{
    
		private Subscriber<? super TempInfo> subscriber;

    @Override
    public void subscriber(Subscriber<? super TempInfo> subscriber) {
        this.subscriber = subscriber;
    }
    @Override
    public void onNext(TempInfo tempInfo) {
        //인자로 받은 TempInfo의 온도를 섭씨로 변환한 새 TempInfo를 onNext로 전달
        subscriber.onNext(
					new TempInfo(tempInfo.getTown(), (tempInfo.getTemp() -32) * 5 / 9)
				);
    }

    //나머지 신호들은 모두 subscriber에게 그대로 전달
    @Override
    public void onSubscribe(Subscription s) {
        subscriber.onSubscribe(s);
    }

    @Override
    public void onError(Throwable t) {
        subscriber.onError(t);
    }

    @Override
    public void onComplete() {
        subscriber.onComplete();
    }
}

```

다음은 Main프로세스에 TempProcessor 를 적용해본다.

```java
package com.kevin.Flow;

public class Main {
    public static void main(String[] args) {
        getCelsiusTemperatures("New York").subscriber(new TempSubscriber());
    }

    private static Publisher<TempInfo> getCelsiusTemperatures(String town){
        return subscriber -> {
            //TempProcessor을 만들고 Subscriber 와 Publisher 사이로 연결함.
            TempProcessor processor = new TempProcessor();
            processor.subscriber(subscriber);
            processor.onSubscribe(new TempSubscription(processor, town));
        };
    }
}
```

결과

![스크린샷 2022-06-15 오후 2.44.58.png](CHAPTER%2017%20%E1%84%85%E1%85%B5%E1%84%8B%E1%85%A2%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20288650f9ad21470c8c5e8b43ddc01ad1/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-15_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_2.44.58.png)

플로 api 에 정의된 인터페이스를 직접 구현하면서 플로 api의 핵심인 발행-구독 프로토콜을 이용한

비동기 스트림처리를 살펴보았다.

## 17.2.4 자바는 왜 플로 api 구현을 제공하지 않는가?

자바9의 플로api는 조금 이상하다. 자바 라이브러리는 보통 인터페이스와 구현을 제공하는 반면,

플로 api는 구현을 제공하지 않으므로, 개발자가 직접 인터페이스를 구현했다.

왜일까? api를 만들 당시 Akka, RxJava 등 다양한 리액티브 스트림의 라이브러리가 이미 존재했기 때문이다.

원래 같은 발행-구독 사상에 기반해 리액티브 프로그래밍을 구현 했지만, 이들 라이브러리는 독립적으로

개발되었고, 서로 다른 이름규칙과 api를 사용했다. 그래서 자바9의 표준화 과정에서

이들 라이브러리는 공식적으로 java.util.concurrent.Flow 의 인터페이스를 기반으로 구현하도록

진화되었다. 이 표준화 덕분에 다양한 라이브러리가 쉽게 협력할 수 있게 되었다.

# 17.3 리액티브 라이브러리 RxJava 사용하기

- Observer 인터페이스와 Observable 클래스는 java9 에서 제거되었다.
- 새로운 코드는  플로api 를 사용해야 한다. 이 변화에 맞춰 RxJava 가 어떻게 진화할 것인지는 지켜봐야 한다.

→ java버전 8로 낮추면 `import java.util.Observable;` 는 되지만.. `just` 메소드를 찾을 수 없는등
제대로 사용할 수 없는 것 같음.

참고 : [https://4z7l.github.io/2020/12/01/rxjava-1.html](https://4z7l.github.io/2020/12/01/rxjava-1.html)

RxJava는 ReactiveX의 Java 언어 라이브러리로, 2013년 2월 넷플릭스에서 처음 소개하였다. 2016년 10월

RxJava2를 발표하였으며 가장 최근인 2020년 2월 RxJava3를 배포했다.공식사이트의 ReactiveX 설명은 

다음과 같다.

---

ReactiveX is a library for composing asynchronous and event-based programs by using observable sequences.
It extends the observer pattern to support sequences of data and/or events and adds operators that allow you to compose
sequences together declaratively while abstracting away concerns about things like low-level threading, synchronization,
thread-safety, concurrent data structures, and non-blocking I/O.

---

번역하면,

ReactiveX는 관찰가능한 절차를 통해 비동기, 이벤트 기반 프로그램을 구성하기 위한 라이브러리이다.

Observer Pattern을 확장하며, Sequence를 조합할 수 있는 연산자를 지원한다.

low-level Thread, 동기화, Thread 안전성, non-blocking I/O에 관한 우려를 줄인다.

RxJava는 러닝 커브가 가파르다. 전통적인 스레드 기반의 프로그래밍을 사용하는 자바와 접근 방식이 다르기 

때문이다. 여러 스레드를 사용하는 경우 예상치 못한 문제가 발생할 수도 있고 디버깅하기도 어렵다.

따라서 RxJava는 함수형 프로그래밍 방식을 도입하였다. 함수형 프로그래밍은 Side Effect가 없는 순수 함수를

지향하기 때문에 Thread-Safe하다.

ReactiveX의 이러한 방식을 구성하게 해주는 핵심 요소가 바로 Observable과 Operator(연산자)이다.

RxJava 예제

```java
import io.reactivex.rxjava3.core.Observable;

public class RxExample {
    public static void main(String[] args) {
        Observable.just(1,2,3)
                .map(x->x*10)
                .subscribe(System.out::println);
    }
}
```

`Observable` : ReactiveX의 핵심 요소이자 데이터 흐름에 맞게 Consumer에게 알림을 보내는 Class이다.

`just()` : 가장 간단한 Observable 생성 방식이다. (생성 연산자라고도 한다)

`map()` : RxJava의 연산자이다. 데이터를 원하는 형태로 바꿀 수 있다.

`subscribe()` : Observable은 구독(subscribe)을 해야 데이터가 발행된다. 따라서 Observable을 구독하여 

데이터를 발행 후, 수신한 데이터를 원하는 방식으로 사용(System.out::println)한다.

## 17.3.1 Marble Diagram

Marble Diagram은 RxJava를 이해하는 핵심 도구이다. 데이터의 흐름과 연산자를 이해하는 데 큰 도움이 된다.

예제에서 실행한 코드를 Marble Diagram으로 바꾸면 다음과 같다.

![스크린샷 2022-06-14 오후 4.41.39.png](CHAPTER%2017%20%E1%84%85%E1%85%B5%E1%84%8B%E1%85%A2%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20288650f9ad21470c8c5e8b43ddc01ad1/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-14_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_4.41.39.png)

- 1, 2, 3이 연산자 map을 거쳐 10, 20, 30이 되었다

![스크린샷 2022-06-15 오후 3.02.34.png](CHAPTER%2017%20%E1%84%85%E1%85%B5%E1%84%8B%E1%85%A2%E1%86%A8%E1%84%90%E1%85%B5%E1%84%87%E1%85%B3%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20288650f9ad21470c8c5e8b43ddc01ad1/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-15_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_3.02.34.png)

# 17.4 마치며

- 리액티브 프로그래밍의 기초사상은 20~30년전에 수립되었지만, 많은 데이터처리량과 빠른 응답때문에 최근에 인기를 얻고 있다.
- 리액티브 소프트웨어가 지녀야할 4가지 특징은 반응성, 회복성, 탄력성, 메세지 주도 이다.
- 리액티브 애플리케이션은 리액티브 스트림이 전달하는 한개 이상의 이벤트를 비동기로 처리함을 전제로 한다.
- 리액티브 스트림은 비동기적으로 처리되므로 역압력 기법이 기본적으로 탑재되어 있다. 역압력은 발행자가 구독자보다 빠른 속도로 이벤트를 발행함으로 발생하는 문제를 방지한다.