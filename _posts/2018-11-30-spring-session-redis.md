---
layout: post
title:  "Spring boot 환경에서 Spring Session을 통해 WAS가 아닌 다른 저장소에 세션 저장하기."
date:   2018-11-30 11:40:00 +0900
categories: Spring
comments: true
tags: [Spring, Java, Redis]
---

---

Spring boot 환경에서 Spring Session 을 통해 WAS가 아닌 다른 저장소에 저장하기.
--
WAS의 세션을 이용했을 때, 서버가 한 대일 때에는 WAS의 세션을 사용하는데에 문제가 없었을 것이다.
사용자가 증가함에 따라 서버에 과부하가 걸려 WAS를 증설하기 시작할 경우 HttpSession을 사용하면 세션이 두군데서 관리된다.

> ex) 로그인 요청은 1번 서버로 들어와 1번 서버의 세션에 담았는데, 그 다음 요청을 2번에 하게 된다면?? 로그인 상태가 아니다. 

그렇기 때문에 단일 서버 환경에서 세션 기능을 사용하기 위해서는 HttpSession 객체를 활용해도 상관없지만,
멀티 서버 환경에서 세션 기능을 사용하기 위해서는 세션 정보를 Redis와 같은 DB에 저장해야 한다.


리눅스에 Redis 설치하기.
--
먼저 리눅스에 redis를 설치하기 위해서 파일을 직접 다운받거나 apt-get 등을 이용할 수 있다. 
만약 apt-get 패키지 다운로드를 사용할 경우 아래와 같이 커맨드를 입력하면 된다.
```
$ sudo apt-get install redis-server
```
redis 동작하기
--
```
$ redis-server
```

Spring Boot에서 Spring session + redis 로 세션 관리 하기.
--

의존성 추가.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session</artifactId>
</dependency>
```

application.yml 추가.

```yml
spring: 
  redis:
    host: 192.168.0.156 # 레디스가 설치되어있는 서버의 아이피
    port: 6379
```

Spring Session Configuration 추가.

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.session.data.redis.config.annotation.web.http.EnableRedisHttpSession;
import org.springframework.session.web.context.AbstractHttpSessionApplicationInitializer;

@Configuration
@EnableRedisHttpSession
public class RedisSessionConfig extends AbstractHttpSessionApplicationInitializer {
} 
```

레디스에서 세션 저장여부 확인.
<img src="{{ site.baseurl }}/public/post/spring-session/spring-session-redis.png"/>

출처: 
 - http://wiki.sys4u.co.kr/pages/viewpage.action?pageId=8552454
 - https://webisfree.com/2018-01-25/%EB%A6%AC%EB%88%85%EC%8A%A4%EC%97%90-redis-%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0-install

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
