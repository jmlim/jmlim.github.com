---
layout: post
title:  "git stash 사용하기. (Intellj 포함)"
date:   2020-01-30 15:00:00 +0900
categories: Git
comments: true
tags: [Git, Git Stash, Intellij Git Stash]
---

---

## git stash 란?
아직 마무리하지 않은 작업을 잠시 저장할 수 있도록 하는 명령어이다. 
 - 이를 통해 아직 완료하지 않은 일을 commit하지 않고 나중에 다시 꺼내와 마무리할 수 있다.

## SYNOPSIS
~~~
git stash list [<options>]
git stash show [<options>] [<stash>]
git stash drop [-q|--quiet] [<stash>]
git stash ( pop | apply ) [--index] [-q|--quiet] [<stash>]
git stash branch <branchname> [<stash>]
git stash [push [-p|--patch] [-k|--[no-]keep-index] [-q|--quiet]
	     [-u|--include-untracked] [-a|--all] [-m|--message <message>]
	     [--] [<pathspec>…​]]
git stash clear
git stash create [<message>]
git stash store [-m|--message <message>] [-q|--quiet] <commit>
~~~

## git stash 기본적인 사용법
 - git stash
    - 현재 작업을 stash 영역에 저장해두고 branch를 head로 리셋(git reset --hard)
    - 기본 명칭 WIP 로 저장됨
 - git stash -u
    - 새롭게 추가한 파일도 함께 stash 영역에 저장
 - git stash save 작업명
    - git stash 로 저장할 때 명칭을 주어 저장함
 - git stash list
    - stash들 보기.
    - 명시적으로 삭제하지 않으면 남아 있음
 - git stash apply
    - 가장 최근에 저장한 stash를 복원.
 - git stash apply stash@{숫자}
    - stash@{숫자} 의 임시 저장이 복원된다.
        - git stash list 명령어를 실행하면 리스트 앞에 stash@{0} 이렇게 개별적인 id 값이 있는데 이를 적용한다.
 - git stash drop
    - 가장 최근에 저장한 stash 삭제.
 - git stash drop stash@{숫자}
    - stash@{숫자} 리스트가 삭제된다.
 - git stash clear
    - stash 기록이 모두 제거된다.
 - git stash pop
    - stash를 복원하고 바로 제거 된다.
 - git stash branch
    - stash 할 당시의 커밋을 checkout 한 후 새로운 브랜치를 만들고 여기에 적용 후 stash를 제거한다.
    
## 인텔리제이에서 stash 기능 사용

### stash 생성
###  1. 프로젝트 오른쪽 클릭 후 git -> Repository -> Stash Changes 
 > 또는 Find Action (shift 2번 클릭 후 상단바 선택) 후 Stash Changes 입력

 <img src="{{ site.baseurl }}/public/post/gitimg/git-stash1.png" width="800px" height="400px"/>
 
### 2. 작업명 작성 후 Create Stash 클릭

<img src="{{ site.baseurl }}/public/post/gitimg/git-stash2.png" width="800px" height="400px"/>

### stash 가져오기 
###  1. 프로젝트 오른쪽 클릭 후 git -> Repository -> UnStash Changes 
 > 또는 Find Action (shift 2번 클릭 후 상단바 선택) 후 Unstash Changes 입력

<img src="{{ site.baseurl }}/public/post/gitimg/git-stash3.png" width="800px" height="400px"/>

### 2. 목록에서 가져올 stash 선택 후 Apply Stash 클

<img src="{{ site.baseurl }}/public/post/gitimg/git-stash4.png" width="800px" height="400px"/>

## 참고자료
 - https://gmlwjd9405.github.io/2018/05/18/git-stash.html
 - https://suwoni-codelab.com/git/2018/04/06/Git-stash/


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
