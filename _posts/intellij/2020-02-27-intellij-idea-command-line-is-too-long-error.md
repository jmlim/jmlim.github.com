---
layout: post
title:  "IntelliJ IDEA 에서 빌드시 Command line is too long. Shorten command line for .. 에러 발생 문제 해결"
date:   2020-02-27 22:35:00 +0900
categories: Intellij
comments: true
tags: [Intellij, Intellij Build Error]
---

---

현상 : 인텔리제이 빌드 시 Command line is too long. Shorten command line for .. 메세지 뜨고 난 후 빌드 진행되지 않음.
### Error 메세지:
~~~
Error running 'All in project-name': Command line is too long. Shorten command line for All in project-name or also for JUnit default configuration.
~~~

### 해결 방법

~~~xml
 - 프로젝트 루트 경로에서 .idea/workspace.xml에서 파일을 열어
 <component name = "PropertiesComponent"> 섹션 안에  <property name="dynamic.classpath" value="true" /> 태그 추가.
~~~
 - 아래와 같은 모양이면 된다.

~~~xml
<component name="PropertiesComponent">
    <property name="dynamic.classpath" value="true" /> <!-- 추가 한 태그 -->
    <property name="WebServerToolWindowFactoryState" value="false" />
    <property name="aspect.path.notification.shown" value="true" />
    <property name="last_opened_file_path" value="$PROJECT_DIR$/pom.xml" />
    <property name="nodejs_interpreter_path.stuck_in_default_project" value="undefined stuck path" />
    <property name="nodejs_npm_path_reset_for_default_project" value="true" />
</component>
~~~

출처 :
 - https://devis.cool/quick-fix/quickfix-intellij-idea-command-line-is-too-long-shorten-command-line-for/

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

---
