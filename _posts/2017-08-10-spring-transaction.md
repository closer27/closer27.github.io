---
layout: post
title: "Spring AOP"
date: 2017-08-10
categories: backend
tags: [backend, spring, aop]
comment: yes
---

# Day 2 - 스프링 AOP(Aspect Oriented Programming)
## 스프링 트랜잭션 (Spring Transaction)

제약
- XML 기반의 AOP 설정만 가능
- `<aop:advisor>` 엘리먼트만 사용

applicationContext.xml
``` xml
<beans xmlns="...
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="...
                           http://www.springframework.org/schema/tx
                           http://www.springframework.org/schema/tx/spring-tx-4.2.xsd">
```

build.gradle
```
    compile group: 'org.springframework', name: 'spring-jdbc', version: '4.2.0.RELEASE'
    compile group: 'org.springframework', name: 'spring-tx', version: '4.2.0.RELEASE'
```


가장 먼저 등록하는 것은 트랜잭션 관리자 클래스
- 어떤 기술을 이용하여 데이터베이스 연동을 처리했냐에 따라 관리자가 달라진다.
- 모든 트랜잭션 관리자는 PlatformTransactionManager 인터페이스를 구현한 클래스

``` java
public interface PlatformTransactionManager {
	TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;
	void commit(TransactionStatus status) thorws TransactionException;
    void rollback(TransactionStatus status) thorws TransactionException;
}
```

지금은 JDBC 기반으로 동작하기 때문에 DataSourceTransactionManager 클래스를 이용하여 처리

applicationContext.xml

``` xml
<!-- DataSource 설정 -->
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
          ^^^^^^^^^^
    <property name="driverClassName" value="org.h2.Driver" />
    <property name="url" value="jdbc:h2:mem:testdb" />
    <property name="username" value="sa" />
    <property name="password" value="" />
</bean>

<!-- Transaction 설정 -->
<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource" />
                                     ^^^^^^^^^^
</bean>
```

DataSourceTransactionManager를 등록했다고 자동으로 되는게 아님

비즈니스 메소드를 실행하다가 예외가 발생하면 해당 메소드에 대한 트랜잭션을 롤백하고 정상수행되면 커밋하면 된다. -> 트랜잭션을 커밋, 롤백하기 위한 객체는 DataSourceTransactionManager로 등록했다.
-> 이제 이 관리자를 이용하여 트랜잭션을 제어하는 어드바이스 등록이 필요


``` xml
<tx:advice id="txAdvice" transaction-manager="txManager">
               ^^^^^^^^
    <tx:attributes>
        <tx:method name="get*" read-only="true"/>
        <tx:method name="*"/>
    </tx:attributes>
</tx:advice>

<aop:config>
    <aop:pointcut id="txPointcut" expression="execution(* io.icednut.spring.exercise..*(..)))"/>
    	              ^^^^^^^^^^
    <aop:advisor pointcut-ref="txPointcut" advice-ref="txAdvice"/>
    	 ^^^^^^^  	       ^^^^^^^^^^              ^^^^^^^^
         <!-- aspect 사용 불가능 -->
</aop:config>
```

트랜잭션 관리 기능의 어드바이스는 직접 구현하지 않으며, 스프링 컨테이너가 `<tx:advice>` 설정을 참조하여 자동으로 생성한다 -> 트랙잭션 관리 어드바이스 객체의 메소드나 클래스 이름을 확인할 수 없다는 뜻; id로 참조만 한다.


`<tx:method>` 엘리먼트는 get으로 시작되는 모든 메소드는 `read-only="true"`, 즉 읽기 전용으로 처리되어 트랜잭션 관리 대상에서 제외하고 나머지는 모두 포함하는 것.

속성
- name : 트랜잭션이 적용될 메소드 이름 지정
- read-only : 읽기 전용 여부 지정(기본값 false)
- no-rollback-for : 트랜잭션을 롤백하지 않을 예외 지정
- rollback-for : 트랜잭션을 롤백할 예외 지정

<img src="/assets/images/20170810_1.jpg">

테스트

—
Referencing

: 채규태 (2016), 스프링 퀵 스타트, 경기 부천: 루비페이퍼
