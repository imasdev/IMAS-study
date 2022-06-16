# CHAPTER 13 디폴트 메서드

발표자: 권윤옥
스터디일정: 2022/06/02 오후 6:00
주차별: 7주차
참여자: 권윤옥, 남채린, 이수환, 이승민, 장현준

<aside>
😎 **이 장의 내용**
- 디폴트 메서드란 무엇인가?
- 진화하는 API가 호환성을 유지하는 방법
- 디폴트 메서드의 활용 패턴
- 해결 규칙

</aside>

인터페이스를 구현하는 클래스는 인터페이스에서 정의하는 모든 메서드 구현을 제공하거나 아니면 슈퍼클래스의 구현을 상속받아야 한다. 근데 인터페이스가 바뀐다면 어떻게 될까?

인터페이스를 상속한 모든 클래스를 고쳐야 한다?

![Untitled](CHAPTER%2013%20%E1%84%83%E1%85%B5%E1%84%91%E1%85%A9%E1%86%AF%E1%84%90%E1%85%B3%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%206bec6dac5d0d48ffa8e28210b3e176ce/Untitled.png)

하지만 자바 8에서는 이 문제를 해결할 방법을 제공한다! 바로 기본 구현이 된 메서드를 인터페이스가 포함할 수 있게 된 것

디폴트 메서드를 이용하면 아래의 그림과 같이 자바 API의 호환성을 유지하면서 인터페이스를 변경할 수 있음

![KakaoTalk_Photo_2022-06-01-21-22-19.jpeg](CHAPTER%2013%20%E1%84%83%E1%85%B5%E1%84%91%E1%85%A9%E1%86%AF%E1%84%90%E1%85%B3%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%206bec6dac5d0d48ffa8e28210b3e176ce/KakaoTalk_Photo_2022-06-01-21-22-19.jpeg)

디폴트 메서드를 이용하면 인터페이스의 기본 구현을 그대로 상속하기 때문

# 13.1 변화하는 API(디폴트 메서드 등장 배경)

다음과 같은 그리기 라이브러리를 설계하여 배포했다고 가정하자

```java
public interface Resizable
{
    int getWidth();
    int getHeight();
    void setWidth(int width);
    void setHeight(int height);
    void setAbsoluteSize(int width, int height);
}

public class Ellipse implements Resizable
{
    private int width;
    private int height;

    @Override
    public int getWidth()
    {
        return width;
    }

    @Override
    public int getHeight()
    {
        return height;
    }

    @Override
    public void setWidth(int width)
    {
        this.width = width;
    }

    @Override
    public void setHeight(int height)
    {
        this.height = height;
    }

    @Override
    public void setAbsoluteSize(int width, int height)
    {
        this.width = width;
        this.height = height;
    }
}

public class Game
{
    public static void main(String[] args)
    {
        List<Resizable> resizableShapes = Arrays.asList(new Square(), new Rectangle(), new Ellipse());
        Utils.paint(resizableShapes);
    }
}

public class Utils
{
    public static void paint(List<Resizable> l)
    {
        l.forEach(r -> {
            r.setAbsoluteSize(42, 42);
            r.draw();
        });
    }
}
```

얼마 뒤에 다음과 같이 Resizable을 개선했다고 한다면 어떤 문제가 발생하는가

```java
public interface Resizable
{
    int getWidth();
    int getHeight();
    void setWidth(int width);
    void setHeight(int height);
    void setAbsoluteSize(int width, int height);
    void setRelativeSize(int wFactor, int hFactor); // 새로 추가된 메서드
}
```

### 문제점

1. `Resizable`을 구현하는 모든 클래스는 `setRelativeSize` 메서드를 구현해야함
2. 사용자가 Ellipse를 포함하는 전체 애플리케이션을 재빌드할 때 컴파일 에러가 발생

이러한 문제는 디폴트 메서드를 사용하면 해결할 수 있다.

# 13.2 디폴트 메서드란 무엇인가?

디폴트 메서드: 인터페이스에 존재하는 구현 메서드, 메서드 앞에 `default` 예약어가 붙으며 메서드 바디(구현부 `{ }`) 가 있어야 함.

인터페이스를 구현하는 클래스에서 구현하지 않은 메서드는 **인터페이스 자체에서 기본으로 제공**한다고 해서 디폴트 메서드라고 부름

```java
public interface Sized{
		int size();
		default boolean isEmpty(){
				return size() == 0;
		}
}
```

디폴트 메서드를 이용해서 위에서 언급한 `setRelativeSize` 메서드 구현을 제공한다면 **호환성을 유지**하면서 라이브러리를 고칠 수 있다.

```java
default void setRelativeSize(int wFactor, int hFactor){
		setAbsoluteSize(getWidth() / wFactor, getHeight() / hFactor);
}
```

### 자바 8에 활용된 디폴트 메서드

![Collection 인터페이스의 stream 메서드](CHAPTER%2013%20%E1%84%83%E1%85%B5%E1%84%91%E1%85%A9%E1%86%AF%E1%84%90%E1%85%B3%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%206bec6dac5d0d48ffa8e28210b3e176ce/Untitled%201.png)

Collection 인터페이스의 stream 메서드

![List 인터페이스의 sort 메서드](CHAPTER%2013%20%E1%84%83%E1%85%B5%E1%84%91%E1%85%A9%E1%86%AF%E1%84%90%E1%85%B3%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%206bec6dac5d0d48ffa8e28210b3e176ce/Untitled%202.png)

List 인터페이스의 sort 메서드

### 추상 클래스와 자바 8의 인터페이스

- 🤔  **추상 클래스와 자바 8의 인터페이스는 무엇이 다를까?**
    
    <aside>
    💡 1. 클래스는 하나의 추상 클래스만 상속받을 수 있지만 인터페이스를 여러 개 구현할 수 있다.
    2. 추상 클래스는 인스턴스 변수(필드)로 공통 상태를 가질 수 있다. 하지만 인터페이스는 인스턴스 변수를 가질 수 없다.
    
    </aside>
    

# 13.3 디폴트 메서드 활용 패턴

디폴트 메서드를 이용하는 두 가지 방식, 선택형 메서드와 동작 다중 상속을 알아보자.

### 선택형 메서드

자바 8 이전에 Iterator에서 remove() 메서드가 선언되어 있었지만 잘 사용하지 않아서 인터페이스를 구현하는 클래스에서 빈 구현(메서드의 내용을 비워둠)을 제공했다고 한다. 

하지만 다음과 같이 디폴트 메서드를 통해 기본 구현을 제공하므로써 Iterator 인터페이스를 구현하는 클래스는 빈 remove 메서드를 구현할 필요가 없어졌고 불필요한 코드를 줄일 수 있게 되었다고 한다. 

```java
public interface Iterator<E> {
    boolean hasNext();
    E next();
    default void remove() {
        throw new UnsupportedOperationException("remove");
    }
}
```

### 동작 다중 상속

디폴트 메서드를 이용하면 기존에는 불가능했던 동작 다중 상속 기능도 구현할 수 있다.

클래스는 한 개의 다른 클래스만 상속할 수 있지만 인터페이스는 여러 개 구현할 수 있기 때문

![자바 API에 정의된 ArrayList 클래스](CHAPTER%2013%20%E1%84%83%E1%85%B5%E1%84%91%E1%85%A9%E1%86%AF%E1%84%90%E1%85%B3%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%206bec6dac5d0d48ffa8e28210b3e176ce/Untitled%203.png)

자바 API에 정의된 ArrayList 클래스

자바 8에서는 인터페이스가 구현을 포함할 수 있으므로 클래스는 여러 인터페이스에서 동작(구현 코드)를 상속받을 수 있다.

게임에 다양한 특성을 갖는 여러 모양을 정의 한다고 가정하자. 

- 어떤 모양은 회전할 수 없지만 크기는 조절할 수 있음.
- 어떤 모양은 회전할 수 있고 움직일 수 있지만 크기는 조절할 수 없음

```java
public interface Rotatable
{
    void setRotationAngle(int angleInDegrees);
    int getRoationAngle();
    default void rotateBy(int angleInDegrees)
    {
        setRotationAngle( (getRoationAngle() + angleInDegrees) % 360 );
    }
}

public interface Moveable
{
    int getX();
    int getY();
    void setX(int x);
    void setY(int y);

    default void moveHorizontally(int distance)
    {
        setX(getX() + distance);
    }

    default void moveVertically(int distance)
    {
        setY(getY() + distance);
    }
}

public interface Resizable
{
    int getWidth();
    int getHeight();
    void setWidth(int width);
    void setHeight(int height);
    void setAbsoluteSize(int width, int height);

    default void setRelativeSize(int wFactor, int hFactor)
    {
        setAbsoluteSize(getWidth() / wFactor, getHeight() / hFactor);
    }
}
```

이 인터페이스들을 조합해서 게임에 필요한 다양한 클래스를 구현할 수 있다.

```java
public class Monster implements Rotatable, Moveable, Resizable
{
    @Override
    public int getX(){        return 0;    }

    @Override
    public int getY(){        return 0;    }

    @Override
    public void setX(int x){    }

    @Override
    public void setY(int y){    }

    @Override
    public int getWidth(){        return 0;    }

    @Override
    public int getHeight(){        return 0;    }

    @Override
    public void setWidth(int width){    }

    @Override
    public void setHeight(int height){    }

    @Override
    public void setAbsoluteSize(int width, int height){    }

    @Override
    public void setRotationAngle(int angleInDegrees){    }

    @Override
    public int getRoationAngle(){        return 0;    }
}
```

```java
public static void main(String[] args)
{
    Monster m = new Monster();
    m.rotateBy(180);      // Rotatable의 rotateBy 호출
    m.moveVertically(10); // Moveable의 moveVertically 호출
}
```

```java
public class Sun implements Moveable, Rotatable {
...
}
```

여기서 `moveVertically`의 구현을 더 효율적으로 고쳐야 한다고 하면 `Moveable` 인터페이스를 직접 고칠 수 있기 때문에 `Moveable`을 구현하는 모든 클래스도 자동으로 변경한 코드를 상속받을 수 있다는 장점이 생김! (`moveVertically`를 재정의하지 않았다는 가정하에)

# 13.4 해석 규칙

여러 인터페이스를 구현하는 클래스가 있을 때 같은 시그니처를 가진 디폴트 메서드가 두 개 이상있다면 어떤 인터페이스의 디폴트 메서드를 사용하게 될 것인가?

### 세 가지 규칙

1. **클래스가 항상 이긴다.** 
클래스나 슈퍼클래스에서 정의한 메서드가 디폴트 메서드보다 우선권을 갖는다.
2. **1번 규칙 이외에 상황에서는 서브 인터페이스가 이긴다.**
상속관계를 갖는 인터페이스에서 같은 시그니처를 갖는 메서드를 정의할 때는 서브인터페이스가 이긴다. 즉, B가 A를 상속받는다면 B가 A를 이긴다.
3. 여전히 디폴트 메서드의 우선순위가 결정되지 않았다면 여러 인터페이스를 상속받는 클래스가 명시적으로 디폴트 메서드를 오버라이드하고 호출해야 한다.

### 디폴트 메서드를 제공하는 서브 인터페이스가 이긴다

![KakaoTalk_Photo_2022-06-02-12-36-05 003.jpeg](CHAPTER%2013%20%E1%84%83%E1%85%B5%E1%84%91%E1%85%A9%E1%86%AF%E1%84%90%E1%85%B3%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%206bec6dac5d0d48ffa8e28210b3e176ce/KakaoTalk_Photo_2022-06-02-12-36-05_003.jpeg)

```java
public interface A{
		default void hello(){
				System.out.println("Hello from A");
		}
}

public interface B extends A{
		default void hello(){
				System.out.println("Hello from B");
		}
}

public class C implements B, A{
		public static void main(String[] args){
				new C().hello();     // -> 무엇이 출력될까??
		}
}
```

- 컴파일 결과
    
    ![Untitled](CHAPTER%2013%20%E1%84%83%E1%85%B5%E1%84%91%E1%85%A9%E1%86%AF%E1%84%90%E1%85%B3%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%206bec6dac5d0d48ffa8e28210b3e176ce/Untitled%204.png)
    

다음과 같이 A라는 인터페이스를 상속받는 D가 추가되어, C가 D를 상속받았을 때는 어떻게 될까?

![KakaoTalk_Photo_2022-06-02-12-36-05 002.jpeg](CHAPTER%2013%20%E1%84%83%E1%85%B5%E1%84%91%E1%85%A9%E1%86%AF%E1%84%90%E1%85%B3%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%206bec6dac5d0d48ffa8e28210b3e176ce/KakaoTalk_Photo_2022-06-02-12-36-05_002.jpeg)

```java
public class D implements A
{
}

public class C extends D implements B, A
{
    public static void main(String[] args)
    {
        new C().hello();
    }
}
```

- 컴파일 결과
    
    ![Untitled](CHAPTER%2013%20%E1%84%83%E1%85%B5%E1%84%91%E1%85%A9%E1%86%AF%E1%84%90%E1%85%B3%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%206bec6dac5d0d48ffa8e28210b3e176ce/Untitled%205.png)
    

D가 다음과 같이 바뀌었을 때는 무엇이 출력될까?

```java
public class D implements A
{
    public void hello()
    {
        System.out.println("Hello from D");
    }
}
```

- 컴파일 결과
    
    ![Untitled](CHAPTER%2013%20%E1%84%83%E1%85%B5%E1%84%91%E1%85%A9%E1%86%AF%E1%84%90%E1%85%B3%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%206bec6dac5d0d48ffa8e28210b3e176ce/Untitled%206.png)
    

### 충돌 그리고 명시적인 문제 해결

B가 A를 상속받지 않는 상황으로 변경해서 가정했을 때는 어떻게 될것인가

![KakaoTalk_Photo_2022-06-02-12-36-05 004.jpeg](CHAPTER%2013%20%E1%84%83%E1%85%B5%E1%84%91%E1%85%A9%E1%86%AF%E1%84%90%E1%85%B3%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%206bec6dac5d0d48ffa8e28210b3e176ce/KakaoTalk_Photo_2022-06-02-12-36-05_004.jpeg)

```java
public interface A
{
    default void hello()
    {
        System.out.println("Hello from A");
    }
}

public interface B
{
    default void hello()
    {
        System.out.println("Hello from B");
    }
}

public class C implements B, A
{
    public static void main(String[] args)
    {
        new C().hello();
    }
}
```

다음과 같은 컴파일 에러가 발생한다

![Untitled](CHAPTER%2013%20%E1%84%83%E1%85%B5%E1%84%91%E1%85%A9%E1%86%AF%E1%84%90%E1%85%B3%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%206bec6dac5d0d48ffa8e28210b3e176ce/Untitled%207.png)

왜냐하면 컴파일러는 어떤 메서드를 선택해서 호출해야 하는지 알 수 없기 때문이다.

따라서, 개발자가 호출할 메서드를 명시적으로 지정해줘야 하며 자바 8에서 제공하는 다음과 같은 문법을 사용하면 된다. `X.super.m();`

```java
public class C implements B, A
{
    public void hello()
    {
        B.super.hello();
    }
}
```

### 다이아몬드 문제

D는 B와 C중 누구의 디폴트 메서드 정의를 상속받을까?

![KakaoTalk_Photo_2022-06-02-12-36-05 005.jpeg](CHAPTER%2013%20%E1%84%83%E1%85%B5%E1%84%91%E1%85%A9%E1%86%AF%E1%84%90%E1%85%B3%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A5%E1%84%83%E1%85%B3%206bec6dac5d0d48ffa8e28210b3e176ce/KakaoTalk_Photo_2022-06-02-12-36-05_005.jpeg)

```java
public interface A
{
    default void hello()
    {
        System.out.println("Hello from A");
    }
}
public interface B extends A {
		default void hello()
    {
        System.out.println("Hello from A");
    }
}
public interface C extends A {
		void hello();
}

public class D implements B,C
{
    public static void main(String[] args)
    {
        new D().hello();
    }
}
```

**Quiz !!**

- B에도 같은 시그니처의 디폴트 메서드가 있다면?
- B와 C가 모두 디폴트 메서드 `hello()`를 정의한다면?
- C에 추상 메서드 `hello()`가 추가된다면?