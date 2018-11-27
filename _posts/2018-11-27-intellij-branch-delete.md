---
layout: post
title:  "Intellij에서 git branch 삭제하기"
date:   2018-11-27 18:32:00 +0900
categories: etc
comments: true
tags: [intellij, Git]
---

---
### Intellij에서 git branch 삭제하기

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
