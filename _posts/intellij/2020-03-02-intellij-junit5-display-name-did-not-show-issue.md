---
layout: post
title:  "Junit5의 @DisplayName 으로 표시한 이름이 Intellij 실행 탭에 표시되지 않는 문제 수정"
date:   2020-03-02 22:02:00 +0900
categories: Intellij
comments: true
tags: [Intellij, Junit5]
---

---

백기선님의 인프런 강의를 듣는 도중.. <br>
각각의 테스트에 이름을 표기하기 위해 @DisplayName을 사용하는 부분이 있었는데 적용이 되지 않고 이전처럼 메소드명이 테스트 이름 그대로 나옴을 확인하여 찾아보았다.

## 현상
 - @DisplayName 을 지정한 테스트 실행

<img src="{{ site.baseurl }}/public/post/junit5-display-name-did-not-show/display-name-did-not-show1.png"/>

## 해결
아래 방법 적용 후 정상적으로 실행탭에 @DisplayName으로 지정한 이름이 정상 표기됨을 확인하였다.

### 1. Build, Execution, Deployment -> Build Tools -> Gradle로 이동한 다음 Run tests using 을 Gradle -> Intellij IDE 로 수정
<img src="{{ site.baseurl }}/public/post/junit5-display-name-did-not-show/display-name-did-not-show2.png"/>

### 2. 수정 후 테스트를 재 실행.
<img src="{{ site.baseurl }}/public/post/junit5-display-name-did-not-show/display-name-did-not-show3.png"/>


출처 :
 - https://medium.com/@sorravitbunjongpean/fix-junit5-display-name-did-not-show-in-run-tab-intellij-a00c94f39679

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

---
