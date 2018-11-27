---
layout: post
title:  "[intellij-idea] IntelliJ를 사용하여 git commit 메시지 수정"
date:   2018-11-27 17:22:00 +0900
categories: git
comments: true
tags: [Git, intellij]
---

---
### IntelliJ를 사용하여 git commit 메시지 수정

로컬에만 커밋 되어있고 push 처리가 되지 않았을 경우에 할 수 있는 2가지 방법.

1. IntelliJ 2017.1 이상 => 로그로 이동하여 + 단어를 오른쪽 클릭 후 F2 키를 눌러서 커밋메세지를 변경할 수 있다.
<img src="{{ site.baseurl }}/public/post/gitimg/git_commit_update.png" width="800px" height="400px"/>
2. jetBrains Go to View -> Version Control -> Log 탭 선택 후 오른쪽 마우스를 클릭하고 undo commit 을 선택하면 다시 커밋할 수 있다.
<img src="{{ site.baseurl }}/public/post/gitimg/git_commit_update1.png" width="800px" height="400px"/>

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
