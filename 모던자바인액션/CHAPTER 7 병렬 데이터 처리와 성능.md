# CHAPTER 7 병렬 데이터 처리와 성능

발표자: 이승민
스터디일정: 2022/05/12 오후 6:30
주차별: 4주차
참여자: 권윤옥, 남채린, 이승민, 장현준

이 장에서는 스트림으로 데이터 컬렉션 관련 동작을 얼마나 쉽게 병렬로 실행할 수 있는지 설명한다.

## 병렬 스트림

컬렉션에 parallelStream을 호출하면 **병렬 스트림**이 생성된다.

- 간단한 예제: 숫자 n을 인수로 받아서 1 ~ n까지의 모든 숫자 합계를 반환하는 메서드

```java
// 일반 스트림을 이용
public long sequentialSum(long n){
	return Stream.iterate(1L, i -> i+1) // 무한 자연수 스트림 생성
							 .limit(n)
							 .reduce(0L, Long::sum); // 모든 숫자를 더하는 스트림 리듀싱 연산
}

// 병렬 스트림을 이용
public long parallelSum(long n){
	return Stream.iterate(1L, i -> i+1)
							 .limit(n)
							 **.parallel()** // 스트림을 병렬 스트림으로 변환
							 .reduce(0L, Long::sum);
}
```

내부적으로 parallel을 호출하면 이후 연산이 병렬로 수행해야 함을 의미하는 불리언 플래그가 설정된다.

- 💁‍♂️ 병렬 스트림을 사용하면 성능이 좋아질까?
    
    병렬화를 이용하면 순차나 반복 형식에 비해 성능이 더 좋아질 것이라 추측했다. 하지만 소프트웨어 공학에서 추측은 위험한 방법이다. 특히 성능을 최적화할 때는 세 가지 황금 규칙을 기억해야 한다. 측정, 측정, 측정
    
    따라서 JMH라는 라이브러리를 이용해 작은 벤치마크를 구현할 것이다.
    자세한 내용은 책에서...
    
    결론적으로 특정한 상황에 있을땐 성능이 나빠질수 있다.
    그러므로 parallel메서드를 호출했을때 내부적으로 어떤 일이 일어나는지 꼭 이해해야 한다.
    
- 병렬 스트림의 올바른 사용법

```java
public long sideEffectSum(long n){
	Accumulator accumulator = new Accumulator();
	LongStream.rangeClosed(1,n).forEach(accumulator::add);
	return accumulator.total;
}

public class Accumulator {
	public long total = 0;
	public void add(long value){
		total+=value;
	}
}

/*실행하게되면 올바른 결과값이 나오지 않는다. 왜일까?*/
public long sideEffectParallelSum(long n){
	Accumulator accumulator = new Accumulator();
	LongStream.rangeClosed(1,n).**parallel()**.forEach(accumulator::add);
	return accumulator.total;
}

public long parallelRangeSum(){
	return LongStream.rangeClosed(1,N)
									 .parallel()
									 .reduce(0L, Long::sum);
}
```

병렬 스트림과 병렬 계산에서는 공유된 가변 상태를 피해야 한다.
18장 19장에서 상태 변화를 피하는 방법을 설명한다.

- 병렬 스트림 효과적으로 사용하기
1. 애매하다면 성능을 측정하라! 순차스트림 → 병렬스트림으로 쉽게 바꿀수 있으니 성능을 측정하는 것이 바람직하다.
2. 박싱을 주의하라. 자동 박싱과 언박싱은 성능을 크게 저하시킬 수 있는 요소! 자바8은 박싱 동작을 피할 수 있도록 **기본형 특화 스트림(IntStream, LongStream, DoubleStream)**을 제공
[https://ktko.tistory.com/entry/자바-박싱boxing과-언박싱unboxing](https://ktko.tistory.com/entry/%EC%9E%90%EB%B0%94-%EB%B0%95%EC%8B%B1boxing%EA%B3%BC-%EC%96%B8%EB%B0%95%EC%8B%B1unboxing)
3. 병렬 스트림에서 성능이 더 떨어지는 연산이 있다. limit, findFirst처럼 요소의 순서에 의존하는 연산은 병렬 스트림에서 수행하려면 비싼 비용을 치러야 한다. findAny를 사용해라.
4. 스트림에서 수행하는 전체 파이프라인 연산 비용을 고려하라. 처리할 요소 수를 N, 하나의 요소를 처리하는 비용을 Q라고 하면 전체 처리비용은 N * Q가 된다. Q가 높을때 병렬을 고려해보아라!
5. 소량의 데이터에서는 병렬 스트림이 도움 되지 않는다. 병렬화 과정에서 생기는 부가 비용을 상쇄할 수 있을 만큼의 이득을 얻지 못하기 때문
6. 스트림을 구성하는 자료구조가 적절한지 확인하라. ArrayList가 LinkedList보다 효율적으로 분할할 수 있다. LinkedList는 모든 요소를 탐색해야하지만 ArrayList는 요소를 탐색하지 않고 분할 할 수 있다. 또한 range팩토리 메서드로 만든 기본형 스트림도 쉽게 분해 할 수 있다. 마지막으로 커스텀 Spliterator를 구현해서 분해 과정을 완벽하게 제어할 수 있다.
7. 스트림의 특성과 파이프라인의 중간 연산이 스트림의 특성을 어떻게 바꾸는지에 따라 분해 과정의 성능이 달라질 수 있다. 필터 연산은 효과적이지 않다.
8. 최종 연산의 병합 과정비용을 살펴보라. 병합 과정의 비용이 비싸다면 병렬 스트림으로 얻는 이익이 상쇄 될 수 있다.

병렬 스트림은 내부적으로 자바7에서 추가된 포크/조인 프레임워크로 처리된다. 병렬 스트림의 내부 구조를 잘 알아야함으로 포크/조인 프레임워크를 살펴 보자.

## 포크/조인 프레임워크

포크/조인 프레임워크는 병렬화할 수 있는 작업을 재귀적으로 작은 작업으로 분할한 다음에 서브태스크 각각의 결과를 합쳐서 전체 결과를 만들도록 설계되었다.

포크(Fork) : 다른 프로세스 혹은 스레트(테스크태스크)를 여러 개로 쪼개서 새롭게 생성

조인(Join) : 포크해서 실행한 프로세스 혹은 스레드(태스크)의 결과를 취합한다.

![Untitled](CHAPTER%207%20%E1%84%87%E1%85%A7%E1%86%BC%E1%84%85%E1%85%A7%E1%86%AF%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%8E%E1%85%A5%E1%84%85%E1%85%B5%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%A5%E1%86%BC%E1%84%82%E1%85%B3%E1%86%BC%2098f4997467c2463d867aa0d86058c4f8/Untitled.png)

```java
if(태스크가 충분히 작다면) {
 순차적으로 태스트 계산
}
else{
	태스크를 분할 // 최대한 작을때까지 재귀적으로 호출함
	서브태스크의 결과를 합침
}
```

[RecursiveTask (Java Platform SE 8 )](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/RecursiveTask.html)

```java
import java.util.concurrent.RecursiveTask;

// RecursiveTask를 상속받아 포크조인 프레임워크에서 사용할 테스크 생성
public class ForkJoinSumCalculator extends RecursiveTask<Long> {  
        
    private final long[] numbers; // 더할 숫자 배열
    private final int start; // 이 서브테스크에서 처리할 배열의 초기 위치와 최종 위치
    private final int end;
    public static final long THRESHOLD = 10_000; // 이 값 이하의 서브테스크는 더 이상 분할 x

    // 메인 테스크를 생성할 때 사용할 공개 생성자
    public ForkJoinSumCalculator(long[] numbers) { 
        this(numbers, 0, numbers.length);
    }

    // 메인 테스트의 서브테스크를 재귀적으로 만들 때 사용할 private 생성자
    private ForkJoinSumCalculator(long[] numbers, int start, int end) {
        this.numbers = numbers;
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        int length = end - start; // 이 테스크에서 더할 배열의 길이
        if (length <= THRESHOLD) {
            // 기준값과 같거나 작으면 순차적으로 결과 계산
            return computeSequentially();
        } 
        // 배열의 첫 번째 절반을 더하도록 서브테스크 생성
        ForkJoinSumCalculator leftTask = 
                new ForkJoinSumCalculator(numbers, start, start+length/2);
        // ForkJoinPool의 다른 스레드로 새로 생성한 테스크를 비동기로 실행
        leftTask.fork();
        // 배열의 나머지 절반 더하는 서브테스크 생성
        ForkJoinSumCalculator rightTask =
                new ForkJoinSumCalculator(numbers, start, start+length/2);
        // 두번째 서브테스크 동기 실행. 추가 분할 발생할 수 있음
        Long rightResult = rightTask.compute();
        // 첫번째 서브테스크 결과를 읽거나 아직 결과가 없으면 기다림
        Long leftResult = leftTask.join();
        // 두 서브테스크의 결과를 조합한 값이 이 테스크의 결과값
        return rightResult + leftResult; 
    }

    // 더 분할할 수 없을 때 서브테스크의 결과를 계산하는 알고리즘
    private Long computeSequentially() {
        long sum = 0;
        for (int i = start; i < end; i++) {
            sum += numbers[i];
        }
        return sum;
    }
}
```

---

- join 메서드를 테스크에 호출하면 테스크가 생산하는 결과가 준비될 때까지 호출자를 블록시킨다.
    - 그러므로, 두 서브테스크가 모두 시작된 다음에 join을 호출해야한다.
    - 그렇지 않으면, 각각의 서브테스크가 다른 테스크가 끝나길 기다리게 되며 순차 알고리즘보다 느린 프로그램이 될 가능성이 높다
- `RecursiveTask` 내에서는 `ForkJoinPool`의 `invoke` 메서드를 사용하지 않아야 한다.
    - 대신 `compute`나 `fork` 메서드를 직접 호출할 수 있다. `invoke`는 순차 코드에서 병렬 계산을 시작할 때만 invoke를 사용한다.
- 서브테스크에 fork 메서드를 호출해서 `ForkJoinPool`의 일정을 조절할 수 있다.
    - 양쪽 작업 모두 `fork` 메서드를 호출하는 것이 자연스러울 것 같으나 한쪽 작업에는 `compute`를 호출하는 것이 효율적이다.
    - 두 서브테스크이 한 테스크에는 같은 스레드를 재사용할 수 있어 pool에서 불필요한 테스크를 할당하는 오버헤드를 피할 수 있다.
- 포크/조인 프레임워크를 이용하는 병렬 계산은 디버깅이 어렵다
    - 포크/조인 프레임워크에서는 fork라 불리는 다른 스레드에서 compute를 호출하므로 스택 트레이스가 도움되지 않는다.
- 멀티코어에 포크/조인 프레임워크를 사용하는 것이 순차 처리보다 무조건 빠르지는 않다.
    - 병렬 처리로 성능 개선하려면 테스크를 여러 독립적인 서브테스크로 분할할 수 있어야 한다.
    - 각 서브테스크의 실행시간은 새로운 테스크를 포킹하는데 드는 시간보다 길어야한다.
    - JOT 컴파일어에 의해 최적화되려면 몇 차례의 준비 과정 및 실행 과정을 거쳐야 한다.
    - 따라서 성능 측정은 여러번 프로그램을 실행한 결과를 측정해야한다.
- 작업 훔치기
    
    ---
    
    코어 개수와 관계없이 적절한 크기로 분할된 많은 테스크를 포킹하는 것은 바람직하다.
    
    이론적으로는 코어 개수만큼 병렬화된 테스크로 작업부하를 분할하면 모든 CPU 코어에서 이를 실행할 것이고, 크기가 같은 각각의 테스크는 같은 시간에 종료될 것 같지만 현실에서는 작왑완료 시간이 제각각으로 달라질 수 있다.
    
    포크/조인 프레임워크는 작업 훔치기 기법으로 해결한다.
    
    이는 `ForkJoinPool`의 모든 스레드를 공정하게 분할한다. 각각의 스레드는 할당된 테스크를 포함하는 이중 연결 리스트를 참조하며 작업이 완료될 때마다 큐의 헤드에서 다른 테스크를 가져와 작업을 처리한다.
    
    이때 한 스레드는 다른 스레드보다 할당된 테스크를 빨리 처리할 수 있다. 이 때 작업이 완료된 스레드는 유휴 상태로 바뀌는게 아닌 다른 스레드 큐의 꼬리에서 작업을 훔쳐온다.
    
    따라서 테스크 크기를 작게 나누어야 작업자 스레드 간의 작업부하를 비슷한 수준으로 유지 가능하다.
    
    ![Untitled](CHAPTER%207%20%E1%84%87%E1%85%A7%E1%86%BC%E1%84%85%E1%85%A7%E1%86%AF%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%8E%E1%85%A5%E1%84%85%E1%85%B5%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%A5%E1%86%BC%E1%84%82%E1%85%B3%E1%86%BC%2098f4997467c2463d867aa0d86058c4f8/Untitled%201.png)
    

## Spliterator로 스트림 쪼개기

Java 8은 병렬 작업에 특화되어 있는 `Spliterator`라는 새로운 인터페이스를 제공한다.

```java
public interface Spliterator<T> {
    
    boolean tryAdvance(Consumer<? super T> action);
    Spliterator<T> trySplit();
    long estimateSize();
    int characteristics();
}
```

## 분할 과정

![https://user-images.githubusercontent.com/61372486/128547827-033a007a-9a64-421f-9785-1cda9062a0d6.png](https://user-images.githubusercontent.com/61372486/128547827-033a007a-9a64-421f-9785-1cda9062a0d6.png)

- 첫 번째 `Spliterator`에 `trySplit`를 호출하면 두 번째 `Spliterator`가 생성된다.
- 두 개의 `Spliterator`에 다시 `trySplit`를 호출하면 네 개의 `Spliterator`가 생성된다.
- `trySplit`의 결과가 null이 될 때까지 반복한다.
- 재귀 분할 과정이 종료된다.

---

`characteristics`는 `Spliterator` 자체의 특성 직합을 포함하는 int를 반환한다.

[Spliterator 특성](CHAPTER%207%20%E1%84%87%E1%85%A7%E1%86%BC%E1%84%85%E1%85%A7%E1%86%AF%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%8E%E1%85%A5%E1%84%85%E1%85%B5%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%A5%E1%86%BC%E1%84%82%E1%85%B3%E1%86%BC%2098f4997467c2463d867aa0d86058c4f8/Spliterator%20%E1%84%90%E1%85%B3%E1%86%A8%E1%84%89%E1%85%A5%E1%86%BC%20455519f2b1c641d9be76cce50545e642.csv)

- **커스텀 Spliterator 예제(문자열의 단어 수를 계산하는 메서드 구현)**
    
    ```java
    class WordCounterSpliterator implements Spliterator<Character> {
        private final String string;
        private int currentChar = 0;
        public WordCounterSpliterator(String string) {
            this.string = string;
        }
        @Override
        public boolean tryAdvance(Consumer<? super Character> action) {
            // 현재 문자를 소비한다.
            action.accept(string.charAt(currentChar++));
            //소비할 문자가 남아있으면 true를 반환한다.
            return currentChar < string.length();
        }
        @Override
        public Spliterator<Character> trySplit() {
            int currentSize = string.length() - currentChar;
            // 파싱할 문자열을 순차 처리할 수 있을 만큼 충분히 작아졌음을 알리는 null을 반환
            if (currentSize < 10) {
                return null;
            }
            for (int splitPos = currentSize / 2 + currentChar;
                     // 파싱할 문자열의 중간을 분할 위치로 설정
                     splitPos < string.length(); splitPos++) {
                // 다음 공백이 나올 때까지 분할 위치를 뒤로 이동
                if (Character.isWhitespace(string.charAt(splitPos))) {
                    // 처음부터 분할 위치까지 문자열을 파싱할 새로운 WordCounterSpliterator 생성 
                    Spliterator<Character> spliterator =
                       new WordCounterSpliterator(string.substring(currentChar,
                                                                   splitPos));
                    // 이 WordCounterSpliterator의 시작 위치를 분할 위치로 설정
                    currentChar = splitPos;
                    // 공백을 찾았고 문자열을 분리했으므로 루프를 종료
                    return spliterator;
                }
            }
            return null;
        }
        @Override
        public long estimateSize() {
            return string.length() - currentChar;
        }
        @Override
        public int characteristics() {
            return ORDERED + SIZED + SUBSIZED + NON-NULL + IMMUTABLE;
        }
    }
    
    public class WordCounter {
        private final int counter;
        private final boolean lastSpace;
    
        public WordCounter(int counter, boolean lastSpace) {
            this.counter = counter;
            this.lastSpace = lastSpace;
        }
    
        public WordCounter accumulator(Character c) {
            if (Character.isWhitespace(c)) {
                return lastSpace ? this : new WordCounter(counter, true);
            } else {
                return lastSpace ? new WordCounter(counter + 1, false) : this;
            }
        }
    
        public WordCounter combine(WordCounter wordCounter) {
            return new WordCounter(counter + wordCounter.counter, wordCounter.lastSpace);
        }
        
        public int getCounter() {
            return counter;
        }
    }
    
    private static int countWords(Stream<Character> stream) {
            WordCounter wordCounter = stream.reduce(new WordCounter(0, true),
                    WordCounter::accumulator,
                    WordCounter::combine);
    
            return wordCounter.getCounter();
        }
    
    //활용 방법
    Spliterator<Character> spliterator = new WordCounterSpliterator(SETENCE);
    Stream<Chararacter> stream = StreamSupport.stream(spliterator, true);
    
    countWords(stream);
    ```