---
layout: post
title:  "URL 링크로 파일 저장하기."
date:   2018-11-27 18:12:00 +0900
categories: Java
comments: true
tags: [Java]
---

---
### URL 링크로 파일 저장하기.


```java
InputStream in = new URL(파일이 있는 URL 링크).openStream(); //ex: http://jmlim.github.io/public/img/3632773.png
Files.copy(in, Paths.get(파일을 저장할 링크)); //ex : C:/File/Images/파일명.jpg
```


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---