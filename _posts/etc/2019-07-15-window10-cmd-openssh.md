---
layout: post
title:  "윈도우 10에서 OpenSSH 사용하기."
date:   2019-07-15 16:30:00 +0900
categories: ETC
comments: true
tags: [Windows, OpenSSH, ssh, cmd]
---

---
윈도우 10에 openssh 가 내장되어 있어 putty나 xshell 사용할 필요 없이<br>
맥이나 리눅스처럼 ssh 명령어를 통해 서버접근을 할 수 있다는 정보를 얻어 찾아보았다.

## 1. 시작메뉴(윈도우 키) 선택 후 찾기 버튼 클릭 -> 앱 및 기능 검색 후 선택.
<img src="{{ site.baseurl }}/public/post/windows-openssh/openssh-app.png"/>

## 2. 선택적 관리 기능 선택.
<img src="{{ site.baseurl }}/public/post/windows-openssh/manage-optional-features.png"/>

## 3. OpenSSH 서버 선택 후 설치 클릭.
<img src="{{ site.baseurl }}/public/post/windows-openssh/openssh-setup.png"/>

## 4. 뒤로 돌아간 후 OpenSSH 클라이언트 설치된 것 확인.
<img src="{{ site.baseurl }}/public/post/windows-openssh/openssh-setup-end.png"/>

## 5. 윈도우 재시작. (재시작을 한 후 로그인 해야 시스템에 적용됨.)
<img src="{{ site.baseurl }}/public/post/windows-openssh/window-restart.png"/>

## 6. cmd 창 열고 ssh 실행해서 설치여부 확인.
<img src="{{ site.baseurl }}/public/post/windows-openssh/cmd-ssh.png"/>

## 7. cmd 창에서 ssh 접근 확인. (처음 접근 시 yes 타이핑 하고 시작.)
 - 접속방식은 아래와 같다.
 
 ```
 ssh 아이디@서버아이피
 또는
 ssh 서버아이피 -l 아이디
 
 포트가 다를경우 
 ssh 아이피 -l 아이디 -p 포트
 ```
 
 <img src="{{ site.baseurl }}/public/post/windows-openssh/cmd-ssh-connect.png"/>

## 기타. Windows 10 호스트 키 저장경로
  > %UserProfile%\.ssh\known_hosts
  
   - 접속하는 서버를 재설치 하거나 설정 변경이 있는 경우 호스트 키가 일치하지 않아 접속에 문제가 생길 수 있음.
   - 윈도우 10에 설치된 OpenSSH의 호스트키는 위 경로에 저장되며 편집 가능하다.
   - 만일 접속에 문제가 생기는 서버가 있을 경우 편집기에서 확인 후 해당 라인을 삭제한 다음 저장 후 재접속을 하면 된다.
 
참고자료: 
 - https://amorfati-1000.tistory.com/50
 - https://extrememanual.net/12589

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

---

