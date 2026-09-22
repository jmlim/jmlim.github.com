---
layout: post
title:  "Spring Boot Actuator, 운영 환경에서 어디까지 노출해도 될까?"
date:   2026-09-22 16:00:00 +0900
categories: Spring
comments: true
tags: [Spring, Spring Boot, Actuator, 보안, Kubernetes, Health Check]
---

---

이 블로그의 [MSA 회복탄력성 패턴](/msa/2026/09/22/msa-resilience-patterns/) 글에서 서비스 디스커버리/헬스체크 얘기를 잠깐 했었는데, 실제로 Spring Boot 애플리케이션에서 그 헬스체크(`/actuator/health`)를 어디까지 열어둬야 하는지는 또 별개의 고민이다.

Spring Boot 서비스를 운영하다 보면 `/actuator/health`를 사용하는 경우가 많다.

특히 Kubernetes, AWS ALB, 모니터링 시스템 등에서 애플리케이션이 정상적으로 살아있는지 확인하기 위해 Health Check 용도로 많이 사용한다.

그런데 Actuator 설정을 하다 보면 이런 설정을 만나게 된다.

```yaml
management:
  endpoints:
    web:
      discovery:
        enabled: false
```

처음 보면 이런 생각이 든다.

> `/actuator`가 노출되면 보안상 위험한 건가?
> `discovery.enabled: false`는 반드시 설정해야 하나?

결론부터 말하면 **꼭 필요한 설정은 아니다.**

더 중요한 것은 `/actuator`라는 경로 자체를 숨기는 것이 아니라, **어떤 Actuator Endpoint를 실제로 외부에 노출하고 있는지 관리하는 것**이다.

## 1. `/actuator`에 접속하면 무엇이 나올까?

Spring Boot Actuator가 활성화되어 있다면 기본적으로 다음과 같은 경로를 사용할 수 있다.

```text
/actuator
/actuator/health
```

`/actuator`에 접속하면 환경에 따라 다음과 비슷한 응답을 볼 수 있다.

```json
{
  "_links": {
    "self": {
      "href": "https://example.com/actuator"
    },
    "health": {
      "href": "https://example.com/actuator/health"
    }
  }
}
```

여기서 중요한 점이 있다.

`/actuator`가 애플리케이션의 내부 정보를 전부 보여주는 것은 아니다.

쉽게 말하면 `/actuator`는

> "현재 웹으로 접근할 수 있는 Actuator Endpoint는 이런 것들이 있습니다."

라는 **Endpoint 탐색 정보(Discovery Page)** 를 제공한다.

## 2. 그렇다면 `discovery.enabled: false`는 무엇일까?

다음 설정을 추가해보자.

```yaml
management:
  endpoints:
    web:
      discovery:
        enabled: false
```

이 설정의 역할은 **Actuator Endpoint 자체를 비활성화하는 것이 아니다.**

`/actuator`에서 제공하는 Endpoint 목록, 즉 Discovery Page를 비활성화한다.

예를 들어 기존에

```text
GET /actuator
```

를 호출했을 때 다음과 같은 정보가 보였다면,

```json
{
  "_links": {
    "health": {
      "href": "https://example.com/actuator/health"
    }
  }
}
```

`discovery.enabled: false`를 적용하면 `/actuator`에서 이런 목록을 제공하지 않게 된다.

하지만 중요한 점이 있다.

```text
/actuator/health
```

가 함께 비활성화되는 것은 아니다.

즉,

```text
/actuator
    ↓
Endpoint 목록 제공 안 함

/actuator/health
    ↓
여전히 접근 가능
```

이라고 이해하면 된다.

`discovery.enabled: false`는 **Endpoint를 막는 설정이라기보다는 Endpoint 목록을 보여주지 않는 설정**이다.

## 3. 그러면 보안상 중요한 설정은 무엇일까?

개인적으로 Actuator 설정에서 더 중요하게 봐야 할 부분은 `discovery`보다 **exposure**다.

예를 들어 다음 설정을 보자.

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health
```

이 설정은 웹을 통해 노출할 Endpoint를 `health`로 제한한다.

둘의 차이를 정리하면 다음과 같다.

| 설정                              | 역할                                 |
| ------------------------------- | ---------------------------------- |
| `discovery.enabled: false`      | `/actuator`에서 Endpoint 목록을 보여주지 않음 |
| `exposure.include: health`      | 웹으로 접근 가능한 Endpoint를 `health`로 제한  |
| `endpoint.health.enabled: true` | Health Endpoint 자체를 활성화            |
| `health.show-details: never`    | Health의 상세 정보를 숨김                  |

즉 보안 관점에서는

```yaml
exposure:
  include: health
```

가 훨씬 본질적인 설정이다.

## 4. 왜 Exposure 설정이 더 중요할까?

Actuator에는 Health 외에도 다양한 Endpoint가 존재한다.

대표적으로 다음과 같은 것들이 있다.

```text
/actuator/env
/actuator/beans
/actuator/configprops
/actuator/mappings
/actuator/loggers
/actuator/heapdump
```

이런 Endpoint가 운영 환경에 불필요하게 노출되면 애플리케이션 구조나 설정 등에 대한 정보를 외부에 제공할 수 있다.

따라서 중요한 것은

> `/actuator`라는 주소를 알고 있느냐

보다는

> 실제로 어떤 Endpoint에 접근할 수 있느냐

이다.

예를 들어 `/actuator`에서 Health Endpoint가 존재한다는 사실을 확인할 수 있다고 하더라도,

```text
/actuator/health
```

하나만 접근할 수 있다면 공격자가 얻을 수 있는 정보는 상당히 제한적이다.

반대로 `/actuator` Discovery Page를 숨겼다고 해도

```text
/actuator/env
/actuator/heapdump
/actuator/mappings
```

등이 실제로 접근 가능한 상태라면 Endpoint 주소를 직접 알고 있는 요청까지 막아주는 것은 아니다.

그래서 **목록을 숨기는 것보다 실제 Endpoint의 노출 범위를 제한하는 것이 더 중요하다.**

## 5. Health도 상세 정보는 숨기자

Health Endpoint를 공개하더라도 상세 정보까지 공개할 필요가 없는 경우가 많다.

예를 들어 다음과 같은 정보가 노출된다고 생각해보자.

```json
{
  "status": "UP",
  "components": {
    "db": {
      "status": "UP",
      "details": {
        "database": "Oracle",
        "validationQuery": "isValid()"
      }
    },
    "redis": {
      "status": "UP"
    }
  }
}
```

외부 Health Check의 목적이 단순히

> 애플리케이션이 정상인가?

를 확인하는 것이라면 이런 상세 정보까지 보여줄 필요가 없다.

따라서 운영 환경에서는 다음처럼 설정할 수 있다.

```yaml
management:
  endpoint:
    health:
      show-details: never
```

그러면 응답을 최소한으로 유지할 수 있다.

```json
{
  "status": "UP"
}
```

이 정도면 ALB나 모니터링 시스템에서 Health Check를 수행하기에도 충분한 경우가 많다.

> **[2026년 추가]** `show-details`의 기본값 자체가 `never`다 — 즉 아무 설정도 안 해도 기본적으로는 상세 정보가 숨겨져 있다. 이 값을 명시적으로 써두는 건 "우리 팀은 의도적으로 숨긴 것"이라는 걸 코드로 남겨두는 의미가 크다.

## 6. 운영 환경에서 보수적으로 설정한다면

Health만 필요한 서비스라면 다음처럼 구성할 수 있다.

```yaml
management:
  endpoints:
    enabled-by-default: false
    web:
      exposure:
        include: health

  endpoint:
    health:
      enabled: true
      show-details: never
```

각 설정을 하나씩 보면 이해하기 쉽다.

```yaml
enabled-by-default: false
```

Actuator Endpoint를 기본적으로 활성화하지 않는다.

그리고

```yaml
endpoint:
  health:
    enabled: true
```

필요한 Health Endpoint만 활성화한다.

마지막으로

```yaml
web:
  exposure:
    include: health
```

Health만 HTTP를 통해 노출한다.

결과적으로 의도는 명확하다.

```text
Actuator Endpoint 기본 비활성화
        ↓
Health만 활성화
        ↓
웹에서도 Health만 노출
        ↓
Health 상세 정보는 숨김
```

> **[2026년 추가]** 위 `enabled-by-default` / `endpoint.health.enabled`는 **Spring Boot 3.4부터 deprecated** 되었다. Boot 3.4에서 Actuator의 접근 제어 모델 자체가 단순 on/off에서 `none` / `read-only` / `unrestricted` 3단계 접근 수준으로 바뀌면서, 아래 두 프로퍼티로 대체됐다.
>
> - `management.endpoints.enabled-by-default` → `management.endpoints.access.default`
> - `management.endpoint.<id>.enabled` → `management.endpoint.<id>.access`
>
> 같은 의도를 최신 방식으로 쓰면 다음과 같다.
>
> ```yaml
> management:
>   endpoints:
>     access:
>       default: none        # 모든 엔드포인트 접근을 기본적으로 막음 (opt-in)
>     web:
>       exposure:
>         include: health
>
>   endpoint:
>     health:
>       access: read-only     # health만 읽기 접근 허용
>       show-details: never
> ```
>
> `access.default: none`으로 "기본 차단, 필요한 것만 opt-in" 하는 구조 자체는 동일하다. 최근 버전으로 새로 프로젝트를 시작한다면 `enabled-by-default`가 아니라 이 `access` 기반 설정을 쓰는 게 맞다. (물론 `exposure.include`로 HTTP 노출 자체를 제한하는 건 이 변경과 별개로 여전히 유효하다.)

## 7. 그럼 `discovery.enabled: false`도 추가해야 할까?

여기서 처음 질문으로 돌아가 보자.

```yaml
management:
  endpoints:
    web:
      discovery:
        enabled: false
```

**반드시 추가해야 하는 설정이라고 보기는 어렵다.**

이미

```yaml
exposure:
  include: health
```

로 Health만 노출하고 있고,

```yaml
show-details: never
```

로 Health 상세 정보까지 숨겼다면 `/actuator` Discovery Page에서 얻을 수 있는 정보 자체가 많지 않다.

물론 보안 정책상

> 외부에서 Actuator Endpoint의 존재 자체를 최대한 노출하지 않는다.

라는 기준이 있다면 추가할 수 있다.

```yaml
management:
  endpoints:
    access:
      default: none
    web:
      discovery:
        enabled: false
      exposure:
        include: health

  endpoint:
    health:
      access: read-only
      show-details: never
```

하지만 여기서 기억해야 할 것은

```text
discovery.enabled: false
```

가 `/actuator/health` 접근 자체를 막아주는 보안 설정은 아니라는 것이다.

## 8. 적용 후에는 직접 확인해보자

설정을 변경했다면 실제 운영 환경과 동일한 Profile로 애플리케이션을 실행하고 직접 호출해보는 것이 가장 확실하다.

### Health 확인

```bash
curl -i http://localhost:8080/actuator/health
```

정상적으로

```json
{
  "status": "UP"
}
```

정도만 반환되는지 확인한다.

### 불필요한 Endpoint 확인

```bash
curl -i http://localhost:8080/actuator/env
curl -i http://localhost:8080/actuator/beans
curl -i http://localhost:8080/actuator/configprops
curl -i http://localhost:8080/actuator/mappings
curl -i http://localhost:8080/actuator/heapdump
```

여기서 중요한 것은 **이 Endpoint들에 실제로 접근할 수 없는지 확인하는 것**이다.

설정 파일만 보고

> 아마 안 열려 있겠지.

라고 생각하는 것보다 실제 HTTP 요청으로 확인하는 것이 좋다.

## 9. 정리

Actuator 보안 설정을 처음 보면 `/actuator`라는 경로 자체를 숨겨야 할 것처럼 느껴질 수 있다.

하지만 조금 나눠서 보면 생각보다 단순하다.

```text
discovery
    ↓
어떤 Endpoint가 있는지 목록을 보여줄 것인가?

exposure
    ↓
어떤 Endpoint를 HTTP로 접근할 수 있게 할 것인가?

access (구 enabled)
    ↓
해당 Endpoint에 어느 수준까지 접근을 허용할 것인가?

show-details
    ↓
Health 내부 정보를 어디까지 보여줄 것인가?
```

따라서 Health Check만 필요한 운영 서비스라면 우선 신경 써야 할 부분은 다음과 같다. (Spring Boot 3.4+ 기준)

```yaml
management:
  endpoints:
    access:
      default: none
    web:
      exposure:
        include: health

  endpoint:
    health:
      access: read-only
      show-details: never
```

그리고 `/actuator`의 Endpoint 목록까지 굳이 보여줄 필요가 없거나 사내 보안 정책에서 요구한다면 추가로

```yaml
management:
  endpoints:
    web:
      discovery:
        enabled: false
```

를 적용하면 된다.

결국 핵심은 **"Actuator를 숨겼느냐"가 아니라 "필요한 Endpoint만 실제로 열어두었느냐"**다.

`/actuator`에서 Health Endpoint가 있다는 사실이 보이는 것보다 `/env`, `/mappings`, `/heapdump` 같은 불필요한 Endpoint가 실제로 열려 있는지가 훨씬 중요한 점검 포인트다.

## 참고자료
- [Spring Boot Reference - Monitoring and Management over HTTP (Discovery Page)](https://docs.spring.io/spring-boot/reference/actuator/endpoints.html)
- [Spring Boot 3.4 Release Notes - Controlling Access to Actuator Endpoints](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.4-Release-Notes)

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
