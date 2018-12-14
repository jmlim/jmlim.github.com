---
layout: post
title:  "github jekyll blog를 구글에서 검색되도록 설정하기. (naver, daum 포함)"
date:   2018-12-14 10:56:00 +0900
categories: ETC
comments: true
tags: [blog, jekyll, 지킬, 구글, 구글검색]
---

---

네이버나 티스토리 같은 블로그 서비스들은 자동으로 검색이 노출되어 구글에서 바로 검색이 가능하도록 되어있다.<br>
하지만 github와 같은 플렛폼을 사용하여 블로그를 만들거나 직접 사이트를 구성해서 만들경우엔 구글에서 검색이 되지 않는다고 한다.<br>
아래는 내가 만든 블로그가 검색이 되도록 했던 방법이다. <br>
참고로 이 블로그는 jekyll을 사용하여 제작하였다.<br>

### 절차
1. sitemap.xml 파일 작성.
2. RSS feed 생성.
3. robots.txt 만들기.
4. 검색엔진에 등록하기.

차례대로 등록해보자.

## 1. sitemap.xml 파일 작성
블로그의 /root 경로에 /sitemap.xml 파일을 만들고 아래의 내용을 복사해 넣는다. 반드시 root 디렉토리에 넣어야 한다.

#### sitemap.xml
```xml

{%raw%}
---
layout: null
---
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
    {% for post in site.posts %}
    <url>
        <loc>{{ site.url }}{{ post.url | remove: 'index.html' }}</loc>
    </url>
    {% endfor %}

    {% for page in site.pages %}
    {% if page.layout != nil %}
    {% if page.layout != 'feed' %}
    <url>
        <loc>{{ site.url }}{{ page.url | remove: 'index.html' }}</loc>
    </url>
    {% endif %}
    {% endif %}
    {% endfor %}
</urlset>
{%endraw%}

```

제대로 작성 했을 시 주소로 접근하면 아래와 같이 나온다.

 <img src="{{ site.baseurl }}/public/post/jekyll-search-add/jekyll-sitemap.png"/>
 

## 2. RSS feed 생성

네이버와 다음에 검색이 노출되게 하기 위해 rss feed 파일을 생성한다. <br>
sitemap.xml과 마찬가지로 root 디렉토리에 /feed.xml파일을 생성하고 아래의 코드를 복사한다.

#### feed.xml


```xml
{%raw%}
---
layout: null
---
<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0" xmlns:atom="http://www.w3.org/2005/Atom">
  <channel>
    <title>{{ site.title | xml_escape }}</title>
    <description>{{ site.description | xml_escape }}</description>
    <link>{{ site.url }}{{ site.baseurl }}/</link>
    <atom:link href="{{ "/feed.xml" | prepend: site.baseurl | prepend: site.url }}" rel="self" type="application/rss+xml"/>
    <pubDate>{{ site.time | date_to_rfc822 }}</pubDate>
    <lastBuildDate>{{ site.time | date_to_rfc822 }}</lastBuildDate>
    <generator>Jekyll v{{ jekyll.version }}</generator>
    {% for post in site.posts limit:30 %}
      <item>
        <title>{{ post.title | xml_escape }}</title>
        <description>{{ post.content | xml_escape }}</description>
        <pubDate>{{ post.date | date_to_rfc822 }}</pubDate>
        <link>{{ post.url | prepend: site.baseurl | prepend: site.url }}</link>
        <guid isPermaLink="true">{{ post.url | prepend: site.baseurl | prepend: site.url }}</guid>
        {% for tag in post.tags %}
        <category>{{ tag | xml_escape }}</category>
        {% endfor %}
        {% for cat in post.categories %}
        <category>{{ cat | xml_escape }}</category>
        {% endfor %}
      </item>
    {% endfor %}
  </channel>
</rss>
{%endraw%}

```

제대로 작성 했을 시 주소로 접근하면 아래와 같이 나온다.

 <img src="{{ site.baseurl }}/public/post/jekyll-search-add/jekyll-feed.png"/>

## 3. robots.txt 생성
 - 필자는 모든 검색엔진을 허용하도록 추가하였다.
 - 해당내용 나무위키 : https://namu.wiki/w/robots.txt

```
User-agent: *
Allow: /

Sitemap: https://jmlim.github.io/sitemap.xml
```


## 4. 구글에 검색되도록 하기 
1. 구글 웹 마스터 도구 접속 [https://www.google.com/webmasters/#?modal_active=none] 
2. 구글 서치 콘솔 클릭 후 블로그 사이트 주소 추가. (ex: https://jmlim.github.io/)
3. 추가 후 소유권을 확인하기 위해 html 확인(googlexxxxxx.html) 파일을 받아 root 경로에 추가.
4. 소유권이 확인되면 Google Search Console 화면으로 전환되어 나오는데 색인에 Sitemaps 에 사이트맵의 경로를 추가해준다.
5. 추가한 후 색인이 접수중임을 확인할 수 있다.
 
## 5. 네이버에 검색되도록 하기.
1. 네이버 웹마스터 도구 접속 [https://webmastertool.naver.com/]
2. 네이버에도 블로그 사이트 주소 추가. (ex: https://jmlim.github.io/)
3. 사이트 소유 확인을 위해 html 확인 파일 다운로드 후 root 경로에 추가.
4. html 확인 파일이 블로그에 업로드 되어있는지 확인 후 확인버튼 클릭.
5. 위 블로그의 웹마스터 도구에 접속한다.
6. 요청 -> RSS 제출 메뉴를 클릭 한 후 URL을 포함한 주소인 blogurl/feed.xml을 입력 후 확인한다. ( ex: https://jmlim.github.io/feed.xml)
7. 요청 -> 사이트맵제출로 들어가서 사이트맵을 등록한다. (ex : https://jmlim.github.io/sitemap.xml)
 
## 6. 다음에 검색되도록 하기.
1. 다음 검색등록 사이트 접속 [https://register.search.daum.net/index.daum]
2. 등록 탭에서 블로그 등록을 선택하고 주소를 입력한다. 
  

검색엔진에 노출되기까지는 시간이 어느정도 걸린다고 한다. 얼마나 걸릴지는 지켜봐야겠다. 

참고자료: 
 - http://jinyongjeong.github.io/2017/01/13/blog_make_searched/
 - https://wayhome25.github.io/etc/2017/02/20/google-search-sitemap-jekyll/
 - https://devmjun.github.io/archive/addSearch

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

---

