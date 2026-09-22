---
layout: post
title:  "내 계정으로 GitHub에 Commits 시 contributions 그래프(초록색 타일)이 표시가 되지 않는 경우 확인할 것."
date:   2019-07-11 20:05:00 +0900
categories: Git
comments: true
tags: [Git, Github, 초록색 타일, contributions graph]
---

---

GitHub은 커밋이 연결된 사용자의 이메일을 사용한다. <br/>
만약 계정에 등록된 이메일과 로컬 git config 설정에 등록된 이메일의 정보가 다를 경우 
Github contributions 그래프 (초록색 타일) 에 표시되지 않는다.

뭔가 작업을 열심히 했는데 그래프에 나오지 않으면 조금 기분이 그러하고 억울하기에 아래와 같이 확인해본다.

## 로컬에서 확인할 것 

$ git config user.email
> jmlim@xxx.com

## 이메일이 맞지 않을경우 전역 설정 변경하기.

$ git config --global user.email "jmlim@xxx.com"

## 깃헙에서 확인할 것.

### 1 . 로그인 후 계정의 settings의 email 메뉴 선택.
> (url : https://github.com/settings/emails) 

<img src="{{ site.baseurl }}/public/post/github-contribute-graph/github-contribute-graph-add.png"/>
[이미지 참고] 

### 2. GitHub 계정에서 연결된 이메일을 확인 후 로컬에 등록한 이메일이 없으면 Add email address로 추가.
 - Add email address 로 등록 후 해당 이메일로 아래 해당제목의 메일이 갈 것임.
 > [GitHub] Please verify your email address.
 
### 3. 클릭 후 확인 

다시 github 에 접근해보면 그동안 커밋했던 내용이 contributions 그래프에 보여지게 될 것이다.

> **[2026년 추가]** 최근에 직접 겪어보니, 이메일이 이미 Verified 상태인데도 잔디가 바로 안 뜨는 경우가 있었다. 이메일 말고도 아래 조건들을 같이 확인해볼 것.
> - 커밋한 저장소가 **Public**인지 (Private 저장소 커밋은 프로필 설정에서 "Include private contributions" 를 켜야 잔디에 반영됨)
> - 커밋이 저장소의 **default 브랜치**(보통 `main`/`master`)에 있는지 — 다른 브랜치에 커밋만 하고 병합 전이라면 안 잡힘
> - push는 성공했는데 잔디만 안 뜬다면, 이메일/브랜치/공개 여부 다 정상이어도 **그래프 자체가 반영되는 데 몇 분 정도 걸릴 수 있다.** 강력 새로고침 후에도 안 뜨면 그때 이메일부터 의심해보면 된다.


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
