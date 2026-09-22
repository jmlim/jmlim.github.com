---
layout: post
title:  "스프링 부트에서 JSP view 설정하기."
date:   2018-12-13 14:05:00 +0900
categories: Spring
comments: true
tags: [Spring, JSP]
---

---

기본적으로 spring-boot-starter-web 에 포함된 tomcat은 JSP를 포함하지 않는다.<br/>
하지만 간단한 설정만으로도 JSP view를 사용 가능하다.<br>

> **[2026년 추가]** 중요한 제약이 하나 있다 — **Spring Boot는 실행 가능한 JAR(executable jar)로 패키징할 경우 JSP를 지원하지 않는다.** IDE에서 돌리거나(exploded 상태) WAR로 패키징해서 외부 톰캣에 배포할 땐 이 글대로 잘 동작하지만, `./gradlew bootJar`나 `mvn package`로 JAR를 만들어 `java -jar`로 실행하면 JSP가 안 뜨거나 404가 난다. JAR로 배포해야 한다면 애초에 JSP 대신 **Thymeleaf**(`spring-boot-starter-thymeleaf`)를 쓰는 걸 권장한다 — Spring Boot 공식 문서에서도 JSP는 "practical only when using war" 라고 명시하고 있다. WAR 배포가 확정된 레거시 프로젝트가 아니라면, 요즘 새로 시작하는 프로젝트에서 JSP+Tiles 조합은 잘 안 쓴다.

일단 dependency를 pom.xml 에 추가한다.

#### pom.xml 에 아래 구문 추가.

```xml
....
<!-- JSP 쓰기 -->
<dependency>
	<groupId>org.apache.tomcat.embed</groupId>
	<artifactId>tomcat-embed-jasper</artifactId>
</dependency>

<dependency>
	<groupId>javax.servlet</groupId>
	<artifactId>jstl</artifactId>
</dependency>
....

```

#### application.properties 에 추가할 내용 (jsp 파일이 들어갈 경로 및 확장자 지정.)

```properties

spring.mvc.view.prefix=/WEB-INF/jsp/
spring.mvc.view.suffix=.jsp

```

#### jsp 페이지 추가.

<img src="{{ site.baseurl }}/public/post/springboot/spring-boot-jsp-setting.png"/>
<br>
ex) signin.jsp

```jsp
<!DOCTYPE html>
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>로그인</title>
</head>
<body>
로그인 페이지가 될 부분.
</body>
</html>
```

#### 컨트롤러 작성.

```java

import java.util.Date;
import java.util.Map;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class SignController {
  
  @GetMapping("/")
  public String index(Map<String, Object> model) {
    model.put("time", new Date());
    return "index";
  }
  
  @GetMapping("/signin")
  public String siginin(Map<String, Object> model) {
      model.put("time", new Date());
      return "signin";
  }
  
  @GetMapping("/signup")
  public String siginup(Map<String, Object> model) {
    model.put("time", new Date());
    return "signup";
  }
}

```

#### url 호출
- http://localhost:8080 <br>
- http://localhost:8080/siginin <br>
- http://localhost:8080/siginup <br>

위 url 을 차례대로 실행 시 각각 index.jsp, siginin.jsp, siginup.jsp 파일이 띄워짐을 확인할 수 있다.<br>


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

---

