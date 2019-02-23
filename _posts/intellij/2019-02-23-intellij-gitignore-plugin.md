---
layout: post
title:  " IntelliJ IDEA에서 .gitignore에 파일 / 폴더를 마우스 우클릭으로 편하기 추가하기."
date:   2019-02-23 15:20:00 +0900
categories: Intellij
comments: true
tags: [Intellij, Git]
---

---

Eclipse 에서 IntelliJ IDEA로 전환시에 불편한 점 중에 하나가 .gitignore 파일에 마우스 우클릭으로 커밋에 제외할 파일을 빠르게 추가하지 못하는 것이다.<br>
Eclipse에서는 파일 / 디렉토리를 마우스 오른쪽 버튼으로 클릭하고 ' .gitignore에 추가 '를 선택할 수 있다.

하지만 당연히 IntelliJ IDEA에서 이와 같은 것이 있고 플러그인으로 제공한다.


## 1. File -> Settings의 plugin으로 검색하여 거기서 또 ignore로 검색.
(Ctrl + Alt + S 단축키 사용)
<img src="{{ site.baseurl }}/public/post/intellij-gitignore/intellij-ignore-plugin-setup1.png"/> 

search in repositories 클릭 후 검색하여 플러그인 설치.

## 2. intellij를 재부팅 후 설치가 완료되면 gitignore 파일의 아이콘이 변함.
 > gitignore 파일이 없을 경우 파일생성하라는 노란색 바가 나오게 됨.

<img src="{{ site.baseurl }}/public/post/intellij-gitignore/intellij-gitignore-file.png"/> 

## 3. 이제 파일 또는 경로를 선택 후 마우스 우측 클릭 시 gitignore에 자동으로 포함시켜 커밋대상에서 편리하게 제외시킬 수 있다.
<img src="{{ site.baseurl }}/public/post/intellij-gitignore/intellij-gitignore-plugin.png"/> 


## 0. 추가 : .gitignore가 작동하지 않을때 대처법

위의 내용 적용 후에 .gitignore 가 제대로 작동되지 않아 ignore처리된 파일이 커밋대상에 포함되어 나오는 문제가 발생하여 이 내용을 추가로 작성하였다.
<br/>
git의 캐시가 문제가 되는거라 아래 명령어로 캐시 내용을 전부 삭제후 다시 add All한다.

한번에 처리가 안되서 한번더 반복해서 입력하니 정상적으로 적용되었다.

```
git rm -r --cached .
git add .
```

참고자료 : 
 - https://code.i-harness.com/ko-kr/q/158a1da
 - https://jojoldu.tistory.com/307

```

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

---
