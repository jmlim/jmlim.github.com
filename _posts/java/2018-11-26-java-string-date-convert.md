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

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
