---
layout: post
title:  "eclipse(STS)에 lombok(롬복) 적용."
date:   2018-06-15 11:43:23 +0900
categories: IDE
comments: true
tags: [eclipse, Lombok]
---

---
### eclipse(STS)에 lombok(롬복) 적용

먼저 lombok 이란?

__Lombok는 Java의 코드를 다이어트 시켜주기 위해 나온것으로 볼 수 있다.__ <br>
Java의 큰 단점중 하나는 정보를 일일이 입력해야 하다 보니까 코드가 너무 길어진다는 것이다. <br>
Getter/Setter 메소드를 하나씩 만들어 주려면.. IDE 자동완성 기능을 사용하더라도 너무 번거롭다. <br>

Lombok은 자바에서 @Getter, @Setter 같은 annotation 기반으로 관련 기존 DTO, VO, Domain Class 작성할 때, 
멤버 변수에 대한 Getter/Setter Method, Equals(), hashCode(), ToString()과 
멤버 변수에 값을 설정하는 생성자 등등을 자동으로 생성해 주는 라이브러리다.  <br>

아래 주소로 들어가면 어노테이션 별로 어떤 코드들을 자동으로 생성해주는지 확인 할 수 있다. <br>
<http://jnb.ociweb.com/jnb/jnbJan2010.html>


<img src="{{ site.baseurl }}/public/post/lombok/lombok2.png" width="800px" height="400px"/>

일단 이 글에서는 lombok을 Eclipse(STS)및 eclipse 프로젝트에 적용하는 방법을 설명한다.<br>

- 우선 lombok을 적용할 프로젝트의 pom.xml에 lombok에 대한 의존성 추가.<br>

```xml
<dependency>
    <groupid>org.projectlombok</groupid>
    <artifactid>lombok</artifactid>
</dependency>
```
1. https://projectlombok.org/download 페이지에서 lombok jar 파일 다운로드.

2. lombok.jar 파일이 있는 경로로 가서 해당 jar 파일 더블클릭 하여 실행. (실행이 안될경우 cmd 창에서 java -jar lombok.jar 실행.)
<img src="{{ site.baseurl }}/public/post/lombok/lombok3.png" width="800px" height="400px"/>

3. Specify location.. 버튼 클릭 후 lombok을 적용할 eclipse(STS) 경로 지정. 지정 후 install/update 버튼 클릭하여 설치.
<img src="{{ site.baseurl }}/public/post/lombok/lombok4.png" width="800px" height="400px"/>

4. eclipse가 켜져 있었다면 재시작 후 lombok 적용한 프로젝트의 코드 확인.
<img src="{{ site.baseurl }}/public/post/lombok/lombok5.png" width="800px" height="400px"/>



>참조문서 :
* <http://ijbgo.tistory.com/5>
* <http://jnb.ociweb.com/jnb/jnbJan2010.html>

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---