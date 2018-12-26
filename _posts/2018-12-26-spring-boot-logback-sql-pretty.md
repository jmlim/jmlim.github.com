---
layout: post
title:  "스프링 부트(Spring boot) 마이바티스(Mybatis) 에서 쿼리 로그 출력 및 정렬하기."
date:   2018-12-26 18:40:00 +0900
categories: Spring
comments: true
tags: [Spring, Logback, SQL 정렬, SQL pretty, MyBatis, 마이바티스]
---

---

> (이 내용은 Spring boot의 logback(기본로그설정)을 사용하며 mybatis를 사용한다고 가정한다.)

Spring boot 와 Mybatis를 이용하여 초기 셋팅 후 개발을 하다 보면 쿼리가 나오지 않거나 혹은 쿼리가 나오게 mapper 에 해당하는 패키지의 log를 debug 레벨로 두더라도 쿼리가 한줄로 나오거나 파라미터가 ? 로 따로 찍히는 현상이 발생한다.
Spring data jpa 를 사용하면 아래 간단한 옵션으로 정렬된 SQL 구문을 볼 수 있으나 Mybatis 사용시엔 그렇지 않다.

### spring boot data jpa 사용 시 쿼리 정렬 옵션

```properties
spring.jpa.properties.hibernate.format_sql=true
```

정렬 처리가 되어있지 않을 경우 개발하면서 불편한게 한두가지가 아니다.
아래 내용을 적용하면 쿼리를 보기좋게 하여 좀 더 생산성을 높일 수 있다.



## 1. pom.xml 에 다음 라이브러리 추가.
```xml

<!-- mybatis sql pretty -->
<dependency>
    <groupId>org.bgee.log4jdbc-log4j2</groupId>
    <artifactId>log4jdbc-log4j2-jdbc4.1</artifactId>
    <version>1.16</version>
</dependency>

```

## 2. application.yml에서 기존 사용하던 driver 및 url 정보 변경. (주석 참고)
```yaml
spring:
  datasource:
    #url: jdbc:mysql://localhost:3306/sample  ## 1. 기존 사용하던 jdbc:mysql 로 시작하는 url 주석처리 후 jdbc:log4jdbc:mysql 로 시작하는 url 로 변경.
    url: jdbc:log4jdbc:mysql://localhost:3306/sample
    username: jmlim
    password: 123456
    #driver-class-name: com.mysql.jdbc.Driver ## 2. 기존 사용하던 com.mysql.jdbc.Driver를 net.sf.log4jdbc.sql.jdbcapi.DriverSpy 로 변경.
    driver-class-name: net.sf.log4jdbc.sql.jdbcapi.DriverSpy
    hikari:
      connection-timeout: 5000
      validation-timeout: 1000
      maximum-pool-size: 30
      minimum-idle: 2
      connection-test-query: SELECT 1
```

## 3. application.yml 에서 jdbc.sqlonly 패키지에 대한 로깅 레벨 추가.  (주석 참고)
```yaml

logging:
  level:
    jdbc.sqlonly: DEBUG ## 이부분 추가.
    org.springframework.web: DEBUG
    com.zaxxer.hikari.HikariDataSource: ERROR
```

#### 참고로 log4jdbc는 아래와 같은 옵션들을 제공한다.
```
 1. jdbc.sqlonly : SQL문 만을 로그로 남기며, PreparedStatement일 경우 관련된 argument 값으로 대체된 SQL문이 보여진다.
 2. jdbc.sqltiming : SQL문과 해당 SQL을 실행시키는데 수행된 시간 정보(milliseconds)를 포함한다. 
 3. jdbc.audit : ResultSet을 제외한 모든 JDBC 호출 정보를 로그로 남긴다. 많은 양의 로그가 생성되므로 특별히 JDBC 문제를 추적해야 할 필요가 있는 경우를 제외하고는 사용을 권장하지 않는다. 
 4. jdbc.resultset : ResultSet을 포함한 모든 JDBC 호출 정보를 로그로 남기므로 매우 방대한 양의 로그가 생성된다. 
 5. jdbc.resultsettable : SQL 결과 조회된 데이터의 table을 로그로 남긴다.
```
각옵션들의 기능을 정확히 알기 위해서는 직접 하나씩 변경해 보며 구동하는 것이 좋을 것이다.

## 4. resources 경로 밑에 log4jdbc.log4j2.properties 파일 추가.

```properties

log4jdbc.spylogdelegator.name=net.sf.log4jdbc.log.slf4j.Slf4jSpyLogDelegator
log4jdbc.dump.sql.maxlinelength=0

```

## 5. 출력결과

```sql

2018-12-26 18:31:57.381 [pool-2-thread-1] DEBUG jdbc.sqlonly [223] - com.zaxxer.hikari.pool.ProxyPreparedStatement.execute(ProxyPreparedStatement.java:44)
1. /* [TEST] io.jmlim.sample.mapper.SampleMapper#selectSampleList */
  SELECT CODE
       , NAME
       , START_DATE
       , END_DATE
       , GRADE_TYPE
       , GRADE_VALUE
    FROM JM_SAMPLE
   WHERE USE_FLAG = 'Y'
     AND CODE = 'JM00001'
```

### 참고자료
 - https://www.leafcats.com/45
 - https://modunote.tistory.com/143

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

---
