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

> **[2026년 추가]** 지금은 `workspace.xml`을 직접 안 건드려도 된다 — 각 Run/Debug Configuration 화면에 **"Shorten command line"** 드롭다운이 내장되어 있고, 여기서 `JAR manifest` / `classpath file` / (Java 9+) `@argfile` 방식 중 골라서 같은 문제를 해결할 수 있다. `Modify options`에서 이 항목을 보이게 켤 수 있다. 팀원들과 설정을 공유해야 한다면(이 글의 원래 목적처럼) 이 방식이 XML을 손으로 고치는 것보다 안전하다.

출처 :
 - https://devis.cool/quick-fix/quickfix-intellij-idea-command-line-is-too-long-shorten-command-line-for/

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

---
