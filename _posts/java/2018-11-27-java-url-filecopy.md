---
layout: post
title:  "URL 링크로 파일 저장하기."
date:   2018-11-27 18:12:00 +0900
categories: Java
comments: true
tags: [Java]
---

---


```java
InputStream in = new URL(파일이 있는 URL 링크).openStream(); //ex: http://jmlim.github.io/public/img/imageofhanaumabay.jpg
Files.copy(in, Paths.get(파일을 저장할 링크)); //ex : C:/File/Images/파일명.jpg
```

> **[2026년 추가]** 위 코드는 두 가지를 신경 써야 한다.
> - 저장할 경로에 이미 파일이 있으면 `FileAlreadyExistsException`이 던져진다. 덮어써도 된다면 `StandardCopyOption.REPLACE_EXISTING`을 세 번째 인자로 넘겨야 한다.
> - `openStream()`으로 연 `InputStream`을 명시적으로 닫지 않고 있다 — 예외가 나면 스트림이 계속 열려있게 되므로, try-with-resources로 감싸는 게 안전하다.
> ```java
> try (InputStream in = new URL(파일이 있는 URL 링크).openStream()) {
>     Files.copy(in, Paths.get(파일을 저장할 링크), StandardCopyOption.REPLACE_EXISTING);
> }
> ```


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
