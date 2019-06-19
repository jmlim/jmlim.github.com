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

출처: 
 -  https://blog.geun.kr/349 [Geun`s Page]

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
