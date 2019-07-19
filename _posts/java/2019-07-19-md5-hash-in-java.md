---
layout: post
title:  "Java MD5 hexdigest 샘플 코드."
date:   2019-07-19 18:10:00 +0900
categories: Java
comments: true
tags: [Java, MD5, 암호화]
---

---

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
           System.out.println( new MD5( message ).hexdigest() );
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
