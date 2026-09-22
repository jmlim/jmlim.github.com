---
layout: post
title:  "Redis-cli 원격 접속하기."
date:   2019-06-19 21:39:00 +0900
categories: Redis
comments: true
tags: [Redis, Redis-cli]
---

---

리눅스 데스크탑 매니저 파일이 없어도 외부에서 redis-cli 명령어로 접근이 가능하다는 것을 이제야 알았다...왜 이제 알았을까

## 1. redis 설치 (windows)
 - https://github.com/rgl/redis/downloads 에서 설치파일 내려받고 설치하면 Program Files 에 설치됨.
 - **[2026년 추가]** 이 링크는 이제 접근이 안 될 가능성이 높다 — GitHub의 "Downloads" 기능 자체가 오래전에 없어졌다. 애초에 Redis는 공식적으로 Windows를 지원하지 않으므로, 요즘은 WSL2에 리눅스용 Redis를 설치하거나 `docker run -d -p 6379:6379 redis` 로 띄우는 걸 권장한다.

## 2. 리눅스에 Redis 설치 (ubuntu)
  먼저 리눅스에 redis를 설치하기 위해서 파일을 직접 다운받거나 apt-get 등을 이용할 수 있다. 
  만약 apt-get 패키지 다운로드를 사용할 경우 아래와 같이 커맨드를 입력하면 된다.
  ```
  $ sudo apt-get install redis-server
  ```
 
## 3. 명령어
```
  redis-cli -h 123.123.123.123(레디스 설치되어있는 아이피) -p 6379 -a password
```

 -h : 호스트 아이피

 -p : redis 포트

 -a : 비밀번호

> **[2026년 추가]** `-a` 로 비밀번호를 커맨드에 그대로 넣으면 쉘 히스토리(`~/.bash_history` 등)와 그 순간의 프로세스 목록(`ps aux`로 다른 사용자도 볼 수 있음)에 비밀번호가 그대로 남는다. `redis-cli`도 이 점을 인지해서 `-a` 사용 시 경고 메시지를 띄워준다. `-a` 없이 `redis-cli -h ... -p ...` 로 접속하면 대화형으로 비밀번호를 물어보거나, 스크립트에서는 `REDISCLI_AUTH` 환경변수로 넘기는 게 더 안전하다.
>
> 더 근본적으로, Redis 인스턴스를 인증 없이(또는 기본 설정 그대로) 외부에 노출해두면 실제로 랜섬웨어 공격의 표적이 된 사례가 많았다 — "원격 접속이 된다"는 것 자체보다, 그 포트가 인터넷에 얼마나 열려 있는지(방화벽/보안그룹으로 필요한 IP만 허용했는지)가 훨씬 중요한 점검 포인트다.

출처: 
 -  https://blog.geun.kr/349 [Geun`s Page]

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
