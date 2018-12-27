---
layout: post
title:  "자바에서 클라이언트 IP 주소를 얻는 방법"
date:   2018-12-14 15:30:00 +0900
categories: Java
comments: true
tags: [Java, Spring, Servlet, 클라이언트 IP]
---

---

기본적으로는 Java 서블릿에서 HttpServletRequest.getRemoteAddr()을 사용하여 Java 웹 프로그램에 접근하는 클라이언트의 IP 주소를 가져올 수 있다.
 
```java

import javax.servlet.http.HttpServletRequest;

protected String getRemoteAddr(HttpServletRequest request){
    return request.getRemoteAddr();
}

```



하지만 프록시 환경 또는 클라우드 위에 있는 웹 응용 프로그램의 경우 HTTP 요청 헤더 X-Forwarded-For(XFF)를 통해 클라이언트 IP 주소를 가져와야 한다.<br>
WAS 는 보통 2차 방화벽 안에 있고 Web Server 를 통해 client 에서 호출되거나 cluster로 구성되어 load balancer 에서 호출된다.
이럴 경우에서 getRemoteAddr() 을 호출하면 접근한 클라이언트의 외부아이피가 아닌 웹서버나 load balancer의 IP 가 나오면서 같은아이피가 계속 찍히게 된다. <br>
위와 같은 문제를 해결하기 위해 사용되는 HTTP Header인 X-Forwarded-For 값을 확인해서 있으면 해당 키값을 사용하고 없으면 getRemoteAddr() 를 사용한다.

```java

import javax.servlet.http.HttpServletRequest;

protected String getRemoteAddr(HttpServletRequest request){
    return (null != request.getHeader("X-FORWARDED-FOR")) ? request.getHeader("X-FORWARDED-FOR") : request.getRemoteAddr();
}

```

참고
- https://stackoverflow.com/questions/29910074/how-to-get-client-ip-address-in-java-httpservletrequest


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

---

