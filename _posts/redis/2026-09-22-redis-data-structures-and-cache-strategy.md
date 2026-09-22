---
layout: post
title:  "Redis를 캐시로 제대로 사용하기 — 자료구조부터 장애 대응까지."
date:   2026-09-22 22:00:00 +0900
categories: Redis
comments: true
tags: [Redis, 캐시, Cache Strategy, Cache Stampede, Cache Penetration, Hot Key, TTL, RDB, AOF]
---

---

[Redis-cli 원격 접속하기](/redis/2019/06/19/redis-cli-remote-connect/), [Spring boot 환경에서 Spring Session을 통해 세션 저장하기](/spring/2018/11/30/spring-session-redis/) 글에서 Redis를 이미 써본 적은 있는데, 그때는 설치·연결·세션 저장 정도였다. 이번엔 조금 다른 관점에서 정리해본다.

> **Redis를 실제 서비스의 캐시로 쓴다면, 무엇까지 알아야 할까?**

`SET key value` / `GET key`로 값을 넣고 빼는 것 자체는 어렵지 않다. 하지만 실무에서는 금방 다른 질문들이 따라온다 — 어떤 자료구조를 써야 할지, TTL은 얼마로 줘야 할지, 서버가 재시작되면 데이터는 어떻게 되는지, 캐시가 한꺼번에 만료되면 무슨 일이 생기는지, Redis가 죽으면 서비스도 같이 죽어야 하는지, 캐시를 붙였는데 정말 효과가 있는지는 어떻게 확인하는지. 이번 글은 사용법보다 **"왜 그렇게 쓰는가"**에 초점을 맞춘다.

## 1. Redis의 핵심 자료구조

Redis는 단순 Key-Value 저장소처럼 보이지만, 실제로는 용도별로 골라 쓸 수 있는 다양한 자료구조를 제공한다.

| 타입 | 설명 | 대표 명령어 |
|---|---|---|
| String | 가장 기본. 문자열/숫자/직렬화된 객체 | `SET`, `GET`, `INCR` |
| List | 순서가 있는 문자열 목록 (연결 리스트) | `LPUSH`, `RPUSH`, `LRANGE` |
| Hash | 필드-값 쌍의 집합 (객체 하나를 표현하기 좋음) | `HSET`, `HGET`, `HGETALL` |
| Set | 중복 없는 집합, 순서 없음 | `SADD`, `SISMEMBER`, `SINTER`(교집합) |
| Sorted Set (ZSet) | 각 원소에 score를 매겨 정렬된 집합 | `ZADD`, `ZRANGE`, `ZRANK` |

### 좋아요 수 (String + INCR)
```
INCR post:1001:likes
```
`INCR`은 원자적으로 수행되기 때문에, 동시에 여러 요청이 들어와도 애플리케이션에서 `synchronized` 같은 처리를 따로 할 필요가 없다.

### 최근 조회 상품 (List)
```
LPUSH user:42:recent_view 1001
LTRIM user:42:recent_view 0 9   # 앞에서 10개만 남기고 나머지 삭제
```
새 항목을 앞에 추가하고, 최근 10개만 남긴다. `LTRIM`을 안 하면 뒤에서 다룰 "Big Key" 문제로 이어질 수 있다.

### 실시간 랭킹 (Sorted Set)
```
ZADD leaderboard 1500 "player1"
ZADD leaderboard 2300 "player2"
ZREVRANGE leaderboard 0 9 WITHSCORES   # 상위 10명 조회
```
관계형 DB라면 매번 `SELECT * FROM leaderboard ORDER BY score DESC LIMIT 10`을 실행해야 하는 것을, Sorted Set은 **정렬된 상태 자체를 자료구조 레벨에서 유지**한다 — 읽기가 압도적으로 빈번한 랭킹류 데이터에 잘 맞는다.

### 사용자 정보 (Hash)
```
HSET user:42 name "jmlim" email "example@example.com"
HGETALL user:42
```
객체 하나를 필드별로 관리하고 싶다면 Hash가 좋은 선택지다.

## 2. Redis도 메모리만 쓰는 게 아니다 — 영속성(Persistence)

Redis는 대표적인 인메모리 저장소지만, 서버가 죽거나 재시작돼도 데이터를 지키기 위한 두 가지 영속화 방식을 제공한다.

| 방식 | 동작 | 장점 | 단점 |
|---|---|---|---|
| **RDB** (스냅샷) | 특정 시점 전체 메모리 상태를 `.rdb` 파일로 저장 | 파일이 작고 복구가 빠름, 백업하기 좋음 | 마지막 스냅샷 이후 데이터는 유실될 수 있음 |
| **AOF** (Append Only File) | 쓰기 명령(`SET`, `INCR`, `HSET` ...)을 로그처럼 순서대로 기록, 복구 시 재실행 | 유실 가능성을 최소화 | 파일이 크고 복구 시간이 RDB보다 오래 걸림 |

실무에서는 둘을 **함께** 쓰는 경우가 많다(RDB로 주기적 스냅샷 + AOF로 세밀한 복구) — "쓰기 속도와 내구성은 트레이드오프"라는, 저널링을 쓰는 다른 DB들과 같은 고민이다.

그런데 어떤 걸 켜야 할지는 결국 Redis의 **용도**에 달려 있다.

- 원본이 이미 MySQL 같은 DB에 있고 Redis는 그 복사본을 빠르게 조회하기 위한 **순수 캐시**라면 → Redis 데이터가 통째로 사라져도 DB에서 다시 채우면 그만이다. 영속성을 꺼도 충분히 가능한 선택이다.
- 반면 세션, 랭킹, 작업 상태처럼 **Redis에 있는 데이터 자체가 원본**인 경우라면 → 유실을 감수할 수 없으므로 영속화나 복제 전략을 같이 설계해야 한다.

> 정리하면: "Redis를 쓰니까 무조건 RDB/AOF를 켠다/끈다"가 아니라, **Redis 데이터가 사라졌을 때 서비스가 실제로 어떤 문제를 겪는지**를 먼저 따져야 한다.

## 3. 가장 많이 쓰는 캐시 전략 — Cache-Aside

캐시를 쓸 때 가장 흔한 방식이 Cache-Aside(Lazy Loading)다. 조회 흐름은 이렇다.

```
Client → Application → Redis 조회
                          │
              ┌───────────┴───────────┐
             HIT                     MISS
              │                       │
            바로 반환              DB 조회 → Redis에 저장 → 반환
```

코드로 보면 단순하다.

```java
public Product getProduct(Long id) {
    String key = "product:" + id;
    Product cached = redisTemplate.opsForValue().get(key);
    if (cached != null) {
        return cached;                                 // Cache Hit
    }

    Product product = productRepository.findById(id)
            .orElseThrow();                             // Cache Miss -> DB 조회
    redisTemplate.opsForValue().set(key, product, Duration.ofMinutes(10));
    return product;
}
```

첫 요청은 DB까지 내려가지만(`MISS → DB 조회 → Redis 저장`), 두 번째 요청부터는 `HIT → 바로 반환`이라 DB에 갈 필요가 없다. 애플리케이션이 캐시 로직을 직접 관리하는 방식이라 가장 유연하고 널리 쓰인다. 다만 DB와 캐시 사이에 "잠깐 다른 값을 보게 되는" 시점이 생길 수 있다(최종적 일관성).

### 그 외 전략 — Write-Through / Write-Behind

- **Write-Through** — 쓸 때 DB와 캐시를 동시에 갱신한다. 캐시가 항상 최신 상태를 유지하지만, 매 쓰기마다 캐시 갱신 비용이 붙는다.
- **Write-Behind (Write-Back)** — 캐시에만 먼저 쓰고 DB 반영은 나중에 비동기로 몰아서 처리한다. 쓰기 성능은 가장 좋지만, DB에 반영되기 전에 Redis가 죽으면 그 데이터를 잃을 위험이 있다 — 일반적인 조회 캐시에서는 흔치 않은 방식이고, Cache-Aside가 이해하기도 쉽고 압도적으로 많이 쓰인다.

## 4. 데이터가 바뀌면 캐시는 어떻게 하나 — Stale Data

DB의 상품 가격이 100만 원 → 90만 원으로 바뀌었다고 해보자. Redis에는 여전히 100만 원이 남아있을 수 있다. 이렇게 원본과 캐시가 어긋난 상태를 **Stale Data**라고 부른다.

대응 방법은 크게 두 가지다.

```
DB UPDATE → Redis UPDATE   (캐시도 같이 갱신)
DB UPDATE → Redis DEL key  (캐시 삭제)
```

Cache-Aside를 쓰고 있다면 보통 **삭제** 쪽이 더 이해하기 쉽다. 캐시를 지우면 다음 조회에서 `MISS → DB 최신 데이터 조회 → Redis 재생성`이라는, 이미 쓰고 있던 Cache-Aside 흐름을 그대로 타기 때문이다.

## 5. TTL, 몇 분으로 잡아야 할까

TTL을 정할 때 흔히 "10분 정도면 되겠지" 하고 감으로 정하기 쉬운데, 기준은 Redis가 아니라 **데이터의 특성**이어야 한다.

```
상품 기본정보   → 1시간   (좀 늦게 반영돼도 크게 문제 없음)
카테고리        → 6시간
전시/배너       → 5~10분
재고            → 수초~수십초, 또는 아예 캐시하지 않음
사용자 세션     → 세션 만료시간
```

기준이 되는 질문은 하나다.

> **DB와 Redis의 값이 얼마나 오래 달라도 서비스가 버틸 수 있는가?**

상품 설명이 1시간 늦게 반영되는 건 괜찮을 수 있어도, 재고가 1시간 동안 "Redis: 있음 / DB: 품절" 상태로 어긋나 있으면 주문 로직이 크게 곤란해질 수 있다. 그래서 모든 캐시에 같은 TTL을 걸기보다는, 데이터 성격별로 TTL을 다르게 가져가는 게 맞다.

### TTL에 랜덤값을 섞는 이유 (TTL Jitter)

오전 10시에 상품 1만 건을 캐시에 넣으면서 TTL을 전부 정확히 300초로 줬다고 해보자. 10시 5분이 되는 순간, 1만 개의 키가 동시에 만료되면서 Cache Miss가 한꺼번에 터지고 DB 조회가 폭증한다.

```java
int jitter = ThreadLocalRandom.current().nextInt(0, 60);
redisTemplate.opsForValue().set(key, product, Duration.ofSeconds(300 + jitter));
```

이렇게 TTL에 작은 랜덤 값(위 예시라면 0~60초)을 더해주면 키마다 만료 시점이 흩어진다. 사소해 보이지만 트래픽이 큰 서비스에서는 꽤 중요한 차이를 만든다.

## 6. Cache Stampede — 인기 키 하나가 만료되는 순간

TTL Jitter로도 완전히 막지 못하는 상황이 있다. 인기 상품 하나(`product:iphone`)의 TTL이 끝나는 그 순간, 동시에 들어온 요청들이 전부 Cache Miss가 되어 한꺼번에 DB로 몰려가는 현상을 **Cache Stampede**라고 한다.

```
product:iphone 만료
      │
  ┌───┼───┐
Req1  Req2  Req3   ← 모두 동시에 MISS
  │    │    │
  DB   DB   DB      ← DB에 요청이 몰림
```

평소 Redis가 DB 트래픽 대부분을 막아주고 있었다면, 이 순간 DB가 감당 못 할 수도 있다. 대표적인 완화 방법은 다음과 같다.

- TTL Jitter (위에서 설명한 방식)
- **캐시 갱신을 락으로 한 번만 수행** — Cache Miss가 동시에 100번 발생해도, 락을 획득한 요청 하나만 DB를 조회해서 캐시를 채우고 나머지는 그 결과를 기다렸다가 캐시에서 읽는다
- 캐시가 완전히 만료되기 전에 미리 갱신
- 요청 병합, 적절한 Rate Limit

여기서 "캐시 갱신을 락으로 한 번만 수행"하는 부분은 결국 분산 락 문제이기도 하다. Redis 분산 락을 직접 구현하다 보면 놓치기 쉬운 함정들을 [AOP로 감싼 Redisson 분산 락에서 발견한 버그 두 개](/spring/2026/09/22/redisson-aop-distributed-lock-pitfalls/) 글에서 다뤘다.

## 7. Cache Penetration — 존재하지 않는 데이터를 계속 찾는 요청

`GET /products/999999999`처럼 애초에 존재하지 않는 데이터를 계속 요청받는 경우도 있다. 없는 데이터는 캐시에 저장된 적이 없으니 매번 `MISS → DB 조회 → 없음`을 반복하게 되고, DB는 이 무의미한 조회를 계속 떠안는다. 이를 **Cache Penetration**이라 한다.

해결책은 "없다"는 사실 자체도 짧게 캐시하는 것이다.

```
product:999999999 = "__NULL__"   (TTL 30초)
```

두 번째 요청부터는 `HIT → 없는 상품 처리`로 끝나기 때문에 DB까지 반복해서 내려가는 걸 막을 수 있다. 다만 NULL 캐시의 TTL은 일반 데이터보다 짧게 가져가는 게 일반적이다(나중에 진짜 데이터가 생겼을 때 너무 오래 "없음"으로 남아있으면 안 되므로).

## 8. Hot Key와 Big Key — Redis인데 왜 느리지?

### Hot Key
평소엔 상품마다 트래픽이 고르게 분산돼 있다가, 특정 상품 하나(신제품 예약 판매 등)에 요청이 집중되는 경우가 있다. 특히 Redis Cluster에서는 Key가 특정 노드(shard)에 고정 배치되기 때문에, 클러스터 전체 자원은 충분해 보여도 **그 키를 담당하는 노드 하나만 병목**이 될 수 있다. 대규모 이벤트나 인기 상품처럼 특정 키에 트래픽이 몰릴 가능성이 있다면 미리 고려해야 하는 문제다.

### Big Key
"최근 조회 상품" 같은 List에 삭제/트림 없이 계속 값만 추가하면, 그 키 하나가 수백만 개 원소를 가진 거대한 값이 될 수 있다. 이런 키를 **Big Key**라고 부른다. `HGETALL`, `SMEMBERS`, `LRANGE 0 -1`처럼 컬렉션 전체를 읽는 명령은 데이터가 커질수록 비용이 커지므로 특히 주의해야 한다.

> Redis가 빠르다는 것과, "내가 쓰는 명령어가 빠르다"는 것은 다른 이야기다 — 자료구조뿐 아니라 사용하는 명령어의 시간복잡도도 같이 봐야 한다.

앞서 나온 최근 조회 상품 예시에서 `LTRIM`으로 개수를 제한했던 이유가 바로 이 Big Key를 애초에 만들지 않기 위해서다.

## 9. 메모리가 가득 차면 — Eviction Policy

`maxmemory`에 도달했을 때 Redis가 어떤 키부터 지울지 정하는 정책이다. `maxmemory-policy` 설정으로 지정한다.

| 정책 | 설명 |
|---|---|
| `noeviction` | 새로 못 씀 (에러 반환) — 기본값 |
| `allkeys-lru` | 가장 오랫동안 사용 안 된(Least Recently Used) 키부터 제거 — 캐시 용도로 가장 흔히 씀 |
| `allkeys-lfu` | 사용 빈도(Least Frequently Used)가 가장 낮은 키부터 제거 |
| `volatile-ttl` | TTL이 설정된 키 중 만료가 가장 임박한 키부터 제거 |

"최근에 쓰인 게 중요하다"면 LRU, "자주 쓰이는 게 중요하다"면 LFU — 접근 패턴에 따라 고를 수 있다. 정답은 하나가 아니다.

## 10. Redis가 장애 나면 서비스도 같이 죽어야 할까

순수 Cache-Aside 구조라면 `HIT → Redis 사용`, `MISS → DB 사용`, `Redis 장애 → DB 사용`처럼 자연스럽게 DB로 폴백할 수 있을 것 같지만, 여기엔 함정이 하나 있다.

Redis가 평소 5,000 TPS 중 4,750 TPS(95%)를 처리하고, DB는 나머지 250 TPS 정도만 받고 있었다고 해보자. Redis가 죽는 순간 5,000 TPS가 그대로 DB로 몰린다.

```
Redis 장애 → 5,000 TPS 전부 DB로
          → DB Connection Pool 고갈
          → API Timeout
          → 서비스 장애
```

평소 250 TPS만 받던 DB가 갑자기 5,000 TPS를 받으면 못 버틸 수 있다. 즉,

> **"Redis가 죽으면 DB로 폴백한다"는 설계는 절반짜리다 — DB가 그 트래픽을 실제로 감당할 수 있는지까지 같이 설계해야 완전하다.**

그래서 Redis 타임아웃, Circuit Breaker, Rate Limiting, DB Connection Pool 크기 같은 것도 캐시 설계와 한 세트로 고민해야 한다.

## 11. Spring Boot에서는 `@Cacheable`로 더 간단하게

지금까지는 `RedisTemplate`을 직접 다루는 예시였는데, Spring Cache 추상화를 쓰면 애너테이션 하나로 같은 Cache-Aside 흐름을 표현할 수 있다.

```java
@Cacheable(cacheNames = "products", key = "#id")
public Product getProduct(Long id) {
    return productRepository.findById(id).orElseThrow();
}

@CacheEvict(cacheNames = "products", key = "#id")
public void updateProduct(Long id, ProductUpdateRequest request) {
    // ...
}
```

`getProduct`는 처음 호출되면 `MISS → DB 조회 → 캐시 저장` 흐름을 타고, 이후엔 `HIT → 바로 반환`한다. `updateProduct`가 호출되면 해당 캐시를 지워서 다음 조회 때 최신 값을 다시 채우게 만든다. 결국 `@Cacheable`/`@CacheEvict`도 지금까지 설명한 Cache-Aside와 캐시 무효화를, Spring이 대신 배선해주는 것뿐이라고 보면 된다.

## 12. Redis를 붙였는데 정말 효과가 있었나 — 측정하기

캐시를 적용했다고 성능이 좋아졌다고 **가정**하면 안 되고, 실제 지표로 확인해야 한다. 대표적으로 보는 값들:

```
Cache Hit Ratio / Cache Miss Ratio
Redis Latency, CPU, Memory Usage, Evicted Keys
DB QPS, DB Connection Pool 사용률
API 응답시간
```

**Cache Hit Ratio**가 특히 중요한 지표다.

```
Cache Hit Ratio = Cache Hit / (Cache Hit + Cache Miss)
```

적용 전 상품 API가 5,000 TPS에 DB SELECT도 5,000 QPS, 평균 응답 80ms였다면, 적용 후 Cache Hit이 4,750(Hit Ratio 95%)이 나오면서 DB SELECT는 250 QPS로, 평균 응답은 15ms로 떨어지는 식으로 개선을 수치로 확인할 수 있어야 한다. 반대로 Hit Ratio가 10% 수준이라면, 캐시를 운영하는 비용 대비 실제 효과가 있는지 다시 따져봐야 한다.

## 정리

- 자료구조는 용도에 맞게 고른다 — 단순 값은 String, 카운터는 String+`INCR`, 객체는 Hash, 랭킹/정렬은 Sorted Set.
- 순수 캐시 용도라면 영속성(RDB/AOF)을 꺼도 되지만, 세션처럼 데이터 유실이 문제가 되는 용도라면 반드시 켜야 한다.
- 캐시 전략은 Cache-Aside가 기본값이고, Write-Through/Write-Behind는 트레이드오프를 이해하고 필요할 때만 고려한다.
- TTL은 "데이터가 얼마나 오래 stale해도 괜찮은가"로 정하고, 대량의 키가 한꺼번에 만료되지 않도록 TTL Jitter를 챙긴다.
- Cache Stampede(인기 키 동시 만료), Cache Penetration(존재하지 않는 데이터 반복 조회), Hot Key/Big Key(특정 키 병목)는 트래픽이 커질수록 실제로 마주치는 문제들이다.
- Redis 장애 시 DB로 폴백하는 설계라도, DB가 그 트래픽을 실제로 받아낼 수 있는지까지 같이 설계해야 한다.
- Spring에서는 `@Cacheable`/`@CacheEvict`로 이 패턴들을 추상화할 수 있지만, 결국 그 안에서 일어나는 일은 이 글에서 다룬 것과 같다.
- 마지막으로 Cache Hit Ratio 같은 실제 지표로 캐시가 정말 도움이 되고 있는지 확인한다.

결국 Redis 캐싱에서 중요한 질문은 "Redis를 어떻게 쓸까"보다 **"이 데이터는 왜 캐시해야 하고, 캐시가 사라지면 서비스는 어떻게 동작해야 하는가"**에 더 가깝다. 잘못 만든 캐시는 성능 개선 수단이 아니라 또 하나의 장애 포인트가 될 수 있다.

## 참고자료
- [Redis 공식 문서 - Data types](https://redis.io/docs/latest/develop/data-types/)
- [Redis 공식 문서 - Persistence](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/)
- [Redis 공식 문서 - Eviction policies](https://redis.io/docs/latest/develop/reference/eviction/)
- [Spring 공식 문서 - Cache Abstraction](https://docs.spring.io/spring-framework/reference/integration/cache.html)
- 이 블로그의 [Redis-cli 원격 접속하기](/redis/2019/06/19/redis-cli-remote-connect/), [Spring Session + Redis](/spring/2018/11/30/spring-session-redis/), [AOP로 감싼 Redisson 분산 락에서 발견한 버그 두 개](/spring/2026/09/22/redisson-aop-distributed-lock-pitfalls/)

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
