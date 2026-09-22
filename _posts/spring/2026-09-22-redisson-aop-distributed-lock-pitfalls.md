---
layout: post
title:  "AOP로 감싼 Redisson 분산 락, 내가 직접 짠 코드에서 발견한 버그 두 개."
date:   2026-09-22 23:00:00 +0900
categories: Spring
comments: true
tags: [Spring, AOP, Redisson, 분산락, 동시성, IllegalMonitorStateException]
---

---

개인 학습용으로 짜둔 동시성 락 비교 예제 코드를 오랜만에 다시 열어봤다. Optimistic Lock, Pessimistic Lock, MySQL Named Lock, Redis 기반 락(Lettuce/Redisson)을 각각 구현해서 비교해보는 코드인데, 그중 `@RedissonLock` 이라는 어노테이션 하나로 메서드에 분산 락을 걸 수 있게 만든 AOP `@Around` advice에서 실제로 동작하는 버그를 두 개 발견했다. 둘 다 "동시성 테스트는 통과하는데 실전에서는 위험한" 유형이라 기록해둔다.

## 문제의 코드

대략 이런 구조였다.

```java
@Around("@annotation(RedissonLock)")
public void redissonLock(ProceedingJoinPoint joinPoint) throws Throwable {
    RLock lock = redissonClient.getLock(lockKey);

    boolean lockable;
    try {
        lockable = lock.tryLock(waitTime, leaseTime, TimeUnit.MILLISECONDS);
    } catch (InterruptedException e) {
        throw new RuntimeException(e);
    }

    if (!lockable) {
        log.info("Lock 획득 실패 = {}", lockKey);
    }

    try {
        joinPoint.proceed(); // 반환값을 버림
    } finally {
        lock.unlock();
    }
}
```

## 버그 1 — 락을 못 잡았는데도 로직이 그대로 실행됨

`if (!lockable) { ... }` 블록 안에서 로그만 남기고 **`return`을 하지 않는다.** 그래서 락 획득에 실패해도 코드는 그대로 아래로 흘러내려가 `joinPoint.proceed()`를 호출한다. 즉 "동시에 하나의 스레드만 실행되게 하겠다"는 락의 목적이, 락 획득에 실패하는 순간 완전히 무력화된다.

더해서 메서드가 `void`로 선언되어 있어서, 원래 대상 메서드가 값을 반환하더라도 AOP를 거치면 호출부는 항상 `null`을 받게 되는 문제도 같이 있었다.

수정은 간단하다 — 실패 시 확실히 끝내고, 반환 타입도 `Object`로 바꿔서 실제 결과를 전달하게 했다.

```java
@Around("@annotation(RedissonLock)")
public Object redissonLock(ProceedingJoinPoint joinPoint) throws Throwable {
    RLock lock = redissonClient.getLock(lockKey);

    boolean lockable;
    try {
        lockable = lock.tryLock(waitTime, leaseTime, TimeUnit.MILLISECONDS);
    } catch (InterruptedException e) {
        throw new RuntimeException(e);
    }

    if (!lockable) {
        log.info("Lock 획득 실패 = {}", lockKey);
        return null; // 락을 못 잡았으면 원래 로직을 아예 실행하지 않는다
    }

    try {
        return joinPoint.proceed(); // 반환값을 그대로 전달
    } finally {
        lock.unlock();
    }
}
```

## 버그 2 — 락을 못 잡았는데 unlock()을 호출하면?

같은 프로젝트의 다른 Facade 클래스(순수 Redisson 락, AOP 없이 직접 호출하는 버전)에는 또 다른 패턴의 버그가 있었다.

```java
boolean available = lock.tryLock(15, 1, TimeUnit.SECONDS);

if (!available) {
    log.info("lock 획득 실패");
}

try {
    stockService.decrease(id, quantity);
} finally {
    lock.unlock(); // available이 false여도 항상 호출됨
}
```

`available`이 `false`인 경우, 즉 락을 획득하지 못한 경우에도 `finally` 블록에서 `lock.unlock()`이 호출된다. Redisson 공식 문서를 확인해보면 이건 단순히 "의미 없는 호출" 정도가 아니라 실제로 예외를 던질 수 있는 상황이다.

> Only lock owner thread can unlock it otherwise `IllegalMonitorStateException` would be thrown.

즉 **현재 스레드가 보유하지 않은 락을 `unlock()`하면 `IllegalMonitorStateException`이 발생한다.** 락 획득에 실패했다는 건 이 스레드가 그 락을 보유하지 않았다는 뜻이므로, 잘못하면 "락을 못 잡아서 실패" 로그를 남긴 직후 바로 런타임 예외로 한 번 더 터지는 상황이 된다.

수정은 락을 실제로 획득했을 때만 `try/finally`로 감싸는 것이다.

```java
boolean available;
try {
    available = lock.tryLock(15, 1, TimeUnit.SECONDS);
} catch (InterruptedException e) {
    throw new RuntimeException(e);
}

if (!available) {
    log.info("lock 획득 실패");
    return; // unlock()도 호출하면 안 된다
}

try {
    stockService.decrease(id, quantity);
} finally {
    lock.unlock(); // 여기 도달했다는 건 락을 실제로 획득했다는 뜻
}
```

## 왜 기존 테스트는 이걸 못 잡았나

이 프로젝트엔 이미 동시성 테스트가 있었다. 스레드 풀을 만들어 100개 요청을 동시에 날리고, 최종 수량이 예상값과 일치하는지 확인하는 방식이다.

```java
ExecutorService executorService = Executors.newFixedThreadPool(32);
// ... 100개 요청을 동시에 실행 후
assertThat(stock.getQuantity()).isEqualTo(originQuantity - CONCURRENT_COUNT);
```

문제는 `tryLock`의 `waitTime`이 15초로 꽤 넉넉하게 잡혀 있었다는 점이다. 가벼운 로직을 다루는 테스트 환경에서 100개 스레드가 짧은 시간 안에 락을 주고받다 보면, **대부분 15초 안에 어떻게든 차례가 돌아와서 락 획득에 성공**한다. 즉 `if (!lockable)` 분기, 버그가 실제로 있는 그 경로를 테스트가 실질적으로 거의 타지 않았던 것이다. 최종 수량만 검증하는 테스트는 "락이 제대로 걸렸다"를 증명하지, "락 획득에 실패하는 경로가 안전하다"까지는 증명해주지 않는다.

이런 종류의 버그를 잡으려면 `waitTime`을 아주 짧게(또는 0으로) 준 상태에서 일부러 락 경쟁을 유발하고, 실패 분기가 예외 없이 안전하게 빠져나가는지, 그리고 보호 대상 로직이 정말 실행되지 않았는지를 **직접 검증하는 별도 테스트**가 필요하다.

## 정리

- `@Around` advice는 `void`로 선언하지 말고 `joinPoint.proceed()`의 반환값을 그대로 돌려줘야 한다 — 안 그러면 대상 메서드의 반환값이 항상 사라진다.
- 락 획득 실패 시에는 반드시 그 자리에서 `return`해서 보호 대상 로직이 실행되지 않게 해야 한다.
- Redisson `RLock.unlock()`은 락을 실제로 보유한 스레드만 호출해야 한다 — 획득 실패 경로에서는 절대 호출하면 안 된다(`IllegalMonitorStateException`).
- 동시성 테스트를 짤 때는 "성공 경로가 정상 동작하는지"뿐 아니라 "실패(락 경합) 경로가 안전한지"도 별도로 검증해야 한다. `waitTime`이 넉넉하면 실패 경로 자체가 테스트에서 거의 실행되지 않을 수 있다.

## 참고자료
- [Redisson 공식 문서 - RLock](https://github.com/redisson/redisson/wiki/8.-distributed-locks-and-synchronizers#81-lock)
- [Spring 공식 문서 - AOP @Around Advice](https://docs.spring.io/spring-framework/reference/core/aop/ataspectj/advice.html)
- 이 블로그의 [Spring @Transactional은 어떻게 동작할까?](/spring/2026/09/22/spring-transactional-aop-proxy-cglib/), [Redis 자료구조와 캐시 전략 정리](/redis/2026/09/22/redis-data-structures-and-cache-strategy/)

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
