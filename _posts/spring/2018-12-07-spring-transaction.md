---
layout: post
title:  "Spring 에서의 트랜잭션 처리."
date:   2018-12-07 10:00:00 +0900
categories: Spring
comments: true
tags: [Spring, Java, Transaction]
---

---


먼저 트랜잭션이란? <br/>
데이터베이스 연산들의 논리적 단위이며 트랜잭션 내 모든 연산들이 정상적으로 완료되지 않으면 아무 것도 수행되지 않은 원래 상태로 복원되어야 한다.


예를들어 친구에게 인터넷 뱅킹으로 10,000원을 송금할경우 나의 계좌에서 10,000원을 줄이고 친구의 계좌에 10,000원을 증가시켜야 한다.
하지만 알 수 없는 오류로 인해 나의 계좌에서는 10,000원이 줄었지만 친구 계좌에는 10,000원이 증가되지 않는다면?
나의 10,000원은 그냥 공중으로 증발해버리는 사고가 발생한다.

이러한 경우가 생기지 않도록 중간에 오류가 발생하면 다시 처음부터 송금을 하도록 하는 것이 rollback이다.

위 송금 과정을 하나의 트랜잭션이라 볼 수 있으며 데이터베이스 연산들의 논리적 단위라 할 수 있다.
즉 한 번 질의가 실행되면 질의가 모두 수행되거나 모두 수행되지 않는 작업수행의 논리적 단위인 것이다.


스프링 프레임워크를 사용하는 주목할 만한 이유중 하나가 광범위한 트랜잭션 지원이다. <br/>
대표적으로 선언적인 트랜잭션 관리 지원을 많이 사용하며,
<br/>
어노테이션 @Transactional을 활용하여 트랜잭션 처리를 한다.
단 몇줄의 코드만으로 다이나믹 프록시와 AOP라는 기술을 통해 크게는 인터페이스 단위에서 작게는 메서드까지 세분화하여 트랜잭션을 컨트롤 할 수 있다.

설정하기
--
@Transactional 어노테이션 사용을 위해, Spring  bean 설정은 아래와 같이 하면 된다.
(스프링 부트에서는 그냥 쓰면 된다.)

```xml
....
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
	destroy-method="close">
	<property name="driverClassName" value="org.apache.derby.jdbc.ClientDriver" />
	<property name="url" value="jdbc:derby://localhost:1527/sample" />
	<property name="username" value="user" />
	<property name="password" value="jmlim123" />
</bean>

<!-- transaction manager, use JtaTransactionManager for global tx -->
<bean id="transactionManager"
	class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource" />
</bean>

<!-- enable transaction demarcation with annotations -->
<tx:annotation-driven />
...

```

@Transactional이 적용되어 있을 경우, 이 클래스에 트랜잭션 기능이 적용된 프록시 객체가 생성된다. 
이 프록시 객체는 @Transactional이 포함된 메소드가 호출 될 경우, PlatformTransactionManager를 사용하여 트랜잭션을 시작하고, 정상 여부에 따라 Commit 또는 Rollback 한다.

정상 여부는 RuntimeException이 발생했는지 기준으로 결정되며, 
RuntimeException 외 다른 Exception(대표적으로 SQLException이 있다.)에도 트랜잭션 롤백처리를 적용하고 싶으면 @Transactional의 rollbackFor 속성을 활용하면 된다.


* 아래는 사용 예제 이다.


```java
	/**
	 * <pre>
	 *  
	 *  설정시간이 지난 미결제 예약 레코드 삭제 및 이력 레코드 삽입 
	 * 
	 * </pre>
	 * 
	 * @throws Exception 오류가 발생할 경우 예외를 발생시킵니다.
	 */
	@Transactional(readOnly = true, propagation = Propagation.REQUIRED, rollbackFor={Exception.class})
	public int deleteExpiredReserve() throws Exception {
		
		int count = 0;
		
		Date nowTime = new Date();
		SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");		
		
		List<HashMap<String, Object>> m_reservCdList = reserveDAO.selectExpireReserveCdList(format.format(nowTime));
		
		List<String> reservCdList = new ArrayList<String>();
		for(HashMap<String, Object> obj : m_reservCdList){
			reservCdList.add(obj.get("RESERV_CD").toString());
		}
		
		if(reservCdList.size() > 0){
			reserveDAO.insertExpiredReserveMaster(format.format(nowTime));
			count += reserveDAO.deleteExpiredReserveMaster(reservCdList);
			count += reserveDAO.deleteExpiredReserve(reservCdList);				
		}		
		
		return count;
	}
```

트랜잭션을 처리하다 보면 자주 발생하게 되는 상황이 있다. <br/>
바로 트랜잭션 동작 도중 다른 트랜잭션을 호출(실행)하는 상황이다. <br/>
피호출 트랜잭션의 입장에서는 호출한 쪽의 트랜잭션을 그대로 사용할 수도 있고, 새롭게 트랜잭션을 생성할 수도 있다.<br/>

전자의 경우 중간에 오류가 발생하면 모든 트랜잭션이 롤백될 것이고, 후자의 경우 오류가 발생한 트랜잭션이 롤백 될 것이다.<br/>
이러한 트랜잭션 관련 설정은 @Transactional의 propagation 속성을 통해 지정할 수 있다.

propagation에 지정할 수 있는 값은 다양하다. 그것들을 간단히 정리하면 아래와 같다.

1. REQUIRED: 현재 진행중인 트랜잭션이 있으면 그것을 사용하고, 없으면 생성한다. [DEFAULT 값]
2. MANDATORY: 현재 진행중인 트랜잭션이 없으면 Exception 발생. 없으면 생성한다.
3. REQUIRES_NEW: 항상 새로운 트랜잭션을 만듦 (트랜잭션을 분리)
4. SUPPORTS: 현재 진행중인 트랜잭션이 있으면 그것을 사용. 없으면 그냥 진행.
5. NOT_SUPPORTED: 현재 진행중인 트랜잭션이 있으면 그것을 미사용. 없으면 그냥 진행.
6. NEVER: 현재 진행중인 트랜잭션이 있으면 Exception. 없으면 그냥 진행.


* 트랜잭션 주의 사항
> 트랜잭션은 꼭 필요한 최소한의 범위로 수행해야 한다. 
왜냐하면 일반적으로 데이터베이스 커넥션은 갯수가 제한적이기 때문에 각 트랜잭션에서 커넥션을 소유하는 시간이 길어진다면, 
그 이후에 사용 가능한 커넥션의 갯수가 줄어든다. 그러다 어느 순간 다른 트랜잭션이 수행될 때, 커넥션이 부족하여 커넥션을 받기 위해 기다리는 상황이 발생할 수 있다.


참고: 
- http://virusworld.tistory.com/120
- https://preamtree.tistory.com/97
- http://victorydntmd.tistory.com/129 [victolee]

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
---
