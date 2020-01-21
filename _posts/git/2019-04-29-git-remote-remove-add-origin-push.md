---
layout: post
title:  "git remote branch 변경하기."
date:   2019-04-29 16:00:00 +0900
categories: Git
comments: true
tags: [Git, Remote Branch Change]
---

---

## 깃 리모트 변경 하기

### 기존 리포지토리 pull / push

```
git pull
git add .
git commit -m "clean push"
git push
```

### 기존 리포지토리 remote 제거
```
git remote remove origin
```

### 새 리포지토리 remote 추가
```
git remote add origin https://github.com/playauto/리포지토리명 
```

### 기존 리포지토리를 새 리포지토리 remote 에 전부 push
```
git push -u origin --all
git push -u origin --tags
```

출처 : 
  - https://gist.github.com/480/4681b67d2a906db8c6c1321cc678f05f

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
