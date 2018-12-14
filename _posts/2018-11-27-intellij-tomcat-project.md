---
layout: post
title:  "IntelliJ에서 프로젝트 Tomcat 설정하여 올리기"
date:   2018-11-27 23:44:00 +0900
categories: Intellij
comments: true
tags: [Intellij, Tomcat]
---

---

1. tomcat 에서 돌아갈 프로젝트를 내려받거나 셋팅한다.
  ```
  ex) https://github.com/계정명/프로젝트명.git 을 clone하여 내려받음.
 ```
2. 내려받은 후 프로젝트에서 사용할 java 버전을 설정한다.

3. 톰캣설치
  http://tomcat.apache.org/ 에서 프로젝트 환경에 맞는 tomcat 다운로드 및 압축해제.

4. 톰캣 환경 설정
```
 4-1) 우측 상단 ▶버튼 좌측에 있는 ▽ 버튼 클릭 -> Edit Configurations 클릭
 4-2) 설정파일 열리면 + 버튼 눌러 tomcat server 선택
 4-3) 실행시킬 톰캣의 이름 지정 및 톰캣위치(3. 에서 다운받은 톰캣 위치) 지정
 4-4) 지정한 톰캣에 프로젝트 폴더 deploy (deploy 탭에서 exploded war  파일 선택 후 OK 클릭)
```

마지막으로 ▶ 또는 디버그 버튼으로 톰캣을 실행한다.

추가로 아래내용도 참고하게 될 것이다. (Intellj 에서 jsp 수정 후 tomcat 재구동 없이 배포하기.)
>http://jmlim.github.io/intellij/2018/11/23/intellij-tomcat-non-restart/



[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
