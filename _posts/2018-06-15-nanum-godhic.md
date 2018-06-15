---
layout: post
title:  "나눔고딕 웹 폰트 적용하기."
date:   2018-06-15 16:43:23 +0900
categories: etc
comments: true
tags: [font, Nanum Gothic]
---

---
### 나눔고딕 웹 폰트 적용하기

github 페이지 적용 후 한글폰트가 맘에 들지 않아 나눔글꼴을 적용했다.<br>
구글에서 제공하는 웹폰트를 사용하면 tistory, github 페이지 같은 곳에 어렵지 않게 글꼴을 적용할 수 있다.

[구글에서 제공하는 웹 폰트](https://fonts.google.com/)
1. 나눔고딕 항목 선택
<img src="{{ site.baseurl }}/public/post/fonts/font1.png" width="800px" height="400px"/>
2. Select this font 선택
<img src="{{ site.baseurl }}/public/post/fonts/font2.png" width="800px" height="400px"/>
3. 선택 후 보여지는 링크로 페이지에 적용
<img src="{{ site.baseurl }}/public/post/fonts/font3.png" width="800px" height="400px"/>


본인은 표준 import 방식으로 적용했다.<br>
(head 태그 안에 아래 링크 포함)

```html
<link href="https://fonts.googleapis.com/css?family=Nanum+Gothic" rel="stylesheet">
```


CSS 파일에 아래 스타일 적용
```css
font-family: 'Nanum Gothic', sans-serif;
```


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---