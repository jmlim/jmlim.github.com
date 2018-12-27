---
layout: post
title:  "숫자형의 길이 구하기."
date:   2018-12-06 18:30:00 +0900
categories: Java
comments: true
tags: [Java]
---

---


```java
String a = "abced";
System.out.println("길이>>"+a.length);
```

* 위와 같은 코드에서는 아래의 결과가 나올것이다.

>길이: 5

* String 형의 경우 length 함수를 지원하지만, int형은 길이 함수가 없다. 이럴경우 Math 함수를 사용하면 자리수를 구할 수 있다.

```java
int a = 3291;
int length = (int)(Math.log10(a)+1);
System.out.println("길이 : " + a);
```

>길이 : 4



[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
