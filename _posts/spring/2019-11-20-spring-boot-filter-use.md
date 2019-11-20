---
layout: post
title:  "[Spring] Spring boot 에서 Filter 사용하기"
date:   2019-11-20 19:12:00 +0900
categories: Spring
comments: true
tags: [Spring, Spring boot, Fiiter, Content-Length]
---

---
## Bean 등록
스프링부트에서는 web.xml 이 더 이상 사용되지 않아 서블릿이나 필터를 org.springframework.boot.web.servlet 의 RegistrationBean 을 통해 등록해야 한다.

형식
~~~java
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FilterConfig {
  
  @Bean
  public FilterRegistrationBean getFilterRegistrationBean() {
    FilterRegistrationBean registrationBean = new FilterRegistrationBean(new HSTSFilter());
    // registrationBean.addUrlPatterns("/*"); // 서블릿 등록 빈 처럼 패턴을 지정해 줄 수 있다.
    return registrationBean;
  }
}
~~~


> rest 형태의 API 호출 시 Transfer-Encoding: chunked 대신 Content-Length 정보 나오도록 하고 싶을 때 필터 사용 예제.
 - ShallowEtagHeaderFilter 를 필터 체인에 추가.


~~~java
@Configuration
public class FilterConfig {

     @Bean
    public FilterRegistrationBean filterRegistrationBean() {
        FilterRegistrationBean filterBean = new FilterRegistrationBean();
        filterBean.setFilter(new ShallowEtagHeaderFilter());
        filterBean.setUrlPatterns(Arrays.asList("*"));
        return filterBean;
    }
}
~~~

> 응답결과 
~~~
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
X-Application-Context: application:sxp:8090
ETag: "05e7d49208ba5db71c04d5c926f91f382"
Content-Type: application/json;charset=UTF-8
Content-Length: 232
Date: Wed, 16 Dec 2015 06:53:09 GMT
~~~


### 참고자료 : 
 - https://gs.saro.me/dev?tn=503
 - https://stackoverflow.com/questions/24156490/how-to-set-content-length-in-spring-mvc-rest-for-json

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

---
