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

> **[2026년 추가]** 이 방식은 부동소수점 오차 때문에 **10의 거듭제곱수(10, 100, 1000...)에서 가끔 틀린 값을 낼 수 있다고 알려져 있다.** `Math.log10(1000)`이 이상적으로는 정확히 `3.0`이어야 하지만, 부동소수점 연산 특성상 환경에 따라 `2.9999999999999996` 같은 값이 나올 수 있고, 이 경우 `+1` 후 `(int)`로 잘라내는 과정에서 자릿수가 1 작게 나온다. 자릿수 세는 게 목적이라면 부동소수점 연산을 아예 쓰지 않는 다음 방법이 더 안전하다.
> ```java
> int length = String.valueOf(Math.abs(a)).length(); // 음수는 절대값으로 처리
> ```





[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
