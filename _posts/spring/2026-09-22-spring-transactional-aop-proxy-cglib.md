---
layout: post
title:  "Spring Boot @Transactional은 어떻게 동작할까? AOP Proxy부터 CGLIB까지."
date:   2026-09-22 11:00:00 +0900
categories: Spring
comments: true
tags: [Spring, AOP, Transactional, Proxy, CGLIB, JDK Dynamic Proxy]
---

---

예전에 [Spring 에서의 트랜잭션 처리](/spring/2018/12/07/spring-transaction/) 글에서 트랜잭션의 개념과 propagation 옵션을 정리했었는데, 그때는 "`@Transactional`을 붙이면 된다"는 사용법 위주였다. 이번엔 그 안에서 실제로 무슨 일이 일어나는지 — **Spring AOP Proxy의 동작 원리** — 를 파고들어 본다.

Spring을 사용하다 보면 자연스럽게 이런 코드를 작성하게 된다.

```java
@Service
public class OrderService {

    @Transactional
    public void order() {
        // DB 작업
    }
}
```

`@Transactional`을 붙이면 메서드 실행 중 예외가 발생했을 때 롤백되고, 정상적으로 끝나면 커밋된다.

그런데 여기서 한 단계 더 들어가 보면 몇 가지 궁금증이 생긴다.

* `@Transactional`은 어떻게 메서드 실행 전후에 트랜잭션을 처리할까?
* Spring AOP와 Proxy는 무슨 관계일까?
* JDK Dynamic Proxy와 CGLIB는 뭐가 다를까?
* 왜 자기 자신의 `@Transactional` 메서드를 호출하면 동작하지 않을까?
* 예전에는 `public`이어야 한다고 했는데 왜 Spring Boot 3.x에서는 package-private(default)도 동작할까?
* `private`은 왜 여전히 안 될까?

이 글에서는 이 질문들을 **Spring AOP Proxy의 동작 원리**를 중심으로 정리해본다.

## 1. `@Transactional` 자체가 트랜잭션을 시작하는 것은 아니다

먼저 가장 중요한 부분이다.

```java
@Transactional
public void order() {
    orderRepository.save(...);
}
```

`@Transactional` 어노테이션 자체가 트랜잭션을 시작하고 커밋하는 것은 아니다.

Spring이 해당 객체 앞에 **Proxy 객체를 만들어 놓고**, Proxy가 메서드 호출을 가로채 트랜잭션을 처리한다.

개념적으로 보면 다음과 같다.

```text
Controller
    │
    ▼
┌──────────────────┐
│   Spring Proxy   │
├──────────────────┤
│ Transaction 시작 │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ 실제 OrderService │
│     order()       │
└────────┬─────────┘
         │
    ┌────┴─────┐
    ▼          ▼
 정상 종료    예외 발생
    │          │
 COMMIT     ROLLBACK
```

실제 Spring 내부 구현은 훨씬 복잡하지만 개념적으로는 다음과 비슷하다고 생각하면 된다.

```java
public void order() {

    transactionManager.begin();

    try {
        target.order();

        transactionManager.commit();
    } catch (Exception e) {
        transactionManager.rollback();
        throw e;
    }
}
```

개발자가 직접 `begin`, `commit`, `rollback`을 작성하는 대신 `@Transactional`을 선언하면 Spring이 AOP를 통해 대신 처리해주는 것이다.

## 2. 그렇다면 AOP란 무엇일까?

AOP는 **Aspect-Oriented Programming**, 즉 관점 지향 프로그래밍이다.

이름만 보면 어렵지만 Spring을 사용하는 입장에서는 간단하게 생각할 수 있다.

> 여러 비즈니스 로직에서 반복되는 공통 기능을 실제 비즈니스 코드와 분리하는 방법

예를 들어 다음 메서드가 있다고 해보자.

```java
order();
cancel();
refund();
```

모든 메서드에서 트랜잭션 처리가 필요하다면 직접 구현할 경우 다음과 같은 코드가 반복된다.

```java
public void order() {
    begin();

    // 주문 처리

    commit();
}

public void cancel() {
    begin();

    // 취소 처리

    commit();
}
```

Spring에서는 대신 다음과 같이 작성한다.

```java
@Transactional
public void order() {
    // 주문 처리
}

@Transactional
public void cancel() {
    // 취소 처리
}
```

트랜잭션이라는 공통 관심사는 Proxy에게 맡기고 Service에는 비즈니스 로직을 남기는 것이다.

Spring에서는 이러한 AOP를 구현하는 주요 방법 중 하나로 **Proxy 패턴**을 사용한다.

## 3. Proxy란 무엇일까?

Proxy는 말 그대로 **대리 객체**다.

실제 Service를 바로 호출하지 않고 중간에 대리인을 하나 둔다고 생각하면 쉽다.

```text
호출자
  │
  ▼
Proxy
  │
  ▼
실제 Service
```

Proxy는 실제 메서드를 호출하기 전후로 추가적인 작업을 수행할 수 있다.

```text
order() 호출
     │
     ▼
@Transactional 확인
     │
     ▼
Transaction 시작
     │
     ▼
실제 order() 호출
     │
 ┌───┴────┐
 ▼        ▼
정상      예외
 │        │
commit  rollback
```

이 구조 덕분에 `@Transactional`뿐 아니라 여러 Spring 기능이 AOP Proxy를 활용할 수 있다.

대표적으로 다음과 같은 기능들이 있다.

```java
@Transactional
@Cacheable
@Async
```

## 4. Spring Proxy에는 두 가지 대표적인 방식이 있다

Spring AOP Proxy를 공부하면 반드시 등장하는 것이 있다.

* JDK Dynamic Proxy
* CGLIB Proxy

둘의 목적은 동일하다.

> 실제 객체 앞에 Proxy를 만들어 메서드 호출을 가로챈다.

하지만 **Proxy 객체를 만드는 방법이 다르다.**

가장 간단하게 정리하면 다음과 같다.

|                | JDK Dynamic Proxy  | CGLIB Proxy  |
| -------------- | ------------------ | ------------ |
| 방식             | 인터페이스 구현           | 클래스 상속       |
| 핵심 키워드         | `implements`       | `extends`    |
| 인터페이스          | 필요                 | 없어도 가능       |
| final class    | 영향 없음              | Proxy 생성 불가  |
| final method   | 클래스 override 방식 아님 | AOP 적용 불가    |
| private method | 인터페이스 메서드가 될 수 없음  | override 불가능 |

핵심은 이것이다.

> **JDK Dynamic Proxy = Interface 기반**
>
> **CGLIB Proxy = Class 상속 기반**

## 5. JDK Dynamic Proxy

다음과 같은 Service가 있다고 해보자.

```java
public interface OrderService {

    void order();
}
```

구현체가 존재한다.

```java
@Service
public class OrderServiceImpl implements OrderService {

    @Transactional
    @Override
    public void order() {
        // 주문 처리
    }
}
```

JDK Dynamic Proxy는 동일한 인터페이스를 구현하는 Proxy를 만든다.

개념적으로는 다음과 같다.

```java
class OrderServiceProxy implements OrderService {

    private OrderService target;

    @Override
    public void order() {

        // Transaction 시작

        target.order();

        // Transaction commit
    }
}
```

구조를 보면 다음과 같다.

```text
              OrderService
              (interface)
               ▲       ▲
               │       │
      implements       implements
               │       │
       ┌───────┘       └─────────┐
       │                         │
  JDK Proxy              OrderServiceImpl
       │                         ▲
       └─────────────────────────┘
                 호출
```

즉 JDK Dynamic Proxy는 실제 클래스를 상속하는 것이 아니라 **같은 인터페이스를 구현하는 별도의 객체를 만드는 방식**이다.

## 6. CGLIB Proxy

CGLIB는 접근 방법이 다르다.

```java
@Service
public class OrderService {

    @Transactional
    public void order() {
        // 주문 처리
    }
}
```

CGLIB는 실제 클래스를 **상속**하여 Proxy를 만든다.

개념적으로 보면 다음과 비슷하다.

```java
class OrderService$$SpringCGLIB$$0 extends OrderService {

    @Override
    public void order() {

        // Transaction 시작

        super.order();

        // Transaction commit
    }
}
```

구조는 훨씬 단순하다.

```text
        OrderService
             ▲
             │
           extends
             │
OrderService$$SpringCGLIB$$0
```

실제 런타임에서 Bean의 클래스를 출력해 보면 환경에 따라 다음과 비슷한 이름을 볼 수도 있다.

```text
OrderService$$SpringCGLIB$$0
```

이것이 Spring이 만들어 놓은 Proxy 객체다.

## 7. CGLIB에서 `final`이 문제가 되는 이유

CGLIB의 원리를 알고 나면 `final`이 문제가 되는 이유도 자연스럽게 이해된다.

CGLIB의 핵심은

```java
class Proxy extends OrderService
```

이기 때문이다.

따라서 다음 클래스는 상속할 수 없다.

```java
public final class OrderService {
}
```

즉 개념적으로 다음 코드가 불가능하다.

```java
class Proxy extends OrderService {
    // compile error
}
```

메서드 역시 마찬가지다.

```java
public final void order() {
}
```

자식 클래스에서 `final` 메서드를 override할 수 없다.

따라서 CGLIB가 해당 메서드를 가로채 AOP 기능을 적용할 수 없다.

결국 CGLIB의 제약사항 상당수는 Spring의 특별한 규칙이라기보다 **Java 상속의 제약사항에서 나온다.**

## 8. 가장 유명한 함정: Self Invocation

Proxy의 동작 원리를 이해하면 `@Transactional`의 유명한 문제도 이해할 수 있다.

다음 코드를 보자.

```java
@Service
public class OrderService {

    public void order() {
        saveOrder();
    }

    @Transactional
    public void saveOrder() {
        // DB 작업
    }
}
```

겉으로 보면 `saveOrder()`에 `@Transactional`이 있으니 트랜잭션이 시작될 것 같다.

하지만 일반적인 Spring Proxy 기반 AOP에서는 그렇지 않다.

외부에서 `order()`를 호출할 때는 Proxy를 거친다.

```text
Controller
    │
    ▼
Proxy
    │
    ▼
OrderService.order()
```

하지만 `order()` 안에서

```java
saveOrder();
```

를 호출하는 것은 사실상

```java
this.saveOrder();
```

와 같다.

따라서 호출 구조가 다음과 같이 된다.

```text
Controller
    │
    ▼
Proxy
    │
    ▼
OrderService.order()
    │
    │ this.saveOrder()
    ▼
OrderService.saveOrder()
```

`saveOrder()` 호출 과정에서 Proxy를 다시 거치지 않는다.

따라서 Proxy가 `@Transactional`을 확인하고 새로운 트랜잭션 처리를 수행할 기회도 없다.

이것이 **Self Invocation 문제**다.

## 9. CGLIB면 Self Invocation이 되지 않을까?

처음 CGLIB의 상속 구조를 알게 되면 이런 생각이 들 수 있다.

> CGLIB는 실제 클래스를 상속해서 Proxy를 만드는데 내부 호출도 override된 메서드를 타지 않을까?

하지만 Spring의 일반적인 proxy 기반 AOP를 사용할 때는 **self-invocation을 트랜잭션 경계로 기대하면 안 된다.**

실무에서는 다음 원칙으로 이해하는 것이 가장 안전하다.

> `@Transactional`은 외부에서 Spring Proxy를 통해 들어오는 호출을 트랜잭션 경계로 잡는다.

그래서 트랜잭션을 별도의 Service로 분리하는 패턴을 자주 사용한다.

```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final OrderSaveService orderSaveService;

    public void order() {
        orderSaveService.saveOrder();
    }
}
```

```java
@Service
public class OrderSaveService {

    @Transactional
    public void saveOrder() {
        // DB 작업
    }
}
```

그러면 호출 구조가 명확해진다.

```text
OrderService
     │
     ▼
OrderSaveService Proxy
     │
     ▼
Transaction 시작
     │
     ▼
OrderSaveService.saveOrder()
```

## 10. 그런데 왜 최신 Spring에서는 `public`이 아니어도 될까?

여기서 처음에 가졌던 의문으로 돌아온다.

다음 코드가 있다고 해보자.

```java
@Service
public class OrderService {

    @Transactional
    void save() {
        // package-private
    }
}
```

접근제어자를 작성하지 않았으므로 `save()`는 Java에서 **package-private(default)** 메서드다.

과거 Spring 관련 자료를 보면 흔히

> `@Transactional`은 public 메서드에 사용해야 한다.

라고 설명한다.

하지만 Spring Framework 6부터는 **클래스 기반 Proxy에서 `protected`와 package-private 메서드도 기본적으로 트랜잭션 메서드로 사용할 수 있다.**

Spring Boot 3.x가 Spring Framework 6 계열을 사용하기 때문에 최근 프로젝트에서는 package-private `@Transactional`이 정상적으로 동작하는 모습을 볼 수 있다.

## 11. package-private이 가능한 이유

여기서 CGLIB의 동작 원리를 다시 생각해보면 재미있다.

다음 클래스가 있다고 하자.

```java
package com.example.order;

@Service
public class OrderService {

    @Transactional
    void save() {
    }
}
```

CGLIB Proxy를 아주 단순화해서 표현하면 다음과 비슷하다.

```java
package com.example.order;

class OrderService$$SpringCGLIB$$0 extends OrderService {

    @Override
    void save() {

        // Transaction 처리

        super.save();
    }
}
```

Java에서 package-private 메서드는 **같은 패키지에서 접근 가능하다.**

따라서 같은 패키지에 정의된 CGLIB 기반 Proxy가 해당 메서드를 override하고 가로챌 수 있는 것이다.

```text
com.example.order

 ├── OrderService
 │       │
 │       └── void save()
 │
 └── OrderService$$SpringCGLIB$$0
         │
         └── override save()
```

이 부분을 이해하면 Spring 6에서 package-private 트랜잭션 메서드가 가능한 이유가 훨씬 자연스럽게 이해된다.

## 12. 그런데 `private`은 왜 여전히 안 될까?

다음과 같이 작성했다고 해보자.

```java
@Service
public class OrderService {

    @Transactional
    private void save() {
    }
}
```

CGLIB가 같은 패키지에 Proxy를 만든다고 해도 이 메서드는 가로챌 수 없다.

왜냐하면 Java에서 `private` 메서드는 자식 클래스에서 override할 수 없기 때문이다.

즉 다음과 같은 구조 자체가 성립하지 않는다.

```java
class OrderServiceProxy extends OrderService {

    @Override
    private void save() {
        // 불가능
    }
}
```

따라서 클래스 기반 Proxy를 기준으로 보면 접근제어자에 따른 차이는 대략 다음과 같이 이해할 수 있다.

| 접근제어자           | Proxy 가능 여부 | 이유                         |
| --------------- | ----------: | -------------------------- |
| `public`        |           O | override 가능                |
| `protected`     |           O | override 가능                |
| package-private |           O | 조건을 만족하는 클래스 기반 Proxy에서 가능 |
| `private`       |           X | override 불가능               |
| `final` method  |           X | override 불가능               |

## 13. 접근제어자와 Self Invocation은 별개의 문제다

여기서 특히 주의해야 한다.

다음 코드가 있다고 하자.

```java
@Service
public class OrderService {

    public void order() {
        save();
    }

    @Transactional
    void save() {
        // DB 작업
    }
}
```

Spring 6에서 package-private 메서드가 트랜잭션 대상이 될 수 있다는 것과 이 코드에서 트랜잭션이 적용되는지는 **별개의 문제**다.

두 가지를 분리해서 생각해야 한다.

### 첫 번째: 이 메서드를 Proxy가 가로챌 수 있는가?

```java
@Transactional
void save()
```

Spring 6의 클래스 기반 Proxy에서는 가능하다.

### 두 번째: 실제 호출이 Proxy를 거치는가?

```java
public void order() {
    save();
}
```

이 호출은 self-invocation이다.

```text
OrderService.order()
       │
       │ this.save()
       ▼
OrderService.save()
```

Proxy를 통한 새로운 호출이 아니므로 `save()`의 `@Transactional`을 새로운 트랜잭션 경계로 기대해서는 안 된다.

즉 다음 두 질문을 항상 따로 해야 한다.

```text
1. 이 메서드는 Proxy가 가로챌 수 있는 메서드인가?

                +

2. 실제 호출이 Proxy를 통해 들어오는가?
```

둘 다 만족해야 Proxy 기반 AOP를 제대로 이해할 수 있다.

## 14. Spring Boot에서는 JDK Proxy와 CGLIB 중 무엇을 사용할까?

전통적인 Spring AOP 설명에서는 흔히 다음과 같이 설명한다.

```text
Interface 있음
      ↓
JDK Dynamic Proxy

Interface 없음
      ↓
CGLIB
```

Spring Framework의 기본적인 Proxy 선택 원리를 이해하는 데는 좋은 설명이다.

다만 Spring Boot에서는 설정에 따라 클래스 기반 Proxy를 기본으로 사용하는 환경이 일반적이므로,

> "인터페이스가 존재하면 무조건 JDK Dynamic Proxy다."

라고 단순하게 생각하면 실제 프로젝트에서 혼란이 생길 수 있다.

중요한 것은 현재 애플리케이션이 어떤 Proxy 전략을 사용하고 있는지 확인하는 것이다.

필요하다면 런타임에서 실제 Bean 타입을 확인해볼 수도 있다.

```java
System.out.println(orderService.getClass());
```

CGLIB Proxy라면 환경에 따라 다음과 비슷한 형태를 볼 수 있다.

```text
class com.example.order.OrderService$$SpringCGLIB$$0
```

## 15. 전체 흐름을 하나의 그림으로 정리해보자

결국 Spring의 `@Transactional` 동작 원리는 다음 그림으로 정리할 수 있다.

```text
                     Spring Container
                           │
                           ▼
                    Bean 생성 과정
                           │
                           ▼
                 @Transactional 발견
                           │
                           ▼
                     Proxy 생성
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
      JDK Dynamic Proxy             CGLIB Proxy
              │                         │
       implements Interface          extends Class
              │                         │
              └────────────┬────────────┘
                           │
                           ▼
                     외부 메서드 호출
                           │
                           ▼
                    Proxy가 가로챔
                           │
                           ▼
                   Transaction 시작
                           │
                           ▼
                    실제 Service 호출
                           │
                 ┌─────────┴─────────┐
                 ▼                   ▼
               정상                  예외
                 │                   │
              COMMIT              ROLLBACK
```

반면 self-invocation은

```text
Proxy
  │
  ▼
Service.methodA()
  │
  │ this.methodB()
  ▼
Service.methodB()
```

처럼 Proxy를 다시 거치지 않는다는 것이 핵심이다.

## 16. 실무에서 기억할 것

모든 내부 구현을 외울 필요는 없다.

Spring에서 `@Transactional`을 사용할 때는 다음 정도를 기억하면 대부분의 문제를 이해할 수 있다.

1. **`@Transactional` 자체가 트랜잭션을 실행하는 것이 아니다.**

   Spring AOP Proxy가 메서드 호출을 가로채 트랜잭션을 시작하고 종료한다.

2. **JDK Dynamic Proxy는 인터페이스 기반이다.**

   핵심은 `implements`다.

3. **CGLIB Proxy는 클래스 상속 기반이다.**

   핵심은 `extends`와 method overriding이다.

4. **CGLIB에서는 `final`을 주의해야 한다.**

   상속하거나 override할 수 없기 때문이다.

5. **Spring 6의 클래스 기반 Proxy에서는 package-private `@Transactional`도 사용할 수 있다.**

   따라서 Spring Boot 3.x에서는 `public`이 아니어도 트랜잭션이 정상적으로 동작하는 경우가 있다.

6. **하지만 `private` 메서드는 Proxy가 override할 수 없다.**

7. **접근제어자보다 더 자주 문제가 되는 것은 Self Invocation이다.**

   같은 객체 내부에서 `this.xxx()` 형태로 호출하면 일반적인 Spring Proxy 기반 AOP를 다시 거치지 않는다.

## 마무리

처음 `@Transactional`을 배울 때는 보통 이렇게 외운다.

```java
@Transactional
public void save() {
}
```

> "이렇게 붙이면 트랜잭션이 된다."

사용하는 데는 충분하지만, 조금 복잡한 코드를 만나면 금방 의문이 생긴다.

```java
@Transactional
void save() {
}
```

> "어? public이 아닌데 왜 되지?"

또는

```java
public void order() {
    save();
}

@Transactional
public void save() {
}
```

> "어? `@Transactional`을 붙였는데 왜 안 되지?"

이런 현상을 제대로 이해하려면 어노테이션 자체보다 **Proxy를 먼저 생각해야 한다.**

결국 핵심은 한 문장으로 정리할 수 있다.

> **Spring의 `@Transactional`은 메서드 자체에 트랜잭션 기능을 넣는 것이 아니라, Spring이 만든 Proxy가 메서드 호출을 가로채 트랜잭션을 시작하고 commit/rollback하는 AOP 기능이다.**

그리고 그 Proxy를 만드는 대표적인 방법이 **JDK Dynamic Proxy와 CGLIB**다.

이 원리를 알고 나면 `@Transactional`뿐 아니라 `@Cacheable`, `@Async` 등 Spring의 여러 AOP 기반 기능에서 발생하는 "어노테이션을 붙였는데 왜 동작하지 않지?"라는 문제도 훨씬 쉽게 이해할 수 있다.

## 참고자료
- [Spring Framework Reference - Using @Transactional (Method Visibility)](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html)
- [Spring Framework Reference - Proxying Mechanisms](https://docs.spring.io/spring-framework/reference/core/aop/proxying.html)
- 이 블로그의 [Spring 에서의 트랜잭션 처리](/spring/2018/12/07/spring-transaction/) - propagation, 트랜잭션 개념 기초편

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
