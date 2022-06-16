# CHAPTER 12 새로운 날짜와 시간 API

발표자: 이수환
스터디일정: 2022/06/02 오후 6:00
주차별: 7주차
참여자: 권윤옥, 남채린, 이수환, 이승민, 장현준

## 1. 새로운 날짜와 시간 API 생성 배경

1-1. java 1.0 에서는 java.util.Date 클래스 하나로 날짜와 시간 관련 기능을 모두 제공했다.

날짜를 의미하는 Date 라는 뜻과 달리 특정 시점을 밀리초단위로 표현한다.

게다가 1900년을 기준으로 하고, 0부터 시작하는 달 인덱스등 모호한 설계가 문제였었다.

```java
Date date = new Date();
System.out.println(date.getTime()); 
// 1654145972877  -> 1900년 1월1월 0시0분0초 기준 현재까지의 밀리초
System.out.println(date); //Thu Jun 02 13:59:32 KST 2022

Date date2 = new Date(2022,6,2); // 년,월,일을 그대로 넣으면..
System.out.println(date2); //Sun Jul 02 00:00:00 KST 3922 => 3922년 7월 2일로 나옴.

Date date3 = new Date(122,5,2); // 이렇게 넣어줘야
System.out.println(date3); //Tue Jun 02 00:00:00 KST 2022 => 원하는 날짜로 나옴. 불편!
```

1-2 .자바 1.1 에서는 java.util.Calendar 클래스가 대안으로 제공됨.

안타깝게도 Calendar 클래스도 설계문제가 존재했음.

1900년도부터 시작하는 오프셋은 없앴지만, 여전히 0부터 시작하는 달 인덱스. 

그리고 Date,Calendar 두 가지 클래스를 쓰게 되면서

개발자들에게 혼란이 가중됨. 거기다 DateFormat 같은 기능은 Date 클래스에만 작동했다.

DateFormat 에도 멀티스레드에 안전하지 않다는 문제가 있었다. 두 스레드가 동시에 하나의 포매터로 

날짜를 파싱할때 예기치 못한 문제가 생길 수 있다. (?)

그리고 Date와 Calendar는 모두 가변 클래스 (객체의 날짜정보가 변경가능) 라는 점 때문에 

유지보수도 어려워짐. 그래서 이러한 문제를 해결하기 위해 java8에서는 java.time 패키지가 추가되었다.

## 2. LocalDate, LocalTime, LocalDateTime

2-1. LocalDate 클래스는 시간을 제외한 날짜를 표현하는 불변객체이다. 

```java
LocalDate localDate = LocalDate.of(2019,12,25); //of 메소드를 사용해 특정 날짜 지정 가능
System.out.println(localDate); // 2019-12-25
System.out.println(localDate.getYear()); // 2019
System.out.println(localDate.getMonth()); // DECEMBER
System.out.println(localDate.getMonthValue()); // 12
System.out.println(localDate.getDayOfMonth()); // 25
System.out.println(localDate.getDayOfWeek()); // WEDNESDAY
//Date와 Calerdar의 단점인 1900년 기준으로 연도 입력방식 과 원하는 월보다 -1해서 입력해야 하는 불편함이 해소됨.

LocalDate localDateToday = LocalDate.now(); // now()를 사용해 현재날짜 구함.
System.out.println(localDateToday); // 2022-06-02
```

2-2. 시간은 LocalTime 클래스로 표현할 수 있다. 

```java
LocalTime localTime = LocalTime.of(13,45,20);
System.out.println(localTime); // 13:45:20
System.out.println(localTime.getHour()); // 13
System.out.println(localTime.getMinute()); // 45
System.out.println(localTime.getSecond()); // 20

LocalTime localTimeNow = LocalTime.now();
System.out.println(localTimeNow); // 22:31:30.974735300  // LocalTime 은 나노초(10억분의 1초)까지 표현 가능.
System.out.println(localTimeNow.getHour()); // 22
System.out.println(localTimeNow.getMinute()); // 31
System.out.println(localTimeNow.getSecond()); // 30
```

2-3. LocalDateTime 은 LocalDate와 LocalTime 을 합친 복합클래스이다. 날짜와 시간 모두 표현할 수 있다. 

```java
LocalDateTime nowDateTime = LocalDateTime.now();
System.out.println(nowDateTime); // 2022-06-02T14:26:45.473175

LocalDateTime localDateTime = LocalDateTime.of(2017, Month.APRIL,20,15,30,10);
System.out.println(localDateTime); // 2017-04-20T15:30:10

LocalDate localDate1 = localDate.now();
LocalTime localTime1 = LocalTime.now();
LocalDateTime localDateTime2 = LocalDateTime.of(localDate1, localTime1); //LocalDate 객체와 LocalTime 객체를 결합해서 LocalDateTime 을 만들수도있다.
System.out.println(localDateTime2); // 2022-06-02T14:26:45.473301

//LocalDateTime 간 비교
System.out.println(nowDateTime.compareTo(localDateTime)); // 5  (양수 : 현재날짜가 비교날짜보다 미래이다.  )
System.out.println(nowDateTime.compareTo(localDateTime2)); // -1 (음수 : 현재날짜가 비교날짜보다 과거이다. )
```

## 3. Duration 과 Period

3-1. Duration과 Period 를 사용하여 날짜,시간 클래스들 간의 날짜,시간 차이값을 구할 수 있다. 

Duration 은 시간(일,시,분,초,나노초)간격 을 구할 때 사용하고, 

Period 는 날짜(연,월,일 )간격을 구할 때 사용한다.

```java
LocalDateTime nowDateTime1 = LocalDateTime.of(2022, Month.APRIL,20,6,0,0);
LocalDateTime nowDateTime2 = LocalDateTime.of(2022, Month.APRIL,21,8,20,30);

//duration 을 통해 두시간 사이의 간격을 구할 수 있다. (일수 차이~나노초 차이)
Duration d1 = Duration.between(nowDateTime1, nowDateTime2);
System.out.println(d1); //PT26H20M30S
System.out.println(d1.toDays()); // 1
System.out.println(d1.toHours()); // 26
System.out.println(d1.toMinutes()); //1580
System.out.println(d1.toSeconds()); //94830 초
System.out.println(d1.getNano()); // 0 ??  > 시간차이가 1초를 초과하면 나노초는 표기되지 않음.

LocalDateTime nowDateTime3 = LocalDateTime.of(2022, Month.APRIL,20,06,00,00, 999999999);
Duration d2 = Duration.between(nowDateTime1, nowDateTime3);
System.out.println(d2.getNano()); // 999999999 (nano 초는 0 ~ 999,999,999 나노초까지 표현가능)

//period 는 날짜간격(연, 월, 일)을 구할때 사용한다.
Period period = Period.between(LocalDate.of(2021,03,20), LocalDate.of(2022,06,01));
System.out.println(period); //P1Y2M12D
System.out.println(period.getYears()); // 1
System.out.println(period.getMonths()); // 2
System.out.println(period.getDays()); // 12
```

## 4. 날짜 조정, 변경

4-1. with, plus, minus 등의 메소드를 사용하여 날짜나 시간을 변경 할 수있다. 

모든 메소드는 기존객체의 날짜를 바꾸진 않는다. (java.time 시간객체는 불변객체)

```java
//절대적인 방식으로 LocalDate 속성 바꾸기 with 사용
LocalDate dt1 = LocalDate.of(2022,06,01);
LocalDate dt2 = dt1.withYear(2023);
LocalDate dt3 = dt2.withMonth(8);
LocalDate dt4 = dt3.withDayOfMonth(25);
System.out.println(dt4); // 2023-08-25

//상대적인 방식으로 LocalDate 속성 바꾸기
LocalDate dt5 = LocalDate.of(2022,04,01);
LocalDate dt6 = dt5.plusWeeks(1);
LocalDate dt7 = dt6.minusYears(2);
LocalDate dt8 = dt7.plusMonths(3);
System.out.println(dt8); // 2020-07-08

//LocalDateTime 도 동일함.
LocalDateTime localDateTime = LocalDateTime.now();
System.out.println(localDateTime); // 2022-06-02T15:12:15.678334
LocalDateTime localDateTime1 = localDateTime.plusHours(3);
System.out.println(localDateTime1); // 2022-06-02T18:12:15.678334
```

- **QUIZ**

```java
LocalDateTime quizTime = LocalDateTime.of(2022,06,02,18,00,00);
quizTime.plusYears(1);
quizTime = quizTime.minusMonths(2);
quizTime = quizTime.plusDays(5);

System.out.println(quizTime.format(DateTimeFormatter.ISO_DATE_TIME));
//???
```

- 정답 확인
    
     `2022-04-07T18:00:00`
    

4-2. TemporalAdjusters 사용하기

지금까지 사용한 날짜 조정 기능은 비교적 간단한 편이고, 다음주 일요일, 돌아오는 평일, 어떤달의 마지막날 등

좀더 복잡한 날짜기능이 필요 한 경우 with메소드에 더 다양한 동작을 제공하는 TemporalAjusters를 전달 

하는 방법을 사용한다.

```java
LocalDate date1 = LocalDate.of(2022,06,01);

LocalDate date2 = date1.with(TemporalAdjusters.firstDayOfMonth()); // 이번달 첫일
System.out.println(date2); // 2022-06-01
LocalDate date3 = date1.with(TemporalAdjusters.lastDayOfMonth()); // 이번달 말일
System.out.println(date3); // 2022-06-30 // 이후에 나올 formater 를 사용하여 이번달 첫일, 말일이 무슨 요일인지 확인기능하다.

LocalDate date4 = date1.with(TemporalAdjusters.firstInMonth(DayOfWeek.SUNDAY)); //이번달 첫번째 일요일
System.out.println(date4); // 2022-06-05
LocalDate date5 = date1.with(TemporalAdjusters.lastInMonth(DayOfWeek.SUNDAY)); //이번달 마지막 일요일
System.out.println(date5); // 2022-06-30
LocalDate date6 = date1.with(TemporalAdjusters.dayOfWeekInMonth(3,DayOfWeek.FRIDAY)); //이번달 3번째 금요일
System.out.println(date6); // 2022-06-17
```

## 5. 날짜 포매팅, 파싱

5-1. 날짜 시간 관련 작업에서 포매팅과 파싱은 서로 떨어질 수 없는 관계다.

java8 에서는 포매팅과 파싱전용 패키지인 java.time.format 이 추가되었다.

이 패키지에서 가장 중요한 클래스는 DateTimeFormatter다. DateTimeFormatter를 이용해서 

날짜와 시간을 특정형식의 문자열로 만들 수 있다.

다음은 두개의 서로다른 포매터로 문자열을 만드는 예제이다.

```java
//DateTimeFormatter 에서 기본적으로 제공되는 포맷형식
LocalDate date = LocalDate.of(2022,06,01);
System.out.println(date.format(DateTimeFormatter.BASIC_ISO_DATE)); // 20220601
System.out.println(date.format(DateTimeFormatter.ISO_DATE)); // 2022-06-01

LocalDateTime time = LocalDateTime.of(2022,06,01,11,10,20);
System.out.println(time.format(DateTimeFormatter.ISO_LOCAL_TIME)); // 11:10:20
System.out.println(time.format(DateTimeFormatter.ISO_DATE_TIME)); // 2022-06-01T11:10:20

//날짜나 시간을 표현하는 문자열을 파싱해서 날짜 객체를 만들 수 있다.
System.out.println(LocalDate.parse("20220612", DateTimeFormatter.BASIC_ISO_DATE)); // 2022-06-01
System.out.println(LocalDate.parse("2022-06-12", DateTimeFormatter.ISO_DATE)); // 2022-06-01
System.out.println(LocalDateTime.parse("2022-06-01T11:10:20", DateTimeFormatter.ISO_DATE_TIME)); // 2022-06-01T11:10:20

//원하는 특정 패던으로 DateTimeFormatter 만들기
LocalDateTime time1 = LocalDateTime.of(2022,06,02,13,45,10);

DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy/MM/dd(E) a hh:mm:ss");
System.out.println(time1.format(formatter)); // 2022/06/02(목) 오후 01:45:10

DateTimeFormatter formatter24 = DateTimeFormatter.ofPattern("yyyy/MM/dd(E) HH:mm:ss");
System.out.println(time1.format(formatter24)); // 2022/06/02(목) 13:45:10
```

## 6. ZoneDateTime, ZoneId

6-1.지금까지 살펴본 모든 클래스에는 시간대와 관련된 정보가 없었다. 

기존의 java.util.TimeZone을 대체할 수 있는 java.tome.ZoneId 클래스가 새롭게 등장했다. 

```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy/MM/dd(E) a hh:mm:ss");

LocalDateTime time = LocalDateTime.now();
LocalDateTime timeRome = LocalDateTime.now(ZoneId.of("Europe/Rome"));

System.out.println(time.format(formatter)); // 2022/06/02(목) 오후 03:26:28
System.out.println(timeRome.format(formatter)); // 2022/06/02(목) 오전 08:26:28

//ZonedDateTime 은 LocalDateTime에 시간대, 시차 정보가 추가된 클래스
ZonedDateTime ztime = ZonedDateTime.now();
ZonedDateTime ztimeRome = ZonedDateTime.now(ZoneId.of("Europe/Rome"));

System.out.println(ztime); // 2022-06-02T13:52:25.675228+09:00[Asia/Seoul]
System.out.println(ztimeRome); // 2022-06-02T06:52:25.675258+02:00[Europe/Rome]
//+9:00, +2:00 은 시차기준인 그리니치시 대비 시차를 의미함.

System.out.println(ztime.getZone()); //Asia/Seoul
System.out.println(ztime.getOffset()); //+09:00
```

## 7. 마치며

- 자바 8 이전 버전에서 제공하는 기존의 [java.util.Date](http://java.util.Date) 클래스와 관련 클래스에서는 여러 불일치, 가변성, 기본값, 잘못된 이름 결정등의 설계 결함이 존재했다.
- 새로운 날짜와 시간 API 에서 날짜와 시간 객체는 모두 불변히다.
- 날짜와 시간 객체를 절대적인 방법과 상대적인 방법으로 처리할 수 있으며, 기존 인스턴스를 변경 하지 않도록 처리 결과로 새로운 인스턴스가 생성된다.
- TemporalAdjuster 를 이용하면 단순히 값을 바꾸는것 이상의 복잡한 동작을 수행 할 수 있다.
- 날짜와 시간 객체를 특정 포맷으로 출력하고, 파싱하는 포매터를 정의할 수 있다. 포매터는 스레드 안정성을 보장한다.
- 특정 지역/장소에 상대적인 시간대를 구할 수 있으며, 이 시간대를 날짜와 시간 객체에 적용하여 지역화 할 수 있다.