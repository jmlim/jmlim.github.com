---
layout: post
title:  "Spring RestController 에서 @PathVariable에 특수문자 포함 허용하기."
date:   2019-09-30 19:30:00 +0900
categories: Spring
comments: true
tags: [Spring, Spring boot, PathVariable, 특수문자, ., dot]
---

---


Spring RestController @PathVariable 사용하여 값을 넘겨받을 때 값에 "." 나 특수문자가 포함되어 있으면 해당 문자를 포함하여 뒤 문자열이 전부 잘려서 들어온다.

## 문자가 잘리는 예시

### 1. 요청 URL

```
GET http://localhost:8080/test/jmlim@2ss@aa.ddd
```

### 2. 로직
```java
@Slf4j
@RestController
public class TestController {
   @GetMapping("/test/{path}")
   public String notAllowSpecialCharacters(@PathVariable String path) {
	   log.info(path);
	   return path;
   }
}
```

### 3. 반환값
```
jmlim
```
> 특수문자포함 이후 문자열은 출력되지 않았다.


>  이걸 아래와 같이 바꿔주면 특수문자까지 포함하여 사용할 수 있다.

## 1. 특수문자 포함하여 출력

```java
@Slf4j
@RestController
public class TestController {
 
   @GetMapping("/test/{path:.+}")
   public String allowSpecialCharacters(@PathVariable String path) {
	   log.info(path);
	   return path;
   }

   @GetMapping("/test/{path}")
   public String notAllowSpecialCharacters(@PathVariable String path) {
	   log.info(path);
	   return path;
   }

}
```

### 2. 반환값
```
jmlim@2ss@aa.ddd
```

> 특수문자까지 포함되어 출력되었음을 확인하였다.

### 참고자료 : 
 - https://winmargo.tistory.com/202
 - https://stackoverflow.com/questions/16332092/spring-mvc-pathvariable-with-dot-is-getting-truncated
 - https://javamin.tistory.com/350

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

---
