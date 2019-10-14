---
layout: post
title:  "[ERROR] Invalid character found in the request target. The valid characters are defined in RFC 7230 and RFC 3986"
date:   2019-10-14 20:00:00 +0900
categories: Java
comments: true
tags: [Java, Exception, ERROR, Invalid character, RFC 7230, RFC 3986]
---

---

배경: 
- 스프링 부트를 사용하고 있으며, 최신 내장형 탐캣은 7에서 8.5로 되었다고 가정한다.

## 요청 예 
```html
<BODY onload=f.submit()>
<FORM name=f method=post action='http://jmlim.aaa.com/authentication?jmlim[login]=1&jmlim[app]=72&jmlim[from]=testjmlim'>
</FORM>
</BODY>
```

## 요청을 받는 객체 및 컨트롤러 예
~~~java
@Data
public class JmlimRequest {
    //private String loginSSO;
    //private String from;
    //private String app;
    private Map<String, Object> jmlim;
    //private SsoReloadRequest reload;
}
~~~

~~~java
@Controller
public class ApiController {
  /**
   */
  @RequestMapping("/authentication")
  public String index(@ModelAttribute JmlimRequest jmlimRequest) throws Exception {

   ....
   ....

  }
}
~~~

뭐.. 대충 위와 같은 형태로 post 로 요청을 하고 받는다면 아래와 같은 Exception 이 발생한다.
 
~~~java
Note: further occurrences of HTTP header parsing errors will be logged at DEBUG level.
 
java.lang.IllegalArgumentException: Invalid character found in the request target. The valid characters are defined in RFC 7230 and RFC 3986
    at org.apache.coyote.http11.Http11InputBuffer.parseRequestLine(Http11InputBuffer.java:479) ~[tomcat-embed-core-8.5.34.jar:8.5.34]
    at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:684) ~[tomcat-embed-core-8.5.34.jar:8.5.34]
    at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:66) [tomcat-embed-core-8.5.34.jar:8.5.34]
    at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:806) [tomcat-embed-core-8.5.34.jar:8.5.34]
    at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1498) [tomcat-embed-core-8.5.34.jar:8.5.34]
    at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49) [tomcat-embed-core-8.5.34.jar:8.5.34]
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [na:1.8.0_191]
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [na:1.8.0_191]
    at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61) [tomcat-embed-core-8.5.34.jar:8.5.34]
    at java.lang.Thread.run(Thread.java:748) [na:1.8.0_191]
~~~

사실 위 요청을 애초에 javascript 의 encodeURI 함수를 통해 [ <--문자를 인코딩해서 보낼 수 있다면 위 문제는 발생하지 않는다. 

만약 위 요청을 우리가 컨트롤 할 수 있는 상황이 아니라고 한번 더 가정해본다.


우선.. Tomcat 의 특정 버전 이상에서 RFC 3986 규격이 적용되었다고 한다.
<br/>
RFC 3986에는 영어 문자(a-zA-Z), 숫자(0-9), -. ~4 특수 문자 및 모든 예약 문자만 허용된다.

위 에러는 특수문자 중 ([) 를 차단하면서 발생한 것인데,  
수정하려면 아래 relaxQueryChars 옵션에 허용할 문자를 추가하거나 톰캣 버전을 다운그레이드 해야한다.

아래는 스프링 부트에서 해당 옵션을 추가하는 예 이다.

~~~java
@Configuration
public class TomcatWebServerCustomizer
        implements WebServerFactoryCustomizer<TomcatServletWebServerFactory> {

    /**
     * 톰캣에 옵션 추가.
     *
     * @param factory
     */
    @Override
    public void customize(TomcatServletWebServerFactory factory) {
        factory.addConnectorCustomizers((TomcatConnectorCustomizer)
                connector -> connector.setAttribute("relaxedQueryChars", "<>[\\]^`{|}"));
    }
}
~~~


## 참고자료
- https://stackoverflow.com/questions/51703746/setting-relaxedquerychars-for-embedded-tomcat/51703955#51703955
- https://stackoverflow.com/questions/41053653/tomcat-8-is-not-able-to-handle-get-request-with-in-query-parameters
- https://programmer.help/blogs/characters-are-defined-in-rfc-7230-and-rfc-3986.html


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

---

