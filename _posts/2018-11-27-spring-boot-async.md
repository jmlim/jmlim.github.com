---
layout: post
title:  "스프링에서 @Async로 비동기처리하기"
date:   2018-11-27 18:00:00 +0900
categories: Spring
comments: true
tags: [Spring, Java]
---

---
### 스프링에서 @Async로 비동기처리하기

####언제 사용하면 좋을까?
1. 요청이 긴 경우
2. 로그 처리
3. 푸시 처리

#####Async 기능 켜기 Enable Async Support
자바 설정(Java configuration) 관련 클래스에 @EnableAsync 를 추가해주기만 하면 된다.

```java
@Configuration
@EnableAsync
public class SpringAsyncConfig { ... }
```
#####Config

@EnableAsync만 추가하면 기본적인 설정은 끝이다.<br/>
하지만, 기본값인 SimpleAsyncTaskExecutor 클래스는 매번 Thread를 만들어내는 객체이기 때문에 Thread Pool이 아니다. <br/>
Thread Pool을 설정해기 위해 AsyncConfigurerSupport를 상속받아 재구현하자.

```java
@Configuration
@EnableAsync
public class SpringAsyncConfig extends AsyncConfigurerSupport {
	@Override
	public Executor getAsyncExecutor() {
		ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
		executor.setCorePoolSize(2);
		executor.setMaxPoolSize(10);
		executor.setQueueCapacity(500);
		executor.setThreadNamePrefix("heowc-async-");
		executor.initialize();
		return executor;
	}
}
```

##### 선언 후 호출하기.

어노테이션 선언
- 비동기 작업을 하기 위한 메소드에 @Async를 추가하면 된다. 
  만약 callback이 필요하다면, Future 클래스 등의 객체로 감싸서 반환하면 된다.

```java
@Component
@Slf4j
public class MessageSender {
 /**
   * @param to
   * @param subject
   * @param text
   */
  @Async
  public void sendSimpleMessage(String to, String subject, String text) {
     sendSimpleMessageSync(to, subject, text);
  }

  public void sendSimpleMessageSync(String to, String subject, String text) {
    .....
  }
}
```

호출은 아래와 같이 하면 된다.

```java
 
 @Autowired
 private MessageSender messageSender;

 public void sendEmail(String originCd, String subject, String to, String companyCd, String productCd, String reservCd, String templateID, String dailCode, Map valueMap){
       ...
       messageSender.sendSimpleMessage(to, subject, html);
       ...
  }
```

출처: 
 - https://heowc.github.io/2018/02/10/spring-boot-async/


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
