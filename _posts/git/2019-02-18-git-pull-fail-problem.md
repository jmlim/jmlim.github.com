---
layout: post
title:  "git merge 후 pull 실패 시 해결 방안 - You have not concluded your merge (MERGE_HEAD exists)"
date:   2019-02-18 16:40:00 +0900
categories: Git
comments: true
tags: [Git, Merge]
---

---

## 증상
> You have not concluded your merge (MERGE_HEAD exists)

## 해결방법
### 1. 머지 취소
```
 git merge --abort
```
### 2. 충돌 해결

### 3. 병합을 추가 하고 커밋

```
 git status
 git commit -am "커밋 내용"
```

> 커밋을 제대로 하지 않았을 경우 아래 메세지가 뜰 수 있음. <br/>
> Pulling is not possible because you have unmerged files

### 4. 다시 내려 받기
```
git pull
```

출처 : 
  - https://gutmate.github.io/2018/04/18/git-pull-fail/

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
