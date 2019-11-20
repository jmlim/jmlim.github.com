---
layout: post
title:  "[Spring] Spring boot yml 파일에서 한글 또는 특수문자로 된 key (map)을 받으려고 할 때 에러날 경우"
date:   2019-11-20 20:05:00 +0900
categories: Spring
comments: true
tags: [Spring, Spring boot, yml, yaml, 한글키, map]
---

---

> 사실 아래와 같이 map 으로 된 키를 한글로 받는 경우는 별로 없긴하다. <br/>
> 그런데 최근에 업무를 하면서 아래와 같은 케이스가 발생하여 해결해야 하는 상황에 놓인적이 있어 방법을 기억할 겸 공유한다.
> (아래 부분은 key 에 / 또는 특수문자가 들어갈 경우도 처리가 가능하다.)

## 예제

### application.yml 
~~~yaml
jmlim:
  testMap:
    홍길동: test1
    아버지: test2
~~~

### Configuration class
~~~java
@Data
@ConfigurationProperties(prefix="jmlim")
public class JmlimProperties {
	private Map<String, String> testMap;
}

~~~

### Test class
~~~ java
@Transactional
@RunWith(SpringRunner.class)
@SpringBootTest(classes=DataManage.class)
public class JmlimTest extends BaseTransactionalTest{

	@Autowired
	private JmlimProperties jmlimConfig;
	
	@Test
	public void read() {
		System.out.println(jmlimConfig.toString());
	}
	
}
~~~
### error Exception
~~~
***************************
APPLICATION FAILED TO START
***************************

Description:

Failed to bind properties under 'jmlim.test-map' to java.util.Map<java.lang.String, java.lang.String>:

    Reason: No converter found capable of converting from type [java.lang.String] to type [java.util.Map<java.lang.String, java.lang.String>]

Action:

Update your application's configuration
~~~

### 해결방법
#### application.yml 파일을 아래와 같이 수정
~~~yaml
jmlim:
  testMap:
    "[홍길동]": test1
    "[아버지]": test2
~~~

### 결과
~~~
JmlimProperties(testMap={홍길동=test1, 아버지=test2})
~~~

### 참고자료 : 
 - https://github.com/spring-projects/spring-boot/issues/13404

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

---
