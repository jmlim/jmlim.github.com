---
layout: post
title:  "자바 ExecutorService와 CompletableFuture 제대로 쓰기."
date:   2026-09-22 20:00:00 +0900
categories: Java
comments: true
tags: [Java, ExecutorService, CompletableFuture, Thread Pool, 동시성]
---

---

이 블로그의 [스프링에서 @Async로 비동기처리하기](/spring/2018/11/27/spring-boot-async/), 그리고 최근에 쓴 [Spring @Async는 실제로 어떻게 동작할까?](/spring/2026/09/22/spring-async-thread-pool-proxy/) 글에서 `@Async`가 결국 쓰레드풀에 작업을 던지는 것뿐이라고 정리했었는데, 그 밑바탕이 되는 자바의 `ExecutorService`와 `CompletableFuture`를 스프링 없이 순수 자바 레벨에서 정리해본다.

## 왜 Thread를 직접 만들면 안 되는가

```java
// 나쁜 예 - 요청마다 새 쓰레드 생성
new Thread(() -> doSomething()).start();
```

- 쓰레드 생성/소멸 자체가 비용이 크다 (OS 레벨 자원 할당).
- 몇 개까지 만들지 제한이 없어서, 트래픽이 몰리면 쓰레드가 무한정 늘어나다가 `OutOfMemoryError` 로 서버가 죽을 수 있다.
- → 그래서 쓰레드를 미리 만들어두고 재사용하는 **쓰레드 풀(Thread Pool)** 이 필요하고, 자바에서는 `ExecutorService` 로 이를 추상화한다.

## ExecutorService 기본

```java
ExecutorService executor = Executors.newFixedThreadPool(10); // 쓰레드 10개 고정 풀

executor.submit(() -> {
    // 비동기로 실행될 작업
    System.out.println("작업 실행: " + Thread.currentThread().getName());
});

executor.shutdown(); // 더 이상 새 작업을 받지 않고, 기존 작업이 끝나면 종료
```

### 자주 쓰는 팩토리 메서드 (Executors)

| 메서드 | 특징 | 주의점 |
|---|---|---|
| `newFixedThreadPool(n)` | 고정 크기 n개 쓰레드 | 큐가 무제한(`LinkedBlockingQueue`) — 요청 폭주 시 큐가 무한정 쌓여 메모리 문제 가능 |
| `newCachedThreadPool()` | 필요할 때마다 쓰레드 생성, 60초 유휴 시 회수 | 상한선이 없어서 트래픽 폭주 시 쓰레드가 무한정 늘어날 수 있음 |
| `newSingleThreadExecutor()` | 쓰레드 1개, 작업을 순차 처리 | 순서 보장이 필요한 작업에 적합 |
| `newScheduledThreadPool(n)` | 지연/주기 실행 지원 | [스프링부트 Scheduling](/spring/2018/11/28/spring-boot-schedule/) 글에서 다룬 `@Scheduled`의 기반 개념 |

실무에서는 위 팩토리 메서드보다 `ThreadPoolExecutor` 생성자를 직접 써서 `corePoolSize`, `maximumPoolSize`, `queueCapacity`, `RejectedExecutionHandler` 를 명시적으로 지정하는 걸 권장한다.

- 이유: 기본값(`Integer.MAX_VALUE` 큐)을 그대로 쓰면, 트래픽이 몰려도 max 쓰레드까지 안 늘어나고 큐에만 계속 쌓이는 현상이 생길 수 있다. **실제로 스케일 인(scale-in) 직후 이 문제 때문에 지연 장애를 겪은 적이 있다** — 파드 개수가 줄어든 상태에서 `@Async`가 기본 쓰레드 수(코어풀 사이즈 1)만 쓰고 있었고, 몰린 요청이 죄다 큐에 쌓이기만 하면서 처리 지연이 눈덩이처럼 커졌었다. Spring의 `ThreadPoolTaskExecutor`도 결국 이 `ThreadPoolExecutor`를 감싼 것이라, `corePoolSize`/`maxPoolSize`/`queueCapacity`를 명시적으로 잡아주는 게 왜 중요한지는 [Async 심화 글](/spring/2026/09/22/spring-async-thread-pool-proxy/)에서 더 자세히 다뤘다.

## Future — 결과를 나중에 받기

```java
ExecutorService executor = Executors.newFixedThreadPool(4);

Future<Integer> future = executor.submit(() -> {
    Thread.sleep(1000);
    return 42;
});

// future.get() 은 결과가 준비될 때까지 현재 쓰레드를 블로킹함 (Async지만 결과를 기다리는 부분은 Blocking)
Integer result = future.get();
```

`Future`의 한계
- `get()`을 호출하는 순간 블로킹된다 — 완전한 논블로킹이 아니다.
- 여러 작업을 연결(체이닝)하거나, 실패 시 콜백을 걸거나, 여러 Future를 조합하는 게 번거롭다.
- → 이 한계를 보완하기 위해 Java 8부터 `CompletableFuture`가 등장했다.

## CompletableFuture — 논블로킹 콜백 체이닝

```java
CompletableFuture<Integer> future = CompletableFuture
    .supplyAsync(() -> {          // 1. 비동기로 값을 만들어냄 (별도 쓰레드풀에서 실행)
        return fetchUserCount();
    })
    .thenApply(count -> count * 2)      // 2. 결과를 받아 가공 (콜백 - 블로킹 없이 연결)
    .thenAccept(result -> System.out.println("결과: " + result)) // 3. 최종 소비
    .exceptionally(ex -> {              // 4. 예외 처리
        System.out.println("에러 발생: " + ex.getMessage());
        return null;
    });
```

`get()`으로 결과를 기다리는 대신, "결과가 나오면 이 콜백을 실행해줘"라는 방식으로 코드를 짤 수 있어서 호출 쓰레드를 블로킹하지 않는다. 동기/비동기가 "순서"의 문제이고 블로킹/논블로킹이 "제어권을 바로 돌려주느냐"의 문제라면, `CompletableFuture`는 정확히 **비동기 + 논블로킹** 조합에 해당한다.

여러 비동기 작업을 조합하는 예:
```java
CompletableFuture<User> userFuture = CompletableFuture.supplyAsync(() -> getUser(id));
CompletableFuture<List<Order>> ordersFuture = CompletableFuture.supplyAsync(() -> getOrders(id));

// 두 작업이 모두 끝나면 합쳐서 처리
CompletableFuture<UserDetail> combined = userFuture.thenCombine(ordersFuture,
    (user, orders) -> new UserDetail(user, orders));
```

기본적으로 `supplyAsync`, `thenApply` 등은 `ForkJoinPool.commonPool()`이라는 공용 풀을 사용한다. 실무에서는 **반드시 전용 Executor를 두 번째 인자로 넘겨서** 다른 기능과 쓰레드풀을 공유하지 않도록 하는 게 안전하다.

```java
CompletableFuture.supplyAsync(() -> fetchUserCount(), myExecutor);
```

- 전용 Executor를 안 넘기면, 애플리케이션 안의 서로 무관한 여러 비동기 작업들이 전부 `ForkJoinPool.commonPool()`이라는 같은 풀을 공유하게 된다. 한쪽 작업이 오래 걸리면 다른 쪽 작업까지 그 풀의 쓰레드를 기다리게 되는데, 이건 MSA 회복탄력성 패턴에서 다룬 [Bulkhead(격벽) 패턴](/msa/2026/09/22/msa-resilience-patterns/)이 막으려는 문제와 정확히 같은 모양이다 — 호출 대상별로 쓰레드풀을 분리해야 한쪽 장애가 다른 쪽까지 안 번진다.

## Spring @Async와의 관계

[스프링에서 @Async로 비동기처리하기](/spring/2018/11/27/spring-boot-async/)에서 다룬 `@Async`는, 내부적으로 이 `ExecutorService`(스프링에서는 `ThreadPoolTaskExecutor`)에 작업을 던지는 걸 어노테이션으로 감싸놓은 것이다. `@Async` 메서드가 값을 반환해야 한다면 리턴 타입을 `CompletableFuture<T>`로 선언하면, 호출부에서 결과를 논블로킹으로 조합할 수 있다.

## 정리

- 쓰레드를 직접 만들지 말고 `ExecutorService`(쓰레드 풀)로 재사용한다.
- 팩토리 메서드(`newFixedThreadPool` 등)보다 `ThreadPoolExecutor`를 직접 구성해서 큐/최대 쓰레드 수를 명시적으로 관리하는 게 운영 환경에서 안전하다.
- 결과를 기다려야 한다면 `Future`보다 `CompletableFuture`로 논블로킹 콜백 체이닝을 쓰는 게 낫다.
- `CompletableFuture`를 쓸 때는 공용 풀(`ForkJoinPool.commonPool()`)을 쓰지 말고 전용 Executor를 넘겨서, 다른 기능과 쓰레드풀 자원을 두고 서로 발목 잡는 일을 막는다.

## 참고자료
- [Java 공식 문서 - ExecutorService](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ExecutorService.html)
- [Java 공식 문서 - CompletableFuture](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CompletableFuture.html)
- 이 블로그의 [스프링에서 @Async로 비동기처리하기](/spring/2018/11/27/spring-boot-async/), [Spring @Async는 실제로 어떻게 동작할까?](/spring/2026/09/22/spring-async-thread-pool-proxy/)

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
