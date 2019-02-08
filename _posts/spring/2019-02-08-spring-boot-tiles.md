---
layout: post
title:  "스프링부트 + jsp 환경에서 tiles 사용하기"
date:   2019-02-08 20:44:00 +0900
categories: Spring
comments: true
tags: [Spring, Java]
---

---


뭔가 신규 프로젝트시 최근의 트렌드와는 거리가 좀 있는 구성이지만? <br/>
백엔드 웹 개발자가 화면 개발하기엔 최적의 구성이라고 생각하기도 하고(백오피스 형태의 admin 페이지 같은?) 개인적으로도 자주 쓰기에 포스팅을 해본다.<br/>
스프링부트 + jsp 가 설정이 이미 되어 있다고 가정한다.

[스프링부트 + jsp 설정 참고 : http://jmlim.github.io/spring/2018/12/13/spring-boot-jsp-setting/]

## 1. pom.xml 에 아래 의존성 추가.

```xml
....

  <!--- JSTL 필요 : JSTL Dependency를 Maven에 추가해줘야 함.-->
  <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>jstl</artifactId>
  </dependency>
  
 <dependency>
      <groupId>org.apache.tiles</groupId>
      <artifactId>tiles-jsp</artifactId>
      <version>3.0.7</version>
  </dependency>
....

```

## 2. TilesConfig 클래스를 추가하고 아래 설정을 추가한다.

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.view.tiles3.TilesConfigurer;
import org.springframework.web.servlet.view.tiles3.TilesView;
import org.springframework.web.servlet.view.tiles3.TilesViewResolver;

@Configuration
public class TilesConfig {

    @Bean
    public TilesConfigurer tilesConfigurer() {
        final TilesConfigurer configurer = new TilesConfigurer();
        //해당 경로에 tiles.xml 파일을 넣음
        configurer.setDefinitions(new String[]{"/WEB-INF/tiles/tiles.xml"});
        configurer.setCheckRefresh(true);
        return configurer;
    }

    @Bean
    public TilesViewResolver tilesViewResolver() {
        final TilesViewResolver tilesViewResolver = new TilesViewResolver();
        tilesViewResolver.setViewClass(TilesView.class);
        return tilesViewResolver;
    }
}
```
## 3. tiles.xml 설정 추가.


```xml 

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE tiles-definitions PUBLIC
       "-//Apache Software Foundation//DTD Tiles Configuration 3.0//EN"
       "http://tiles.apache.org/dtds/tiles-config_3_0.dtd">
<tiles-definitions>

   <!-- main -->
   <definition name="main-layout" template="/WEB-INF/views/main/layout/base-main.jsp">
       <put-attribute name="header" value="/WEB-INF/views/common/layout/header.jsp" />
       <put-attribute name="body" value="" />
       <put-attribute name="footer" value="/WEB-INF/views/common/layout/footer.jsp" />
   </definition>
   <definition name="main/*" extends="main-layout">
       <put-attribute name="body" value="/WEB-INF/views/main/body/{1}.jsp" />
   </definition>
   <definition name="main/*/*" extends="main-layout">
       <put-attribute name="body" value="/WEB-INF/views/main/body/{1}/{2}.jsp" />
   </definition>
   <definition name="main/*/*/*" extends="main-layout">
       <put-attribute name="body" value="/WEB-INF/views/main/body/{1}/{2}/{3}.jsp" />
   </definition>

</tiles-definitions>

```

 경로 참고 이미지
 <img src="{{ site.baseurl }}/public/post/springboot/spring-boot-tiles-layout.png"/>
 <br/>

## 4. main-layout 에 해당하는 base-main.jsp

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib uri="http://tiles.apache.org/tags-tiles" prefix="tiles" %>
<!DOCTYPE html>
<html lang="ko">
  <head>
    <meta charset="UTF-8">
    <title>ㅎㅎㅎㅎ</title>
    <!--<meta name="viewport" content="width=device-width, initial-scale=1.0">-->
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    
  </head>
   <body>
    <section class="content">
      <tiles:insertAttribute name="header"/> <!--  /WEB-INF/views/common/layout/header.jsp -->
      <tiles:insertAttribute name="body"/> <!-- body -->

      <tiles:insertAttribute name="footer"/> <!-- /WEB-INF/views/common/layout/footer.jsp -->
    </section>
  </body>
</html>


```

## 5. Controller 에서 페이지 호출 
tiles.xml 설정에서 main/* 를 호출하는 것과 같고 body 안의 index.jsp 페이지를 호출함

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



[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
