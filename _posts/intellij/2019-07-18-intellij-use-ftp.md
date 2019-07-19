---
layout: post
title:  "인텔리제이 FTP 연동하여 서버에 있는 소스 쉽게 수정하기."
date:   2019-07-18 21:00:00 +0900
categories: Intellij
comments: true
tags: [Intellij, FTP]
---

---

사실 자바로 웹 개발시 직접 서버 붙어서 수정하고.. 뭐 그런경우는 이제 별로 없다.

그런데 최근에 루비 온 레일즈로 된 소스를 분석할 일이 생겼다.<br/>
서버에 있는 소스 보고 수정하고 확인하는게 여간 불편한 일이 아니었다.<br/>
다행히 인텔리제이 라이센스가 있고 인텔리제이의 FTP 기능을 통해 소스를 로컬에 내려받고 수정시마다 서버에 쉽게 반영할 수 있었다.<br/>

## FTP 연동 방법
> 서버엔 내가 수정하고 테스트 할 수 있는 소스가 이미 내려받아 있다고 가정한다.

## 1. intellij 에서 프로젝트를 생성한다. 
 - 루비 플러그인 설치 후 루비 프로젝트로 생성하였다.
 - 프로젝트 성격에 따라 생성하면 된다.
 

## 2. intellij 상단메뉴 tools -> Deployment -> Configuration에 접근한다.

<img src="{{ site.baseurl }}/public/post/intellij-use-ftp/tools-deployment-configuration.png"/> 

### 2.1. 왼쪽 상단의 + 버튼으로 커넥션 이름 추가 후 Connection 탭에 sftp 로 접근할 호스트 및 계정정보, 소스 내려받은 root path 정보를 작성 한다.

<img src="{{ site.baseurl }}/public/post/intellij-use-ftp/deployment-configuration.png" />

## 3. Connection 탭 옆의 Mappings 탭에 Deployment path를 지정한다. 
 - 아까 root path 에 지정했던 정보를 지정하면 된다.

<img src="{{ site.baseurl }}/public/post/intellij-use-ftp/deployment-configuration-mapping.png" />

## 4. 지정 후 intellij 상단메뉴 tools -> Deployment -> Browse Remote Host 에 가서 아까 Configuration에 지정한 호스트 선택 후 접근이 되어 파일 확인을 해본다.

## 5. Browse Remote Host 의 루트 폴더 선택
 - 우측 클릭 후 Download from here 선택하여 소스를 로컬에 내려 받는다.

 <img src="{{ site.baseurl }}/public/post/intellij-use-ftp/remote-right-click.png"/>

## 6. 위까지 되었으면 Deployment 메뉴 선택 시 upload to 리모트 호스트 정보 .. Download from 호스트 정보 등등 여러가지가 생긴다.
- Sync with Deployed to 리모트 호스트 정보로 소스가 동기화가 되어있는 상태인지.. 파일명은 깨지지 않았는지 살핀다.

 <img src="{{ site.baseurl }}/public/post/intellij-use-ftp/tools-sync-menu.png"/>

## 7. 이제 소스 수정 후 동기화 한 다음 Deployment -> upload to 호스트 정보로 소스를 쉽게 올릴 수 있다.

 <img src="{{ site.baseurl }}/public/post/intellij-use-ftp//deployment-upload.png" />

## 8. intellij 상단메뉴 tools -> Deployment -> Automatic Upload 를 선택하면 소스 수정할 때마다 자동으로 반영이된다.
- 이미 반영이 되고 나면 되돌릴 수 없으므로 주의한다. 

 <img src="{{ site.baseurl }}/public/post/intellij-use-ftp/automatic-upload.png" />

참고자료: 
- https://0penster.tistory.com/22
- https://www.jetbrains.com/help/idea/creating-a-remote-server-configuration.html

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

---
