---
layout: post
title:  "Linux 에서 cp 명령시 오류 (omitting directory 'directory name') 발생시 해결 방법"
date:   2019-07-16 11:20:00 +0900
categories: ETC
comments: true
tags: [Linux, omitting directory 'directory name']
---

---

## 현상이 발생하는 이유
 - 해당 디렉토리가 사용중일 경우
 - 하위 디렉토리는 복사를 하지 않을때

### 아래와 같이 -r 옵션을 사용하여 해결 가능하다.

```
cp -r [원본디렉토리] [복사할위치]
```

출처: 
- https://ukthe33.tistory.com/entry/cp-%EB%AA%85%EB%A0%B9%EC%8B%9C-%EC%98%A4%EB%A5%98-omitting-directory-directory-name

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

---

