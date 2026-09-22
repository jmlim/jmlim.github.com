---
layout: post
title:  "자바(JAVA) 형 변환(String과 Date)"
date:   2018-11-26 15:20:00 +0900
categories: Java
comments: true
tags: [Java, Date]
---

---


#### String to Date
```java
String from = "2013-04-08 10:10:10";
SimpleDateFormat transFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
Date to = transFormat.parse(from);
```

#### Date to String
```java
Date from = new Date();
SimpleDateFormat transFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
String to = transFormat.format(from);
```

> **[2026년 추가]** `SimpleDateFormat`은 **쓰레드 안전(thread-safe)하지 않다** — 하나의 인스턴스를 여러 쓰레드가 동시에 `parse()`/`format()`으로 호출하면 내부 캘린더 상태가 꼬여서 엉뚱한 날짜가 나오거나 예외가 발생할 수 있다. (매번 `new SimpleDateFormat(...)`으로 새로 만들어 쓰면 괜찮지만, static 필드로 캐싱해두고 재사용하면 위험하다.)
> Java 8부터는 이 문제 자체가 없는 `java.time` 패키지(`LocalDate`, `LocalDateTime`, `DateTimeFormatter`)가 표준이니, 새 코드를 짠다면 `SimpleDateFormat`/`Date` 대신 그쪽을 쓰는 걸 권장한다 — 이 블로그의 [Java8 의 날짜 API 사용하기](/java/2018/12/13/java8-datetime-example/) 글에 예제를 정리해뒀다.

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
