---
layout: post
title:  "[MyBatis] insert 후 key값 가져오기"
date:   2019-06-07 17:00:00 +0900
categories: Mybatis
comments: true
tags: [MyBatis, 마이바티스]
---

---

> mybatis 의 insert 구문을 통해 autoincrement 된 키값을 가져오고 싶을 때 사용하는 방법이다.

## useGeneratedKeys 옵션

```xml
 <insert id="insertData" parameterType="DataClass" useGeneratedKeys="true" keyProperty="id">
     /* query */

 </insert>
```

id가 autoincrement인 PK일 경우, 그 id값은 DataClass에 선언되어 있는 id 필드 안으로 값이 저절로 들어간다.<br/>
debug 찍고 DataClass 객체의 id값을 보면 값이 들어있을 것이다.


## selectKey 옵션

Oracle 같은 경우는 Auto Increment 가 없고 Sequence를 사용해야만 한다. 해당 경우엔 아래와 같이 설정해주면 된다.

```xml
<insert id="insertStudents" parameterType="DataClass">
  <selectKey keyProperty="id" resultType="int" order="BEFORE">
    select SEQ_ID.nextval FROM DUAL
  </selectKey>
     /* query */
</insert>
```


### 참고자료
 - https://marobiana.tistory.com/23 [Take Action]
 - https://taetaetae.github.io/2017/04/04/mybatis-useGeneratedKeys/

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

---
