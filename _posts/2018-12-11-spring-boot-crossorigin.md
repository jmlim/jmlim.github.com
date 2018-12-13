---
layout: post
title:  "스프링 부트에서 크로스도메인 이슈 처리하기. (@CrossOrigin 어노테이션을 사용)"
date:   2018-12-11 12:26:00 +0900
categories: Spring
comments: true
tags: [Spring, CORS, Cross Domain]
---

---

#스프링 부트에서 크로스도메인 이슈 처리하기. (@CrossOrigin 어노테이션을 사용)

크로스도메인 이슈란?
--
Ajax 등을 통해 다른 도메인의 서버에 url(data)를 호출할 경우 XMLHttpRequest는 보안상의 이유로 자신과 동일한 도메인으로만 HTTP요청을 보내도록 제한하고 있어 에러가 발생한다.<br/>
내가 만든 웹서비스에서 사용하기 위한 rest api 서버를 무분별하게 다른 도메인에서 접근하여 사용하게 한다면 보안상 문제가 될 수 있기 때문에 제한하였지만 
지속적으로 웹 애플리케이션을 개선하고 쉽게 개발하기 위해서는 이러한 request가 꼭 필요하였기에 그래서 XMLHttpRequest가 cross-domain을 요청할 수 있도록하는 방법이 고안되었다.<br/>
그것이 CORS 이다.

CORS란?
--
CORS(Cross-origin resource sharing)이란, 웹 페이지의 제한된 자원을 외부 도메인에서 접근을 허용해주는 메커니즘이다.

스프링에서 CORS 설정하기.
--
스프링 RESTful Service에서 CORS를 설정은 @CrossOrigin 어노테이션을 사용하여 간단히 해결 할 수 있다.
RestController를 사용한 클래스 자체에 적용할 수 도 있고, 특정 REST API method에도 설정 가능하다.<br/>
또한, 특정 도메인만 접속을 허용할 수도 있다.  

사용예 ) 
> @CrossOrigin(origins = "허용주소:포트")


예제코드 1) 
 - WebMvcConfigurer를 통해 적용하는 방식.


```java

package io.jmlim.corssample.corssample;
 
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
      // 모든 uri에 대해 http://localhost:18080, http://localhost:8180 도메인은 접근을 허용한다.
        registry.addMapping("/**")
                .allowedOrigins("http://localhost:18080","http://localhost:8180");

    }
}

```

예제코드 2) 
 - @CrossOrigin 어노테이션을 통해 적용하는 방식.


```java
package io.jmlim.corssample.corssample;

 
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
//해당 컨트롤러의 모든 요청에 대한 접근 허용(아래 도메인 두개에 대해서만..)
@CrossOrigin(origins = {"http://localhost:18080", "http://localhost:8180" }) 
@RestController
public class CorssampleApplication {
 
	//아래와 같이 특정 메소드에만 적용할수도 있다.
    //@CrossOrigin(origins = {"http://localhost:18080", "http://localhost:8180" })
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
	
	@GetMapping("/my")
    public String my() {
        return "my";
    }
	
 
    public static void main(String[] args) {
        SpringApplication.run(CorssampleApplication.class, args);
    }
}
```

예제코드 호출하기
 > 만약 이 코드가 실행되는 웹서버의 도메인이 http://localhost:18080과 http://localhost:8080이 아닐경우 fail이 발생한다.
 

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <script
           src="https://code.jquery.com/jquery-3.3.1.js"
            integrity="sha256-2Kok7MbOyxpgUVvAk/HJ2jigOSYS2auK4Pfzbm7uH60="
            crossorigin="anonymous"></script>
    <script>
       // alert(1);
        $(function() {
			  // 만약 이 코드가 실행되는 웹서버의 도메인이 http://localhost:18080과 http://localhost:8080이 아닐경우 fail이 발생한다.
            $.ajax("http://localhost:8180/hello")
                .done(function(msg) {
                    alert(msg);
                }).fail(function() {
                    alert("fail");
                });
        });
    </script>
</head>
<body>

</body>
</html>
```

참고: 
 - http://zamezzz.tistory.com/137 [배워가는블로거]

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

---

