---
layout: post
title:  "Spring @Async는 실제로 어떻게 동작할까? Thread Pool부터 Self-Invocation까지."
date:   2026-09-22 15:00:00 +0900
categories: Spring
comments: true
tags: [Spring, Async, Thread Pool, Proxy, ThreadPoolTaskExecutor]
---

---

예전에 [스프링에서 @Async로 비동기처리하기](/spring/2018/11/27/spring-boot-async/) 글에서 기본 사용법을 정리했었는데, 그때는 `@EnableAsync` 붙이고 `ThreadPoolTaskExecutor` 설정하는 선에서 끝났다. 이번엔 그 안에서 실제로 무슨 일이 일어나는지 — Spring Proxy가 어떻게 비동기로 만들어주는지, Thread Pool은 왜 필요한지, self-invocation 문제와 트랜잭션/예외 처리까지 깊이 들어가 본다.

**핵심부터 말하면:** Spring의 `@Async`는 오래 걸리지만 요청 결과에 즉시 필요하지 않은 작업을 **현재 요청 Thread가 아닌 별도의 Thread에서 실행**하게 해주는 기능이다. 이메일·푸시·알림처럼 본 작업과 분리할 수 있는 작업에 유용하지만, 단순히 `@Async`만 붙이는 것으로 끝나는 것은 아니다. **Thread Pool, Spring Proxy, 예외 처리, 트랜잭션 경계**까지 함께 이해해야 안전하게 사용할 수 있다.

## 1. `@Async`를 왜 사용할까?

일반적인 Java 메서드 호출은 **동기(Synchronous)** 방식이다.

예를 들어 주문 처리 과정에서 다음 작업을 수행한다고 해보자.

```java
public void order() {
    saveOrder();      // 1초
    sendEmail();      // 3초
    sendPush();       // 2초
}
```

실행 순서는 다음과 같다.

```text
요청 Thread
   │
   ├─ saveOrder()   1초
   │
   ├─ sendEmail()   3초
   │
   ├─ sendPush()    2초
   │
   └─ 응답
       총 6초
```

`sendEmail()`과 `sendPush()`가 완료되어야만 주문이 성공하는 것이 아니라면 사용자가 이 작업까지 기다릴 이유는 없다.

이런 작업을 별도의 Thread에 맡기면 다음처럼 처리할 수 있다.

```text
HTTP 요청 Thread
   │
   ├─ saveOrder()
   │
   ├─ sendEmail() ──────────────┐
   │                            │
   └─ 응답                      │
                                ▼
                         Async Thread
                                │
                                └─ 이메일 발송
```

즉, `@Async`의 핵심은 다음 한 문장으로 정리할 수 있다.

> **호출한 Thread는 기다리지 않고 다음 작업을 계속하고, 실제 작업은 다른 Thread가 처리한다.**

### 언제 사용하면 좋을까?

대표적으로 다음과 같은 작업이 있다.

- 이메일 발송
- SMS 발송
- Push 알림
- 중요도가 낮은 로그/이력 저장
- 통계성 데이터 처리
- 응답과 분리해도 되는 외부 API 호출
- 시간이 오래 걸리지만 사용자가 결과를 기다릴 필요가 없는 작업

반대로 **결제, 재고 차감 등 반드시 성공 여부를 확인해야 하는 핵심 비즈니스 로직**을 아무 생각 없이 `@Async`로 넘기는 것은 위험하다.

## 2. 가장 간단한 사용 방법

Spring에서 비동기 처리를 활성화하려면 설정 클래스에 `@EnableAsync`를 선언한다.

```java
@Configuration
@EnableAsync
public class AsyncConfig {
}
```

그리고 비동기로 실행하고 싶은 Spring Bean의 메서드에 `@Async`를 붙인다.

```java
@Service
public class MessageSender {

    @Async
    public void sendMessage(String message) {
        System.out.println(message);
    }
}
```

다른 Bean에서 호출한다.

```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final MessageSender messageSender;

    public void order() {
        saveOrder();

        messageSender.sendMessage("주문 완료");
    }

    private void saveOrder() {
        // 주문 저장
    }
}
```

호출하는 코드만 보면 평범한 메서드 호출과 거의 동일하다.

하지만 내부에서는 Spring이 `sendMessage()`를 별도의 실행기에 넘겨 비동기로 처리한다.

## 3. `@Async`는 어떻게 비동기가 되는 걸까?

`@Async`를 이해할 때 가장 중요한 것은 **Spring Proxy**다.

Spring Bean에 `@Async`가 있다고 해서 Java 자체가 해당 메서드를 특별하게 실행하는 것은 아니다.

Spring이 Bean 앞에 Proxy를 두고 메서드 호출을 가로챈다.

```text
OrderService
     │
     │ messageSender.sendMessage()
     ▼
┌────────────────────┐
│ Spring Proxy       │
│                    │
│ @Async 확인        │
└─────────┬──────────┘
          │
          │ 작업 제출
          ▼
┌────────────────────┐
│ TaskExecutor       │
│                    │
│ async-thread-1     │
│ async-thread-2     │
└─────────┬──────────┘
          │
          ▼
 MessageSender
 .sendMessage()
```

개념적으로 보면 Spring이 다음과 비슷한 일을 대신 해준다고 생각할 수 있다.

```java
executor.submit(() -> {
    messageSender.sendMessage();
});
```

실제 구현은 더 복잡하지만 학습 단계에서는 이렇게 이해하면 충분하다.

### `@Transactional`과 비슷하다

`@Transactional` 역시 Proxy 기반으로 동작한다. ([Spring @Transactional은 어떻게 동작할까?](/spring/2026/09/22/spring-transactional-aop-proxy-cglib/) 글에서 자세히 다뤘다.)

```java
@Transactional
public void save() {
}
```

개념적으로는 다음과 같다.

```text
호출
 ↓
Spring Proxy
 ↓
Transaction 시작
 ↓
save()
 ↓
commit / rollback
```

`@Async`도 같은 관점으로 보면 된다.

```text
호출
 ↓
Spring Proxy
 ↓
TaskExecutor에 작업 전달
 ↓
별도 Thread
 ↓
실제 메서드 실행
```

따라서 `@Async`를 이해하면 Spring AOP와 Proxy에 대한 이해도 자연스럽게 연결된다.

## 4. Thread Pool은 왜 필요한가?

비동기 요청이 들어올 때마다 Thread를 무한정 새로 만든다고 생각해보자.

```text
요청 1     → Thread 생성
요청 2     → Thread 생성
요청 3     → Thread 생성
...
요청 10,000 → Thread 생성
```

Thread 생성에는 비용이 든다.

Thread마다 메모리가 필요하고 Thread 수가 지나치게 많아지면 Context Switching 비용도 증가한다.

그래서 일반적으로 일정 개수의 Thread를 만들어 놓고 재사용한다.

이것이 **Thread Pool**이다.

```text
             작업 Queue
                 │
        ┌────────┴────────┐
        │                 │
        ▼                 ▼
     Thread-1          Thread-2
        │                 │
      작업 A             작업 B
        │                 │
      작업 C             작업 D
```

Spring에서는 대표적으로 `ThreadPoolTaskExecutor`를 사용할 수 있다.

## 5. `ThreadPoolTaskExecutor` 설정하기

명시적으로 비동기 전용 Executor를 구성하면 어떤 Thread Pool을 사용하는지 코드에서 확인하기 쉽다.

```java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean(name = "asyncExecutor")
    public Executor asyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();

        executor.setCorePoolSize(2);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(500);
        executor.setThreadNamePrefix("async-");

        executor.initialize();

        return executor;
    }
}
```

그리고 다음처럼 사용할 Executor를 지정할 수 있다.

```java
@Async("asyncExecutor")
public void sendMessage() {
    // 비동기 작업
}
```

### `corePoolSize`

```java
executor.setCorePoolSize(2);
```

기본적으로 작업을 처리하는 Thread 수라고 이해하면 된다.

```text
Thread Pool

Thread-1
Thread-2
```

### `maxPoolSize`

```java
executor.setMaxPoolSize(10);
```

작업량이 증가했을 때 Thread Pool이 확장할 수 있는 최대 Thread 수다.

```text
평상시

Thread-1
Thread-2

부하 증가

Thread-1
Thread-2
Thread-3
...
Thread-10
```

### `queueCapacity`

```java
executor.setQueueCapacity(500);
```

모든 Thread가 작업 중일 때 새로운 작업을 대기시키는 Queue의 크기다.

여기서 중요한 점이 있다.

**작업이 많아졌다고 바로 `maxPoolSize`까지 Thread가 증가하는 것은 아니다.**

대략 다음 순서로 동작한다.

```text
1. corePoolSize까지 Thread 사용
            ↓
2. 추가 작업을 Queue에 저장
            ↓
3. Queue가 가득 참
            ↓
4. maxPoolSize까지 Thread 증가
            ↓
5. Thread와 Queue가 모두 한계에 도달
            ↓
6. 새로운 작업 거절
```

예를 들어 다음 설정을 사용한다고 하자.

```text
corePoolSize  = 2
maxPoolSize   = 10
queueCapacity = 500
```

작업이 한꺼번에 들어오고 앞선 작업들이 아직 끝나지 않았다고 단순화하면:

```text
1번 작업   → Thread-1
2번 작업   → Thread-2

3 ~ 502번  → Queue 대기

503번 작업
      ↓
Queue가 가득 참
      ↓
추가 Thread 생성
```

따라서 `maxPoolSize`만 크게 설정한다고 처리량이 바로 증가하는 것은 아니다.

Thread Pool 크기는 **CPU 작업인지, I/O 작업인지, 작업 시간이 얼마나 긴지, 초당 요청량이 어느 정도인지** 등을 고려해 정해야 한다.

## 6. 실제 예제

이메일 발송을 비동기로 처리한다고 해보자.

```java
@Service
@Slf4j
public class MessageSender {

    @Async("asyncExecutor")
    public void sendSimpleMessage(
            String to,
            String subject,
            String text) {

        log.info(
            "send mail. thread={}",
            Thread.currentThread().getName()
        );

        sendSimpleMessageSync(to, subject, text);
    }

    private void sendSimpleMessageSync(
            String to,
            String subject,
            String text) {

        // 실제 이메일 발송
    }
}
```

호출하는 서비스는 다음과 같다.

```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final MessageSender messageSender;

    public void order() {

        saveOrder();

        messageSender.sendSimpleMessage(
                "test@test.com",
                "주문 완료",
                "주문이 완료되었습니다."
        );
    }

    private void saveOrder() {
        // 주문 저장
    }
}
```

실행 Thread는 다음과 같이 분리될 수 있다.

```text
http-nio-8080-exec-1
        │
        │ order()
        │
        ├─ saveOrder()
        │
        └─ sendSimpleMessage()
                 │
                 │ @Async
                 ▼
              async-1
                 │
                 └─ 이메일 발송
```

`order()`를 실행하는 요청 Thread가 이메일 발송 완료를 기다리지 않는 것이 핵심이다.

## 7. 가장 많이 하는 실수: 같은 클래스 내부 호출

다음 코드를 보자.

```java
@Service
public class MessageSender {

    public void send() {
        sendAsync();
    }

    @Async
    public void sendAsync() {
        // 비동기 작업
    }
}
```

겉으로 보면 `sendAsync()`에 `@Async`가 있으므로 비동기로 실행될 것 같다.

하지만 기본 Proxy 방식에서는 기대한 대로 동작하지 않는다.

내부 호출은 사실상 다음과 같기 때문이다.

```java
this.sendAsync();
```

호출 흐름은:

```text
MessageSender
     │
     │ send()
     ▼
 sendAsync()
```

중간에 Spring Proxy가 없다.

따라서 `@Async`를 처리할 기회도 없다.

이를 **self-invocation 문제**라고 한다.

다른 Bean에서 호출하면 Proxy를 통과한다.

```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final MessageSender messageSender;

    public void order() {
        messageSender.sendAsync();
    }
}
```

```text
OrderService
     │
     ▼
Spring Proxy
     │
     │ @Async 확인
     ▼
TaskExecutor
     │
     ▼
별도 Thread
     │
     ▼
MessageSender.sendAsync()
```

`@Transactional`에서 같은 클래스 내부 호출이 문제가 되는 이유와 같은 맥락이다.

## 8. 반환값이 필요하면 `CompletableFuture`

비동기 작업의 결과가 필요하다면 `CompletableFuture`를 사용할 수 있다.

```java
@Async("asyncExecutor")
public CompletableFuture<String> sendMessage() {

    // 작업 수행

    return CompletableFuture.completedFuture("SUCCESS");
}
```

호출하는 쪽에서는:

```java
CompletableFuture<String> future =
        messageSender.sendMessage();
```

결과가 완료된 뒤 후속 작업을 연결할 수도 있다.

```java
future.thenAccept(result -> {
    log.info("result={}", result);
});
```

단순한 이메일, Push, 알림처럼 호출자가 결과를 받을 필요가 없다면 `void`도 충분하다.

## 9. 비동기 예외 처리는 다르다

다음 메서드에서 예외가 발생한다고 생각해보자.

```java
@Async
public void sendMessage() {
    throw new RuntimeException("메일 발송 실패");
}
```

호출 Thread와 실행 Thread는 다르다.

```text
요청 Thread

messageSender.sendMessage()
        │
        │ 작업 전달
        ▼
      return


Async Thread

sendMessage()
      │
      └─ Exception 발생
```

따라서 호출부에서 다음처럼 작성해도 비동기 Thread에서 발생한 예외를 일반적인 방식으로 잡을 수 없다.

```java
try {
    messageSender.sendMessage();
} catch (Exception e) {
    // @Async void 메서드 내부 예외를
    // 이런 방식으로 처리할 수 있다고 생각하면 안 된다.
}
```

`void` 반환 `@Async` 메서드의 예외는 호출자에게 직접 전달할 수 없으므로 별도의 예외 처리 전략이 필요하다.

예를 들어:

- 비동기 메서드 내부에서 로깅
- `AsyncUncaughtExceptionHandler` 구성
- 재시도 정책 적용
- 실패 데이터를 별도 저장
- 중요한 작업이라면 메시지 큐 도입

등을 고려할 수 있다.

## 10. `@Transactional`과 같이 사용할 때 주의

다음 코드를 보자.

```java
@Transactional
public void order() {

    Order order = saveOrder();

    messageSender.sendAsync(order.getId());
}
```

개발자는 자연스럽게 다음 순서를 기대할 수 있다.

```text
주문 저장
   ↓
COMMIT
   ↓
비동기 작업
```

하지만 반드시 그런 것은 아니다.

실제로는 다음과 같은 상황이 가능하다.

```text
Thread A                     Thread B

Transaction 시작

INSERT Order
     │
sendAsync() ────────────────→ 실행
     │                         │
     │                         └─ Order 조회
     │
COMMIT
```

비동기 Thread가 먼저 실행되면 아직 Transaction이 Commit되지 않은 데이터를 조회하려 할 수도 있다.

따라서 **DB Commit 이후에 반드시 실행되어야 하는 작업**이라면 단순히 `@Transactional` 메서드 안에서 `@Async`를 호출하는 것만으로는 부족할 수 있다.

이럴 때는 예를 들어:

```java
@TransactionalEventListener(
    phase = TransactionPhase.AFTER_COMMIT
)
```

등을 이용해 **Commit 이후 이벤트를 처리하는 구조**를 고려할 수 있다.

## 11. `@Async`와 메시지 큐는 다르다

`@Async`는 편리하지만 작업의 영속성을 보장하는 메시징 시스템은 아니다.

예를 들어:

```text
요청
 ↓
@Async 작업 등록
 ↓
아직 작업 실행 전
 ↓
서버 프로세스 종료
```

같은 상황에서는 작업이 유실될 가능성을 고려해야 한다.

따라서 다음과 같이 구분해서 생각하는 것이 좋다.

### `@Async`가 잘 맞는 경우

```text
실패해도 재시도가 절대적으로 필요하지 않음
작업 유실 가능성을 어느 정도 허용할 수 있음
같은 애플리케이션 안에서 간단하게 비동기로 처리하고 싶음
```

### Kafka / RabbitMQ 등의 메시징을 고려할 경우

```text
작업 유실이 허용되지 않음
재처리가 중요함
대량의 비동기 작업을 처리함
서비스 간 비동기 통신이 필요함
```

즉,

> `@Async`는 **간단한 애플리케이션 내부 비동기 처리**에 매우 편리하지만, 신뢰성 있는 메시징 시스템을 대체하는 기능은 아니다.

## 12. 2018년 코드에서 바뀐 부분

예전 글에서는 다음처럼 `AsyncConfigurerSupport`를 상속하는 예제를 썼었다.

```java
@Configuration
@EnableAsync
public class SpringAsyncConfig
        extends AsyncConfigurerSupport {

    @Override
    public Executor getAsyncExecutor() {
        // ...
    }
}
```

하지만 **Spring Framework 6.0부터 `AsyncConfigurerSupport`는 deprecated 되었고**, 필요한 경우 `AsyncConfigurer`를 직접 구현하는 방식이 권장된다. (Javadoc에도 "as of 6.0 in favor of implementing AsyncConfigurer directly"라고 명시되어 있다.)

또한 단순히 특정 비동기 작업용 Executor를 만들고 싶다면 다음처럼 Bean을 명시적으로 등록하는 방식도 이해하기 쉽다.

```java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean(name = "asyncExecutor")
    public Executor asyncExecutor() {

        ThreadPoolTaskExecutor executor =
                new ThreadPoolTaskExecutor();

        executor.setCorePoolSize(2);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(500);
        executor.setThreadNamePrefix("async-");
        executor.initialize();

        return executor;
    }
}
```

그리고:

```java
@Async("asyncExecutor")
public void sendMessage() {
}
```

처럼 어떤 Executor를 사용하는지 명확하게 표현할 수 있다.

참고로 최신 Spring Boot에서는 별도의 `Executor`가 없을 때 비동기 작업에 사용할 실행기를 자동 구성해주는 기능도 있으므로, **"Spring에서는 무조건 `SimpleAsyncTaskExecutor`가 기본이다"라고 단정하기보다는 Spring Framework 자체의 기본 동작과 Spring Boot의 Auto Configuration을 구분해서 이해하는 것이 좋다.**

## 13. 전체 동작 구조

지금까지의 내용을 한 장으로 정리하면 다음과 같다.

```text
                 HTTP Request
                      │
                      ▼
                OrderService
                      │
                      │
           messageSender.send()
                      │
                      ▼
               Spring Proxy
                      │
                 @Async 확인
                      │
                      ▼
             TaskExecutor
                      │
                      ▼
          ThreadPoolTaskExecutor
                      │
            ┌─────────┼─────────┐
            ▼         ▼         ▼
        Thread-1  Thread-2    Queue
            │         │
            ▼         ▼
        이메일 발송   Push 발송
```

가장 중요한 흐름은 이것이다.

```text
@Async
   ↓
Spring Proxy
   ↓
TaskExecutor
   ↓
Thread Pool
   ↓
별도의 Thread에서 실제 메서드 실행
```

## 14. 핵심 정리

`@Async`를 공부할 때 단순히

```java
@Async
public void send() {
}
```

만 기억하면 실제 운영 환경에서 문제가 생겼을 때 원인을 찾기 어렵다.

다음 다섯 가지를 함께 기억하는 것이 중요하다.

### 1. `@Async`는 다른 Thread에서 실행한다

호출자는 비동기 작업의 완료를 기다리지 않고 다음 작업을 진행할 수 있다.

### 2. 실제 비동기 처리는 Spring Proxy가 담당한다

그래서 같은 객체 내부에서 메서드를 호출하는 **self-invocation**에서는 주의가 필요하다.

### 3. Thread는 무한하지 않다

`ThreadPoolTaskExecutor`의 `corePoolSize`, `maxPoolSize`, `queueCapacity` 관계를 이해해야 한다.

### 4. Thread가 다르면 예외와 트랜잭션도 분리해서 생각해야 한다

특히 `@Transactional`과 함께 사용할 때 Commit 시점에 주의해야 한다.

### 5. 중요한 비동기 작업은 `@Async`만으로 부족할 수 있다

작업 유실 방지와 재처리가 중요하다면 Kafka, RabbitMQ 등의 메시징 시스템도 고려해야 한다.

## 마무리

Spring의 `@Async`는 사용법 자체는 매우 간단하다.

```java
@Async
public void sendMessage() {
}
```

하지만 이 한 줄 뒤에는 다음 개념들이 숨어 있다.

```text
Spring AOP / Proxy
        ↓
TaskExecutor
        ↓
Thread Pool
        ↓
Multi Thread
        ↓
Transaction / Exception
```

따라서 `@Async`를 제대로 이해한다는 것은 단순히 비동기 어노테이션 하나를 배우는 것이 아니라 **Spring이 Proxy를 통해 부가기능을 적용하는 방식과 Java의 Thread Pool이 실제로 어떻게 동작하는지를 함께 이해하는 것**이라고 볼 수 있다.

## 참고자료
- [Spring Framework Reference - Task Execution and Scheduling](https://docs.spring.io/spring-framework/reference/integration/scheduling.html)
- [Spring Framework Javadoc - AsyncConfigurerSupport (deprecated)](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/annotation/AsyncConfigurerSupport.html)
- 이 블로그의 [스프링에서 @Async로 비동기처리하기](/spring/2018/11/27/spring-boot-async/) - 기본 사용법편
- 이 블로그의 [Spring @Transactional은 어떻게 동작할까?](/spring/2026/09/22/spring-transactional-aop-proxy-cglib/) - 같은 Proxy 원리를 다룬 짝 글

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
