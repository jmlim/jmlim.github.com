---
layout: post
title:  "스프링부트 + jsp 환경에서 tiles 사용하기 (desktop, mobile 접근 구분하여 페이지 분기)"
date:   2019-02-09 09:44:00 +0900
categories: Spring
comments: true
tags: [Spring, Java, tiles, 타일즈, mobile, 모바일]
---

---

스프링부트 + jsp + tiles 설정이 이미 되어 있다고 가정한다.

[스프링부트 + jsp + tiles 설정 참고 : http://jmlim.github.io/spring/2019/02/08/spring-boot-tiles/]

테스트한 버전은 2.x 버전이며 스프링 부트 버전에 따라 버전을 조금 다르게 가야할 수도 있을 것 같다.

## 1. pom.xml 에 spring mobile 추가. 

스프링 모바일 2.x 버전은 아직 정식버전이 나오지 않았으므로 마일스톤에서 내려받는다. (스프링 마일스톤 레포지토리를 추가하여 내려받음.)

```xml
....

  <!-- 스프링 마일스톤 repo -->
 <repositories>
     <repository>
         <id>spring-milestones</id>
         <name>Spring Milestones</name>
         <url>https://repo.spring.io/milestone</url>
     </repository>
 </repositories>
....

  <!-- 스프링 모바일 -->
  <dependency>
      <groupId>org.springframework.mobile</groupId>
      <artifactId>spring-mobile-starter</artifactId>
      <version>2.0.0.M2</version>
  </dependency>

```

## 2. yml or properties 설정 추가.

yml 사용 시 

```yaml
spring: 
  mobile:
    devicedelegatingviewresolver:
      enabled: true
```

properties 사용 시

```properties
spring.mobile.devicedelegatingviewresolver.enabled: true
```

이렇게만 추가해도 컨트롤러의 요청 메소드에 Device 정보를 argument 로 받을 수 있다.
```java
@RequestMapping("/")
public void home(Device device) {
    if (device.isMobile()) {
        logger.info("Hello mobile user!");
    } else if (device.isTablet()) {
        logger.info("Hello tablet user!");
    } else {
        logger.info("Hello desktop user!");         
    }
}


```

## 3. TilesConfig 에 모바일 분기 관련 설정 추가.

1. tiles Configurer 에 tiles-mobile.xml 설정 추가.
2. LiteDeviceDelegatingViewResolver 설정 추가.

```java

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.mobile.device.view.LiteDeviceDelegatingViewResolver;
import org.springframework.web.servlet.view.tiles3.TilesConfigurer;
import org.springframework.web.servlet.view.tiles3.TilesView;
import org.springframework.web.servlet.view.tiles3.TilesViewResolver;

@Configuration
public class TilesConfig {

    @Bean
    public TilesConfigurer tilesConfigurer() {
        final TilesConfigurer configurer = new TilesConfigurer();
        configurer.setDefinitions(new String[]{"/WEB-INF/tiles/tiles.xml",
             // 모바일 분기 관련 설정 추가부분..
                "WEB-INF/tiles/tiles-mobile.xml"});
        configurer.setCheckRefresh(true);
        return configurer;
    }

    @Bean
    public TilesViewResolver tilesViewResolver() {
        final TilesViewResolver tilesViewResolver = new TilesViewResolver();
        tilesViewResolver.setViewClass(TilesView.class);
        return tilesViewResolver;
    }

    /** 
    * 모바일 분기 관련 설정 추가부분..
    */
    @Bean
    public LiteDeviceDelegatingViewResolver liteDeviceDelegatingViewResolver() {
        LiteDeviceDelegatingViewResolver resolver = new LiteDeviceDelegatingViewResolver(this.tilesViewResolver());
        
        /** 
          tilesViewResolver의 Order 값을 최상위로 높여 주지 않으면  앞서 프로퍼티에서 생성한
          LiteDeviceDelegatingViewResolver에의해 ContentNegotiatingViewResolver 에서 org.springframework.web.servlet.view.JstlView 이 실행되어 Tiles가 동작하지 않게 된다.
        */
        resolver.setOrder(0);
        resolver.setMobilePrefix("mobile/");
        resolver.setEnableFallback(true);
        return resolver;
    }
}

```
## 4. tiles-mobile.xml 설정 추가. (기존 tiles.xml 에서 mobile 키워드 추가.)

> ex1) name = main-layout -> mobile-main-layout <br/>
> ex2) value = /WEB-INF/views/main/body/{1}.jsp -> /WEB-INF/views/mobile/main/body/{1}.jsp

```xml 

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE tiles-definitions PUBLIC
       "-//Apache Software Foundation//DTD Tiles Configuration 3.0//EN"
       "http://tiles.apache.org/dtds/tiles-config_3_0.dtd">
<tiles-definitions>

   <!-- main -->
   <definition name="mobile-main-layout" template="/WEB-INF/views/mobile/main/layout/base-main.jsp">
       <put-attribute name="header" value="/WEB-INF/views/mobile/common/layout/header.jsp" />
       <put-attribute name="body" value="" />
       <put-attribute name="footer" value="/WEB-INF/views/mobile/common/layout/footer.jsp" />
   </definition>
   <definition name="mobile/main/*" extends="mobile-main-layout">
       <put-attribute name="body" value="/WEB-INF/views/mobile/main/body/{1}.jsp" />
   </definition>
   <definition name="mobile/main/*/*" extends="mobile-main-layout">
       <put-attribute name="body" value="/WEB-INF/views/mobile/main/body/{1}/{2}.jsp" />
   </definition>
   <definition name="mobile/main/*/*/*" extends="mobile-main-layout">
       <put-attribute name="body" value="/WEB-INF/views/mobile/main/body/{1}/{2}/{3}.jsp" />
   </definition>

</tiles-definitions>

```

 경로 참고 이미지
 <img src="{{ site.baseurl }}/public/post/springboot/spring-boot-tiles-mobile-layout.png"/>
 <br/>

## 5. Controller 에서 페이지 호출 
위 설정대로 모바일로 접근 시 경로 리턴값에 자동으로 mobile/ 이 붙게 된다. 

아래 LiteDeviceDelegatingViewResolver 소스 부분 참고

> ex) mv.setViewName("main/index"); <br/>
> 로 리턴시 mv.setViewName("mobile/main/index"); 로 설정값이 바뀜

```java


@Controller
@RequestMapping("/")
public class MainController  {

    @GetMapping(value = {"", "index"})
    public ModelAndView index() {

        ModelAndView mv = new ModelAndView();
        
        .....
        로직로직
        ....
        
        mv.setViewName("main/index");
        return mv;
    }
}
```

### LiteDeviceDelegatingViewResolver.java
```java
public class LiteDeviceDelegatingViewResolver extends AbstractDeviceDelegatingViewResolver {
    private String normalPrefix = "";
    private String mobilePrefix = "";
    private String tabletPrefix = "";
    private String normalSuffix = "";
    private String mobileSuffix = "";
    private String tabletSuffix = "";
 .....
  
   // 1. 앞에 키워드를 붙이는 부분이며 아까 설정에서 mobilePrefix 를 mobile/ 로 설정하였다. 
    protected String getDeviceViewNameInternal(String viewName) {
        RequestAttributes attrs = RequestContextHolder.getRequestAttributes();
        Assert.isInstanceOf(ServletRequestAttributes.class, attrs);
        HttpServletRequest request = ((ServletRequestAttributes)attrs).getRequest();
        Device device = DeviceUtils.getCurrentDevice(request);
        SitePreference sitePreference = SitePreferenceUtils.getCurrentSitePreference(request);
        String resolvedViewName = viewName;
        if (ResolverUtils.isNormal(device, sitePreference)) {
            resolvedViewName = this.getNormalPrefix() + viewName + this.getNormalSuffix();
        } 
        // 2. 모바일 디바이스일 경우 이부분이 조건으로 타게되며 그 경우에 모바일 prefix 인 mobile/ 이 붙에 됨. 
        else if (ResolverUtils.isMobile(device, sitePreference)) {
            resolvedViewName = this.getMobilePrefix() + viewName + this.getMobileSuffix();
        } else if (ResolverUtils.isTablet(device, sitePreference)) {
            resolvedViewName = this.getTabletPrefix() + viewName + this.getTabletSuffix();
        }

        return this.stripTrailingSlash(resolvedViewName);
    }

    private String stripTrailingSlash(String viewName) {
        return viewName.endsWith("//") ? viewName.substring(0, viewName.length() - 1) : viewName;
    }
}

```
참고: 
 - https://mycup.tistory.com/208
 - http://projects.spring.io/spring-mobile/

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
