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

> **[2026년 추가]** 위 코드에는 실무에서 자주 걸리는 함정이 두 가지 있다.
> 1. **`X-Forwarded-For`는 값이 하나가 아닐 수 있다.** 요청이 프록시를 여러 번 거치면 `클라이언트IP, 프록시1IP, 프록시2IP` 처럼 **콤마로 구분된 목록**이 담긴다. 위 코드처럼 헤더값을 통째로 쓰면 IP 하나가 아니라 이 목록 전체가 그대로 들어가 버린다 — 실제 클라이언트 IP만 쓰려면 첫 번째 값만 잘라 써야 한다. `request.getHeader("X-FORWARDED-FOR").split(",")[0].trim()`
> 2. **`X-Forwarded-For`는 클라이언트가 마음대로 조작해서 보낼 수 있는 값이다.** 로드밸런서/프록시를 거치지 않고 애플리케이션에 직접 요청을 보낼 수 있는 경로가 있다면, 이 헤더는 그냥 요청자가 원하는 값으로 위조 가능하다. IP 기반으로 접근 제어나 로그인 시도 제한 같은 **보안 목적**으로 쓴다면, 반드시 신뢰할 수 있는 리버스 프록시(nginx, ALB 등)만 통해서 들어오도록 네트워크를 막아두고, 그 프록시가 이 헤더를 항상 덮어쓰도록(append가 아니라 overwrite) 설정해야 한다.
>
> Spring을 쓴다면 이 헤더 처리를 직접 파싱하는 대신 내장된 `ForwardedHeaderFilter`(`server.forward-headers-strategy=framework`)를 쓰면 `request.getRemoteAddr()`가 알아서 올바른 클라이언트 IP를 반환해주므로 더 안전하다.

참고
- https://stackoverflow.com/questions/29910074/how-to-get-client-ip-address-in-java-httpservletrequest


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

---

