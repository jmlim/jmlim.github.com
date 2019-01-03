---
layout: post
title:  "스프링부트에서 Scheduling 사용하기"
date:   2018-11-28 00:06:00 +0900
categories: Spring
comments: true
tags: [Spring, Java]
---

---


스케줄링은 특정 기간 동안 작업을 실행하는 프로세스이다. 
Spring Boot를 통해 Spring에서 지원하는 스케줄러를 간편하게 작성할 수 있다.



Schedule 기능 켜기
--
자바 설정(Java configuration) 관련 클래스에 @EnableScheduling 를 추가하면 기능을 사용할 수 있다.


```java
@SpringBootApplication
@EnableScheduling
public class DemoApplication {
   public static void main(String[] args) {
      SpringApplication.run(DemoApplication.class, args);
   }
}
```

구현하기
--

@Scheduled 어노테이션을 메소드에 선언하며 실행이 가능하며 실행주기는 cron, fixedDelay, fixedRate 라는 세개의 속성으로 지정할 수 있다.

cron 으로 실행주기 설정하기. 
```
그 전에crontab 주기설정 방법부터 알아보자.

*　　　　　　*　　　　　　*　　　　　　*　　　　　　*
분(0-59)　　시간(0-23)　　일(1-31)　　월(1-12)　　　요일(0-7) 
각 별 위치에 따라 주기를 다르게 설정 할 수 있다.
순서대로 분-시간-일-월-요일 순이다. 그리고 괄호 안의 숫자 범위 내로 별 대신 입력 할 수도 있다.
요일에서 0과 7은 일요일이며, 1부터 월요일이고 6이 토요일이다.
```

cron 속성 지정하여 사용. 
```java
package 패키지위치;

import java.text.SimpleDateFormat;
import java.util.Date;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class Scheduler {
   @Scheduled(cron = "0 * 9 * * ?")
   public void cronJobSch() {
      SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.SSS");
      Date now = new Date();
      String strDate = sdf.format(now);
      System.out.println("Java cron job expression:: " + strDate);
   }
}
```

다음 예제는 고정된 시간이 5초마다 호출되고 이 시간은 이전 호출이 완료된 시점부터 계산될 것이다.
```java
// 1초에 한번 실행된다.
@Scheduled(fixedDelay = 1000) 
public void scheduleFixedRateTask() {
    System.out.println(
      "Fixed rate task - " + System.currentTimeMillis() / 1000);
}
```

고정된 비율로 실행하기를 원한다면 어노테이션에서 지정한 프로퍼티명만 바꿔주면 된다. 다음 예제는 각 호출의 연속적인 시작 시각의 간격으로 계산된 5초마다 실행될 것이다.
```java
@Scheduled(fixedRate = 1000)
public void scheduleFixedRateTask() {
  // 주기적으로 실행될 것이다
}
```

> 위 차이를 쉽게 얘기하면 fixedDelay 는 이전 수행이 종료된 시점부터 delay 후에 재 호출되고 fixedRate 는 이전 수행이 시작된 시점부터 delay 후에 재 호출된다. 그러므로 fixedRate 로 지정 시 동시에 여러개가 돌 가능성이 존재한다.

Thread pool 설정
--
기본적으로 모든 @Scheduled 작업은 Spring에 의해 생성 된 한개의 스레드 풀에서 실행된다.
그렇기 때문에 하나의 Scheduled이 돌고 있다면 그것이 다 끝나야 다음 Scheduled이 실행되는 문제가 있다.

실제로 로그를 보면 같은 쓰레드로 확인 될 것이다.
```java
logger.info("Current Thread : {}", Thread.currentThread().getName());
```

결과
```java
Current Thread : pool-1-thread-1
```
스프링 부트에서 설정을 통해 Schedule에 대한 쓰래드 풀을 생성하고 그 쓰레드 풀을 사용하여 모든 스케줄 된 작업을 실행하도록 할 수 있다.

아래는 설정 방법에 대한 예제이다.

```java
@Configuration
public class SchedulerConfig implements SchedulingConfigurer {
    private final int POOL_SIZE = 10;

    @Override
    public void configureTasks(ScheduledTaskRegistrar scheduledTaskRegistrar) {
        ThreadPoolTaskScheduler threadPoolTaskScheduler = new ThreadPoolTaskScheduler();

        threadPoolTaskScheduler.setPoolSize(POOL_SIZE);
        threadPoolTaskScheduler.setThreadNamePrefix("my-scheduled-task-pool-");
        threadPoolTaskScheduler.initialize();

        scheduledTaskRegistrar.setTaskScheduler(threadPoolTaskScheduler);
    }
}
```

스케줄이 돌고있는 메소드에서 현재 스레드의 이름을 로깅하면 아래와 같은 출력이 표시된다.

```java
Current Thread : my-scheduled-task-pool-1
Current Thread : my-scheduled-task-pool-2
```


참고: 
 - https://blog.outsider.ne.kr/1066
 - https://jdm.kr/blog/2
 - https://www.callicoder.com/spring-boot-task-scheduling-with-scheduled-annotation/


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
