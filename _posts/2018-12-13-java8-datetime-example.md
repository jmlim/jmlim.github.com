---
layout: post
title:  "Java8 의 날짜 API 사용하기."
date:   2018-12-13 09:50:00 +0900
categories: Java
comments: true
tags: [Java, Java8, DateTime]
---

---

# Java8 의 날짜 API 사용하기.

Java 1.8 이전 버전의 SDK에서 날짜와 시간을 다루는 java.util.Date 클래스와 java.util.Calendar 클래스는 
사용하기가 불편하고 직관적이지 않으며 또한 java.util.Date와 SimpleDateFormatter는 Thread-Safe 하지 않아서 잠재적인 동시성 문제를 가지고 있다.
많이 늦었지만 JDK 8에서는 개선된 날짜와 시간 API가 제공된다.

기존 API
--
 - java.util.Date - 날짜와 시간, 기본 시간대를 사용하여 출력.
 - java.util.Calendar - 날짜와 시간, 날짜를 조작하는데 더 많은 메소드 제공.
 - java.text.SimpleDateFormat - 날짜와 달력을위한 형식 (날짜 -> 텍스트), 변환 (텍스트 -> 날짜).

JAVA8 에서 추가된 API
--
- java.time.LocalDate - 날짜(시간 포함하지 않음), 타임존 사용하지 않음.
- java.time.LocalTime - 시간(날짜 포함하지 않음), 타임존 사용하지 않음.
- java.time.LocalDateTime - 날짜 및 시간, 타임존 사용하지 않음.
- java.time.ZonedDateTime - 날짜 및 시간, 타임존 사용.
- java.time.DateTimeFormatter - java.time에 대한 형식 (날짜 -> 텍스트), 변환 (텍스트 -> 날짜)
- java.time.Duration - 시간을 초 단위 및 나노초 단위로 측정한다.
- java.time.Period - 시간을 년, 월, 일로 측정한다.
- java.time.TemporalAdjuster - 날짜를 조정한다.
 
사용해보기
--

예제코드 1) 
 - 컴퓨터의 현재 날짜 출력.

```java
import java.time.*;

public class WorkingWithLocalDate {
  public static void main(String[] args) {
      LocalDate localDate = LocalDate.now();
      System.out.println(localDate); 
  }
}
```

예제코드 2) 
 - 특정 일, 월 및 연도 지정하여 반환하고 싶을 때 LocalDate.of 메서드를 사용하거나 LocalDate.parse 메서드를 사용. 

```java

import java.time.*;
import java.time.format.DateTimeFormatter;

public class WorkingWithLocalDate {
    public static void main(String[] args) {
        //다음날짜로 지정 (2017-12-12)
        LocalDate localDate = LocalDate.of(2017, 12, 12);
        System.out.println(localDate);

        //다음날짜로 지정 (2017-12-05)
        localDate = LocalDate.parse("2018-12-05");
        System.out.println(localDate);
	}
}

```

출력 : 
> 2017-12-12<br>
2018-12-05

예제코드 3) 
 - LocalDate parse 사용시에 포멧 형식 변경하여 사용할 경우.


```java

import java.time.*;
import java.time.format.DateTimeFormatter;

public class WorkingWithLocalDate {
    public static void main(String[] args) {
		// LocalDate parse 사용 시 포멧 형식 변경해서 사용 시
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("d/MM/yyyy");
        String date = "18/09/2018";
        //convert String to LocalDate
        LocalDate localDate = LocalDate.parse(date, formatter);
        System.out.println(localDate);

        // LocalDateTime 사용 시 pattern 지정하여 출력하기.
        LocalDateTime now = LocalDateTime.now();
        System.out.println("Before : " + now);
        DateTimeFormatter formatter2 = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        String formatDateTime = now.format(formatter2);
        System.out.println("After : " + formatDateTime);
	}
}

```

출력 : 
> 2018-09-18<br>
Before : 2018-12-13T09:49:53.383<br>
After : 2018-12-13 09:49:53


예제코드 4) 
 - 날짜 시간 증감시키기 예제

```java

import java.time.*;
import java.time.format.DateTimeFormatter;

public class WorkingWithLocalDate2 {
    public static void main(String[] args) {
        //날짜 시간 증감.
        LocalDateTime localDateTime = LocalDateTime.of(2018, 12, 12, 0, 0, 0);
        LocalDateTime changedLocalDateTime = localDateTime.plusDays(1);  // 일
        System.out.println(changedLocalDateTime); // 출력 : 2018-12-13T00:00

        changedLocalDateTime = localDateTime.plusMonths(1);  // 월
        System.out.println(changedLocalDateTime); //출력 : 2019-01-12T00:00

        changedLocalDateTime = localDateTime.plusHours(1); // 시간
        System.out.println(changedLocalDateTime); //출력 : 2018-12-12T01:00

        changedLocalDateTime = localDateTime.plusWeeks(1); // 주
        System.out.println(changedLocalDateTime); //출력 : 2018-12-19T00:00

        changedLocalDateTime = localDateTime.minusYears(1);  // 년
        System.out.println(changedLocalDateTime); //출력 : 2017-12-12T00:00

        changedLocalDateTime = localDateTime.minusMinutes(1);  // 분
        System.out.println(changedLocalDateTime); //출력 : 2018-12-11T23:59
  }
}
```

출력 : 
 > 2018-12-13T00:00 <br/>
2019-01-12T00:00<br/>
2018-12-12T01:00<br/>
2018-12-19T00:00<br/>
2017-12-12T00:00<br/>
2018-12-11T23:59


예제코드 5) 
 - 특정 지역의 시간에 의존하지 않고 날짜와 시간을 표현하고 싶다면 ZonedDateTime 클래스를 사용하면 된다.

```java

import java.time.*;
import java.time.format.DateTimeFormatter;

public class WorkingWithLocalDate {
    public static void main(String[] args) {
      LocalDateTime dateTime = LocalDateTime.of(2018, Month.DECEMBER, 12, 9, 00, 00);
      // UTC+9는 서울시간
      ZonedDateTime seoulTime = dateTime.atZone(ZoneId.of("Asia/Seoul"));
      System.out.println("ZonedDateTime : " + seoulTime);
      
      //세계표준시.
      Instant instant = seoulTime.toInstant();
      System.out.println("Instant : " + instant);
	}
}

```

>서울시간이 UTC (세계 표준시) 보다 9시간이 빠르다. <br/>
만약 서울시간이 2018-12-12 09:00:00 일경우 표준시는 2018-12-12 00:00:00 가 된다.<br/>

출력 : 

>ZonedDateTime : 2018-12-12T09:00+09:00[Asia/Seoul]<br/>
Instant : 2018-12-12T00:00:00Z<br/>

예제코드 6) 
 - Period를 통해 두 "날짜" 사이의 간격을 알 수 있다.
 
```java

import java.time.*;
import java.time.format.DateTimeFormatter;

public class WorkingWithLocalDate {
    public static void main(String[] args) {
      LocalDate startDate = LocalDate.of(2010, 10, 18);
      LocalDate endDate = LocalDate.of(2018, 12, 12);
      
      Period period = Period.between(startDate, endDate);
      
      System.out.println(period.getYears() + " 년" + period.getMonths() + " 월" + period.getDays() + " 일");
	}
}

```

출력: 
 > 8 년1 월24 일

출처: 
 - http://starplatina.tistory.com/entry/Java-8-%EC%83%88%EB%A1%9C%EC%9A%B4-Date-Time-API
 - http://www.mkyong.com/tutorials/java-date-time-tutorials/


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

---

