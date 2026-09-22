---
layout: post
title:  "HandlerMethodArgumentResolver 를 통해 특정 Argument 사용 시 변경하기."
date:   2019-04-29 19:00:00 +0900
categories: Spring
comments: true
tags: [Spring, Java, HandlerMethodArgumentResolver, Argument]
---

---

* HandlerMethodArgumentResolver 를 사용하여 컨트롤러 메소드에 선언된 특정 Arguments를 변경하거나 또는 권한체크를 하여 arguments에 값을 넣어주거나 할 수 있다.

> **[2026년 추가]** 이 글에서 쓴 `WebMvcConfigurerAdapter`는 Spring 5(2017년)부터 deprecated 되었고, Spring 6부터는 아예 삭제되어 더 이상 컴파일도 안 된다. 자바 8부터 인터페이스에 기본 구현(default method)을 넣을 수 있게 되면서, 모든 메서드를 구현할 필요 없이 필요한 메서드만 오버라이드하도록 도와주던 이 어댑터 클래스 자체가 필요 없어졌기 때문이다. 지금은 `WebMvcConfigurer` 인터페이스를 바로 구현하면 된다 — 아래 2번 항목의 코드만 그렇게 바꾸면 이 글의 나머지 내용은 최신 Spring Boot에서도 그대로 유효하다.

## 0. 멤버 클래스
~~~java

@Getter
@RequiredArgsConstructor
public class Member {
  private final String username;
  private final String email;
}
~~~


## 1. MemberHandlerMethodArgumentResolver 추가.

1. 컨트롤러에서 파라미터를 바인딩.
 - ex) 특정클래스나 특정 어노테이션등의 요청 파라미터를 수정해야한다거나 또는 클래스의 파라미터를 조작, 혹은 공통적으로 써야하는 파라미터들을 바인딩 해주는 역할을 한다.
 - 예제에서는 유저정보를 고정으로 가져와서 파라미터로 쓰기 위해 사용하였다.

```java
@Slf4j
public class MemberHandlerMethodArgumentResolver implements HandlerMethodArgumentResolver {

    /**
     * @param parameter
     * @return
     */
    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return Member.class.isAssignableFrom(parameter.getParameterType());
    }

    /**
     * @param parameter
     * @param mavContainer
     * @param webRequest
     * @param binderFactory
     * @return
     * @throws Exception
     */
    @Override
    public Member resolveArgument(MethodParameter parameter,
                                  ModelAndViewContainer mavContainer,
                                  NativeWebRequest webRequest,
                                  WebDataBinderFactory binderFactory) throws Exception {

        // 컨트롤러의 메소드에 아래 아규먼트가 선언되었을 때만 아래 로직 태움..
        if (parameter.getParameterType().equals(Member.class)) {
           Member member = new Member("jmlim", "hackerljm@naver.com");
           return member;
        }
    }
}

```

## 2. WebConfig에 작성한 ArgumentResolver 등록


```java
// (2026년 수정) WebMvcConfigurerAdapter 대신 WebMvcConfigurer 인터페이스를 직접 구현
@Configuration
public class WebConfig implements WebMvcConfigurer {
  
  @Override
  public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
    argumentResolvers.add(new MemberHandlerMethodArgumentResolver());
  }
}

```

## 3. 컨트롤러에서 사용.

> Member 클래스에는 아무 파라미터를 넣지 않아도 테스트를 해보면 위에서 넣어둔 jmlim과 hackerljm@naver.com 이 나오게 된다.

```java

@Slf4j
@RestController
@RequestMapping()
public class TestController {


    /**
     *
     * @param member
     * @return
     */
    @GetMapping("/member/info")
    public ResponseEntity getLoginInfo(Member member) {
       /**해당 메소드 호출 전에 resolveArgument 를 타게 됨.*/
       ....
    }

```

참고: 
 - http://wonwoo.ml/index.php/post/1092

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
