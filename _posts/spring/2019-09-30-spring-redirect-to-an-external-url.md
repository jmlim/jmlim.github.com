---
layout: post
title:  "[Spring] Spring Controller 에서 외부 URL로 redirect 하기."
date:   2019-09-30 16:10:00 +0900
categories: Spring
comments: true
tags: [Spring, Spring boot, Redirect, External, 외부 URL, Java]
---

---

Spring 웹 어플리케이션 내의 페이지가 아닌 외부 URL로 redirect 하고 싶을 경우 아래와 같이 다양한 방법으로 redirect 가 가능하다.

```java

import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.view.RedirectView;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;


@Controller
public class ExRedirectController {
   // return string
   @GetMapping("/ex_redirect1")
   public String exRedirect() {
       return "redirect:http://www.naver.com";
   }
   // return ModelAndView
   @GetMapping("/ex_redirect2")
   public ModelAndView exRedirectAnother() {
       String projectUrl = "redirect:http://www.naver.com";
       return new ModelAndView("redirect:" + projectUrl);
   }
   //
   @GetMapping("/ex_redirect3")
   public void redirectToTwitter(HttpServletResponse httpServletResponse) throws IOException {
       httpServletResponse.sendRedirect("https://naver.com");
   }
   @RequestMapping("/ex_redirect4")
   public RedirectView localRedirect() {
       RedirectView redirectView = new RedirectView();
       redirectView.setUrl("http://www.naver.com");
       return redirectView;
   }
   @RequestMapping("/ex_redirect5")
   public ResponseEntity<Object> redirectToExternalUrl() throws URISyntaxException {
       URI yahoo = new URI("http://www.naver.com");
       HttpHeaders httpHeaders = new HttpHeaders();
       httpHeaders.setLocation(yahoo);
       return new ResponseEntity<>(httpHeaders, HttpStatus.SEE_OTHER);
   }
}
```

### 참고자료 : 
 - https://code-examples.net/ko/q/111fbc1
 - https://stackoverflow.com/questions/17955777/redirect-to-an-external-url-from-controller-action-in-spring-mvc

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

---
