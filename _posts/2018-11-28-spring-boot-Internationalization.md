---
layout: post
title:  "스프링부트에서 다국어 기능 사용하기."
date:   2018-11-28 09:12:00 +0900
categories: Spring
comments: true
tags: [Spring, Java]
---

---
### 스프링부트에서 다국어 기능 사용하기.

기존 Spring + java web application 다국어 지원 설정을 하기 위해선 여러가지 사항들을 고려 해야 한다.
spring framework 및 설정을 통해서 다국어를 사용하는 방법에 여러가지가 있다.

하지만 Spring Boot의 가장 큰 장점은 대부분의 설정이 이미 되어있어서 별로로 설정하지 않아도 사용 가능하다. 
MessageSource 설정도 역시 이미 되어있다.

추기로 이 블로그에선 yml 파일을 사용하여 다국어를 설정해 보는 방법을 작성하였다.

다국어 관련 Config 파일 작성.
--

yml YamlResourceBundle 클래스를 사용하기 위한 pom.xml 파일 디펜던시 추가.
```xml
<!-- https://mvnrepository.com/artifact/net.rakugakibox.util/yaml-resource-bundle -->
<dependency>
  <groupId>net.rakugakibox.util</groupId>
  <artifactId>yaml-resource-bundle</artifactId>
  <version>1.1</version>
</dependency>
```

Configuration 작성.
```java
package 파일이 들어갈 패키지명;

import net.rakugakibox.util.YamlResourceBundle;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.MessageSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.support.ResourceBundleMessageSource;
import org.springframework.web.servlet.LocaleResolver;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.servlet.i18n.LocaleChangeInterceptor;
import org.springframework.web.servlet.i18n.SessionLocaleResolver;
import java.util.Locale;
import java.util.MissingResourceException;
import java.util.ResourceBundle;

@Configuration
public class MessageSourceConfig extends WebMvcConfigurerAdapter {

     @Bean
    public LocaleResolver localeResolver() {
        SessionLocaleResolver slr = new SessionLocaleResolver();
        slr.setDefaultLocale(Locale.KOREA);
        return slr;
    }

    @Bean
    public LocaleChangeInterceptor localeChangeInterceptor() {
        LocaleChangeInterceptor lci = new LocaleChangeInterceptor();
        lci.setParamName("lang");
        return lci;
    }
    
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(localeChangeInterceptor());
    }
    
    @Bean("messageSource")
    public MessageSource messageSource(
            @Value("${spring.messages.basename}") String basename,
            @Value("${spring.messages.encoding}") String encoding
    ) {
        YamlMessageSource ms = new YamlMessageSource();
        ms.setBasename(basename);
        ms.setDefaultEncoding(encoding);
        ms.setAlwaysUseMessageFormat(true);
        ms.setUseCodeAsDefaultMessage(true);
        ms.setFallbackToSystemLocale(true);
        return ms;
    }
}

class YamlMessageSource extends ResourceBundleMessageSource {
    @Override
    protected ResourceBundle doGetBundle(String basename, Locale locale) throws MissingResourceException {
        return ResourceBundle.getBundle(basename, locale, YamlResourceBundle.Control.INSTANCE);
    }
}
```


src/main/resources/appilcation.yml 에 다국어 설정파일이 있는 경로 추가.

```yaml
spring: 
 .....
  messages:
    basename: i18n/messages
    encoding: UTF-8
```

src/main/resources/i18n/messages.yml 파일

```yaml
alert:
 confirm: "확인"
 cancel: "취소"
 loginEmail: "아이디(이메일)를 입력해 주세요."
 loginPassword: "비밀번호를 입력해 주세요."
 loginFail: "아아디(이메일) 또는 비밀번호가 올바르지 않습니다." 
```

src/main/resources/i18n/messages_en.yml 파일

```yaml
alert:
 confirm: "Ok"
 cancel: "Cancel"
 loginEmail: "Please enter your ID(Email)."
 loginPassword: "Please enter a password."
 loginFail: "The email or password is invalid." 
```

src/main/resources/i18n/messages_jp.yml 파일
```yaml
alert:
 confirm: "確認"
 cancel: "取り消し"
 loginEmail: "ID(Eメール）を入力してください"
 loginPassword: "パスワードを入力してください"
 loginFail: "ID(Eメール）またはパスワードをもう一度ご確認ください。" 
```

src/main/resources/i18n/messages_zh.yml 파일.
```yaml
alert:
 confirm: "确认"
 cancel: "取消"
 loginEmail: "请输入用户名(邮箱)。"
 loginPassword: "请输入密码。"
 loginFail: "请重新确认用户名(邮箱)或者密码。" 
```

jsp 페이지에서 사용하기
--
```jsp
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags"%> <-- 이와같이 태그 선언 후 

<spring:message code="alert.loginEmail" /> 원하는 텍스트가 쓰여진 메세지 코드값 정의

```

thymeleaf 페이지에서 사용하기.
```html

  ...
  <div>
    <span th:text="#{alert.loginEmail}">아이디(이메일)를 입력해 주세요.</span>
  </div>
  ...

```

default url 조회
<img src="{{ site.baseurl }}/public/post/internationalization/default.png"/>

영문조회 시 (lang=en)
<img src="{{ site.baseurl }}/public/post/internationalization/english.png"/>

일문 조회시 (lang=jp)
<img src="{{ site.baseurl }}/public/post/internationalization/japan.png"/>



[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
