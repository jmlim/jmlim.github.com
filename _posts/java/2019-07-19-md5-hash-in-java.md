---
layout: post
title:  "Java MD5 hexdigest 샘플 코드."
date:   2019-07-19 18:10:00 +0900
categories: Java
comments: true
tags: [Java, MD5, 암호화]
---

---

> **[2026년 추가]** MD5는 체크섬(파일 중복 검사, 캐시 키 생성 등) 용도로는 지금도 쓸 수 있지만, **비밀번호 해싱이나 보안이 중요한 무결성 검증에는 쓰면 안 된다.** 이미 오래전에 충돌(collision) 공격이 실용화되어, 서로 다른 두 입력이 같은 MD5 해시값을 갖도록 의도적으로 만들어낼 수 있음이 증명되었다. 비밀번호를 저장해야 한다면 bcrypt, scrypt, Argon2처럼 느리게(계산 비용을 일부러 높여서) 설계된 해시 알고리즘을 써야 한다.
> 참고로 원본 코드의 `main()` 메서드에는 클래스명 오타(`MD5` → `Md5Utils`)가 있어서 아래에서 고쳐뒀다.

#### Md5Utils.java
```java

import java.security.MessageDigest;
import java.math.BigInteger;

class Md5Utils {
   private String message;
   public Md5Utils( String message ) {
       this.message = message;
   }
   public String hexdigest() throws Exception {
       String hd;
       MessageDigest md5 = MessageDigest.getInstance( "MD5" );
       md5.update( this.message.getBytes() );
       BigInteger hash = new BigInteger( 1, md5.digest() );
       hd = hash.toString(16); // BigInteger strips leading 0's
       while ( hd.length() < 32 ) { hd = "0" + hd; } // pad with leading 0's
       return hd;
   }
   public static void main( String[] args ) throws Exception {
       for ( String message: args ) {
           System.out.println( new Md5Utils( message ).hexdigest() ); // 오타 수정: MD5 -> Md5Utils
       }
   }
}
```

출처: 
 - https://pad.yohdah.com/101/python-vs-java-md5-hexdigest

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
