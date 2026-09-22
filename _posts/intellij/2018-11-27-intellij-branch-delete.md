---
layout: post
title:  "Intellij에서 git branch 삭제하기"
date:   2018-11-27 18:32:00 +0900
categories: Intellij
comments: true
tags: [Intellij, Git]
---

---

명령어로 삭제하는 방법 
---

console 명령어로 삭제 시 local branch 삭제
```
git branch --delete <branch_name> 
```
 
remote branch 삭제
```
git push <repository> :<branch_name>

예)) git push origin :branch_hotfix
```
> **[2026년 추가]** 위 `:<branch_name>` 문법(빈 값을 push해서 삭제)은 지금도 동작하지만, 요즘은 의미가 더 명확한 `git push origin --delete <branch_name>` 을 더 많이 쓴다.

Intellij 에서는 branch 또는 remote branch 를 오른쪽 하단메뉴를 통해 간편하게 삭제할 수 있다. 
---
1. 오른쪽 하단 Git 메뉴 클릭 후 삭제 하려는 브랜치 클릭 후 삭제.
<img src="{{ site.baseurl }}/public/post/gitimg/intellij-branch-delete.png" width="800px" height="400px"/>
2. Remote branch 를 삭제 후 원격 저장소가 있는 곳에 들어가서 확인하면 삭제된 것을 확인할 수 있다.
<img src="{{ site.baseurl }}/public/post/gitimg/github-branch-delete.png" width="800px" height="400px"/>


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
