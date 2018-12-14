---
layout: post
title:  "Intellj 에서 jsp 수정 후 tomcat 재구동 없이 배포"
date:   2018-11-23 15:11:00 +0900
categories: Intellij
comments: true
tags: [Intellij, IDE, Java]
---

---

jsp를 수정하게되면 tomcat을 재구동하지 않고서는 웹에 반영이 되지 않는 증상이 발생하여 설정을 수정하였다.

1. 사용하는 was나 웹컨테이너의 설정 에 들어간다.
2. war: exploded 를 선택후 Edit Artifact 를 누른다. 
3. output 디렉토리 경로를 현재 내 웹소소의 경로로 맞춘다. maven 사용시 target 디렉토리가 아닌 src/main/webapp 로 설정한다.  

<img src="{{ site.baseurl }}/public/post/tomcat/exploded_set.png" width="800px" height="400px"/>

>참조문서 :
<http://devsh.tistory.com/entry/Intellj-에서-jsp-수정-후-tomcat-재구동-없이-배포하기 [날샘 코딩]>

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
