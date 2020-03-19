---
layout: post
title:  "docker를 통한 ngrinder 설치 및 사용"
date:   2019-07-02 00:00:00 +0900
categories: ngrinder
comments: true
tags: [ngrinder, Docker, 엔그라인더, 부하테스트]
---

---

# nGrinder 란?
네이버에서 성능 측정 목적으로 개발된 오픈소스 프로젝트이며, The Grinder라는 오픈소스 기반으로 개발되었다. nGrinder는 서버에 대한 부하 테스트를 하는 것으로 서버의 성능을 측정할 수 있다.

## nGrinder Architecture
Controller, Agent, Target 서버로 나누어져 있음.


 <img src="{{ site.baseurl }}/public/post/ngrinder/ngrinder-system-architecture.png"/>

 [ngrinder system 아키텍쳐 이미지]

### Controller
 - 퍼포먼스 테스팅(부하테스트)를 위해 웹 인터페이스를 제공
 - 테스트 프로세스를 체계화
 - 테스트 결과를 수집해 통계로 보여줌

### Agent: Controller의 명령을 받아 실행.
 - agent 모드가 실행될 때 target이 된 머신에 프로세스와 스레드를 발생시켜 부하를 발생.
 - moniter 모드가 실행되면 대상 시스템의 cpu와 memory를 모니터링.

### Target: 부하테스트를 받는 머신.


## Controller 및 Agent 설치 및 설명 (도커로 설치)

### docker
직접 다운받아서 설치해도 되지만 귀찮은 것들을 만져줘야하는 번거로움이 있어 docker사용을 추천한다.

### docker 설치 참고 자료
 - mac: https://docs.docker.com/docker-for-mac/install/
 - windows: https://docs.docker.com/docker-for-windows/install/
 - linux: https://docs.docker.com/install/linux/docker-ce/centos/


#### nGrinder controller:
```
$ docker run -d -v ~/ngrinder-controller:/opt/ngrinder-controller -p 8080:80 -p 16001:16001 -p 12000-12009:12000-12009 ngrinder/controller:3.4
```

> grinder controller는 포트옵션으로 웹 포트, 에이전트와의 연결, 부하관리를 위한 포트들로 구성되어 있으며 자세한 내용은 https://hub.docker.com/r/ngrinder/controller/ 에서 확인할 수 있다.

#### nGrinder agent:

```
$ docker run -v ~/ngrinder-agent:/opt/ngrinder-agent -d ngrinder/agent:3.4 controller_ip:controller_port
agent는 controller_ip:controller_webport 부분을 옵션 argument로 전달해야 한다.
```

> ex) controller가 떠있는 instance의 public ip가 192.168.100.12이고 80번 포트를 웹포트로 열었다면 "192.168.100.12:80" 을 뒤에 붙여주면 된다.

## 실행하기
브라우저에서 아래 주소 입력
```
http://controller_ip:port
id: admin
pw: admin
```

Agent: Any ==> Controller: 16001
Agent: Any ==> Controller: 12000 ~ 1200x

 ==> 는 단방향 통신을 뜻함.

### 16001 : 테스트를 하지 않은 에이전트가 컨트롤러에게 "할 일 없으니 테스트 가능" 이라는 메세지를 알려주는 포트
 - 컨트롤러는 "테스트가 실행하는데 해당 테스트는 1200x에서 발생하니, 해당 포트에 접속해서 테스트 실행 준비" 라는 메세지를 에이전트에게 지시를 한다.

### 12000 ~ 1200x 포트는 "테스트 실행, 테스트 종료" 와 같은 컨트롤러 명령어와 에이전트별 테스트 실행 통계를 초별로 수집하는 포트.

## nGrinder 관련 용어 설명

 <img src="{{ site.baseurl }}/public/post/ngrinder/ngrinder-controller.png"/>

 [ngrinder 설정화면]

 - vuser : virtual user로 동시에 접속하는 유저의 수를 의미. (vuesr = agent * process * thread)
 - TPS : 초당 트랜잭션의 수 - 초당 처리 수
 - 트랜잭션 : HTTP Request가 성공할 때마다, 트랜잭션 수가 1씩 증가.
 - Peak TPS : 초당 처리 수의 최대치.
 - Response Time : 사용자가 request한 시점에서 시스템이 response할 때까지 걸린 시간.
 - Think Time : 사용자에게 전달된 정보는 사용자가 해당 내용을 인지하고 다음 동작을 취할 때까지의 생각하는 시간

## 스크립트
 - nGrinder 에 로그인을 하면 스크립트 관리 화면에서 SVN URL을 확인할 수 있다. (ngrinder 에 svn 이 내장되어있다.)

 <img src="{{ site.baseurl }}/public/post/ngrinder/ngrinder-script-svn.png"/>

 [ngrinder 스크립트 관리]

 > 예를 들어 위 화면에서는 http://ngrinder-staging.nhncorp.com:8080/svn/admin/project SVN URL로 Groovy 메이븐 프로젝트를 접근할 수 있다.

### 인텔리제이에서 엔그라인더의 그루비 스크립트 메이븐 임포트 시 참고자료
 - https://github.com/naver/ngrinder/wiki/Import-Groovy-Maven-Project-in-IntelliJ

### 스크립트 작성 시 참고
 - https://junoyoon.tistory.com/entry/Groovy-%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EA%B5%AC%EC%A1%B0?category=487801

### 인텔리제이에서 엔그라인더 그루비 스크립트 실행 시 vm option 에 추가.
-javaagent:/Users/jmlim/.m2/repository/net/sf/grinder/grinder-dcr-agent/3.9.1/grinder-dcr-agent-3.9.1.jar

### Get, Post 요청 샘플 스크립트.

 - application/json 요청 (jsonBody 형태의 요청일 경우..)

~~~groovy
import HTTPClient.Cookie
import HTTPClient.CookieModule
import HTTPClient.HTTPResponse
import HTTPClient.NVPair
import groovy.json.JsonOutput
import net.grinder.plugin.http.HTTPPluginControl
import net.grinder.plugin.http.HTTPRequest
import net.grinder.script.GTest
import net.grinder.scriptengine.groovy.junit.GrinderRunner
import net.grinder.scriptengine.groovy.junit.annotation.BeforeProcess
import net.grinder.scriptengine.groovy.junit.annotation.BeforeThread
import org.junit.Before
import org.junit.Test
import org.junit.runner.RunWith

import static net.grinder.script.Grinder.grinder
import static org.hamcrest.Matchers.is
import static org.junit.Assert.assertThat

/**
 * ngrinder get, post 요청 샘플 스크립트.
 */
@RunWith(GrinderRunner)
class SampleGetPostRunner {

    public static GTest test
    public static HTTPRequest request
    public static NVPair[] headers = []
    public static Cookie[] cookies = []

    static String url = "http://테스트할아이피:포트"
    static String commonPath = "/test"

    static String findUrl = url + commonPath + "/find";
    static String createUrl = url + commonPath + "/create"

    @BeforeProcess
    static void beforeProcess() {
        HTTPPluginControl.getConnectionDefaults().timeout = 6000
        test = new GTest(1, "api.test.com")
        request = new HTTPRequest()
        // Set header datas
        List<NVPair> headerList = new ArrayList<NVPair>()
        headerList.add(new NVPair("Content-Type", "application/json"))
        headerList.add(new NVPair("Authorization", "Bearer 토큰토큰"))

        headers = headerList.toArray()
        grinder.logger.info("before process.");
    }

    @BeforeThread
    void beforeThread() {
        test.record(this, "test")
        grinder.statistics.delayReports = true;
        grinder.logger.info("before thread.");
    }

    @Before
    void before() {
        request.setHeaders(headers)
        cookies.each { CookieModule.addCookie(it, HTTPPluginControl.getThreadHTTPClientContext()) }
        grinder.logger.info("before thread. init headers and cookies");
    }

    @Test
    void test() {
        find();

        create();
    }


    /**
     * get test
     */
    void find() {
        HTTPResponse result = request.GET(findUrl)
        resultCheck(result)
    }

    /**
     * post test
     */
    void create() {
        String goodsId = "1234";
        Integer quantity = 2;

        Map<String, Object> paramData = new HashMap<>();
        List<Map<String, Object>> items = new ArrayList<>();
        Map<String, Object> item = new HashMap<>();
        item.put("goods_id", goodsId);
        item.put("quantity", quantity);
        items.add(item);
        paramData.put("items", items);

        String json = JsonOutput.toJson(paramData);
        HTTPResponse result = request.POST(createUrl, json.getBytes())
        resultCheck(result)
    }

    /**
     * 결과값 체크
     * @param result
     */
    void resultCheck(result) {
        if (result.statusCode == 301 || result.statusCode == 302) {
            grinder.logger.warn("Warning. The response may not be correct. The response code was {}.", result.statusCode);
        } else {
            assertThat(result.statusCode, is(200));
        }
    }
}

~~~

- 일반 form post 요청일 경우

~~~
import HTTPClient.HTTPResponse
import HTTPClient.NVPair
import net.grinder.plugin.http.HTTPPluginControl
import net.grinder.plugin.http.HTTPRequest
import net.grinder.script.GTest
import net.grinder.scriptengine.groovy.junit.GrinderRunner
import net.grinder.scriptengine.groovy.junit.annotation.BeforeProcess
import net.grinder.scriptengine.groovy.junit.annotation.BeforeThread
import org.junit.Before
import org.junit.Test
import org.junit.runner.RunWith

import static net.grinder.script.Grinder.grinder
import static org.hamcrest.Matchers.is
import static org.junit.Assert.assertThat

/**
 * ngrinder get, post 요청 샘플 스크립트.
 */
@RunWith(GrinderRunner)
class TestRunner {

    public static GTest test
    public static HTTPRequest request
    public static NVPair[] headers = []

    static String url = "https://타겟url"
    static String commonPath = "/api"

    static String appinitUrl = url + commonPath + "/app"

    @BeforeProcess
    static void beforeProcess() {
        HTTPPluginControl.getConnectionDefaults().timeout = 6000
        test = new GTest(1, "타겟명")
        request = new HTTPRequest()
        grinder.logger.info("before process.");
    }

    @BeforeThread
    void beforeThread() {
        test.record(this, "test")
        grinder.statistics.delayReports = true;
        grinder.logger.info("before thread.");
    }

    @Before
    void before() {
        request.setHeaders(headers)
    }

    @Test
    void test() {
        NVPair param1 = new NVPair("appcode", "999");
        NVPair param2 = new NVPair("appversion", "x");
        NVPair param3 = new NVPair("hashcode", "1122334455667788");

        NVPair[] params = [param1, param2, param3]

        HTTPResponse result = request.POST(appinitUrl, params)
        resultCheck(result)
    }

    /**
     * 결과값 체크
     * @param result
     */
    void resultCheck(result) {
        println result
        if (result.statusCode == 301 || result.statusCode == 302) {
            grinder.logger.warn("Warning. The response may not be correct. The response code was {}.", result.statusCode);
        } else {
            assertThat(result.statusCode, is(200));
        }
    }
}

~~~


- 파일 업로드 post 요청 (multipart/form-data)


~~~
import HTTPClient.Codecs
import HTTPClient.HTTPResponse
import HTTPClient.NVPair
import net.grinder.plugin.http.HTTPPluginControl
import net.grinder.plugin.http.HTTPRequest
import net.grinder.script.GTest
import net.grinder.scriptengine.groovy.junit.GrinderRunner
import net.grinder.scriptengine.groovy.junit.annotation.BeforeProcess
import net.grinder.scriptengine.groovy.junit.annotation.BeforeThread
import org.junit.Before
import org.junit.Test
import org.junit.runner.RunWith

import static net.grinder.script.Grinder.grinder
import static org.hamcrest.Matchers.is
import static org.junit.Assert.assertThat

/**
 * ngrinder get, post 요청 샘플 스크립트.
 */
@RunWith(GrinderRunner)
class MultipartTestRunner {

    public static GTest test
    public static HTTPRequest request
    public static NVPair[] headers = []

    static String url = "http://타겟url"
    static String commonPath = "/api"

    static String moreUploadUrl = url + commonPath + "/upload-app-file"

    @BeforeProcess
    static void beforeProcess() {
        HTTPPluginControl.getConnectionDefaults().timeout = 6000
        test = new GTest(1, "http://타겟명")
        request = new HTTPRequest()
        grinder.logger.info("before process.");
    }

    @BeforeThread
    void beforeThread() {
        test.record(this, "test")
        grinder.statistics.delayReports = true;
        grinder.logger.info("before thread.");
    }

    @Before
    void before() {
        headers = [
                new NVPair("Content-Type", "multipart/form-data")
        ]
        request.setHeaders(headers)
    }

    @Test
    void moreUpload() {
        NVPair param1 = new NVPair("appcode", "999");
        NVPair param2 = new NVPair("appversion", "17");
        NVPair param3 = new NVPair("hashcode", "11223344556677");

        NVPair[] params = [param1, param2, param3]

        NVPair[] files = [new NVPair("userfile", "경로/파일명.확장자")]

        def data = Codecs.mpFormDataEncode(params, files, headers)
        HTTPResponse result = request.POST(moreUploadUrl, data)

        resultCheck(result)
    }

    /**
     * 결과값 체크
     * @param result
     */
    void resultCheck(result) {
        println result
        if (result.statusCode == 301 || result.statusCode == 302) {
            grinder.logger.warn("Warning. The response may not be correct. The response code was {}.", result.statusCode);
        } else {
            assertThat(result.statusCode, is(200));
        }
    }
}

~~~



참고자료:
 - https://brownbears.tistory.com/25
 - http://blog.naver.com/PostView.nhn?blogId=simpolor&logNo=221318391959&parentCategoryNo=&categoryNo=27&viewDate=&isShowPopularPosts=true&from=search
 - https://junoyoon.tistory.com/entry/%EC%9D%B4%ED%81%B4%EB%A6%BD%EC%8A%A4%EC%97%90-Groovy-%EB%A9%94%EC%9D%B4%EB%B8%90-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EC%9E%84%ED%8F%AC%ED%8A%B8
 - https://gist.github.com/ihoneymon/a83b22a42795b349c389a883a7bbf356

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

---
