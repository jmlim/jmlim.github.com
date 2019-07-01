---
layout: post
title:  "Ubuntu-18.04 데스크탑에서 Intellij 사용시 한글텍스트 깨짐 현상 해결"
date:   2019-07-02 00:01:00 +0900
categories: Intellij
comments: true
tags: [Intellij, Ubuntu, ubuntu-18.04]
---

---

인텔리제이 설치 후 파일을 열어본 순간 한글이 네모로 나오는 현상이 발생하였다.

아래 블로그를 참고 후 해결하였다.
```
sudo apt install fonts-nanum
sudo fc-cache -fv
```

1. 나눔 이라는 단어가 포함된 글꼴 패키지 모두 설치
2. 글꼴 캐시 삭제 (시간이 오래 걸린다.)
3. 리부팅 (글꼴 캐시 삭제 후 알림창 나옴.)


출처 : 
 - http://memo.polypia.net/archives/3043
 - https://followers.tistory.com/26 

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

---
