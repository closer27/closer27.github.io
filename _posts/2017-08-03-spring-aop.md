---
layout: post
title: "Spring AOP"
date: 2017-08-03
categories: backend
tags: [backend, spring, aop]
comment: yes
---

# Day 2 - 스프링 AOP(Aspect Oriented Programming)
## 개요

낮은 결합도 높은 응집도는 기본, DI는 낮은 결합도를 위한 것이라면 **AOP는 높은 응집도를 위한 것**

엔터프라이즈 애플리케이션들은 보통 핵심 비지니스 로직은 몇 줄 안되고 주로 로깅이나 예외, 트랜잭션 처리 같은 부가 코드가 대부분이다.
: -> 비지니스 메소드 복잡도는 증가
: -> 비지니스 메소드들마다 매번 반복해야한다는 것이 중요

해결 방법

1. 단순 복사 붙여넣기 => 코드 중복이 많아져 코드 분석과 유지보수를 어렵게 만듬.
2. **AOP를 통해 부가적인 공통 코드들을 효율적으로 관리**

---
## AOP에서 중요한 것
### 관심 분리(Separation of Concerns)
- 로깅, 예외, 트랜잭션 처리 같은 코드들은 **횡단 관심(Crosscutting Concerns)**
- 핵심 비지니스 로직은 **핵심 관심(Core Concerns)**

<img src="/assets/images/20170803_1.jpg">

LogAdvice 클래스를 변경하거나 printLog 메서드의 시그니처가 변경되는 상황에서는 유연하게 대처 불가능
: AOP는 핵심 관심과 횡단 관심을 완벽하게 분리할 수 없는 OOP의 한계를 극복하도록 도와준다.
: **소스상의 결합은 발생하지 않는다.** => AOP를 사용하는 목적

---
## 용어 및 기본 설정

### 조인포인트(JoinPoint)

: 클라이언트가 호출하는 모든 비즈니스 메소드, 모든 메소드를 조인포인트로 생각하면 된다. -> 이 중에서 포인트컷이 결정 됨.

### 포인트컷(Pointcut)

: 포인트컷은 필터링된 조인포인트를 의미한다.

<img src="/assets/images/20170803_2.jpg">
: 포인트컷으로 메소드 필터링

포인트컷을 위해 클래스와 패키지는 물론이고 메소드 시그니처까지 정확하게 지정할 수 있다.

app.xml
``` xml
    <bean id="log" class="io.icednut.spring.exercise.common.Log4jAdvice"/>
    <aop:config>
        <aop:pointcut id="allPointcut" expression="execution(* io.icednut.spring.exercise..*Impl.*(..))"/>
        <aop:aspect ref="log">
            <aop:before pointcut-ref="allPointcut" method="printLog"/>
        </aop:aspect>
    </aop:config>
```
allPointcut이라는 id의 포인트컷은 Impl로 끝나는 모든 클래스의 모든 메서드를 포인트컷으로 설정했다.

포인트컷은 `<aop:pointcut>` 엘리먼트로 선언 id 속성으로 식별
expression 속성으로 필터링되는 메소드를 지정할 수 있다.

<img src="/assets/images/20170803_3.jpg">

### 어드바이스(Advice)

: 어드바이스는 횡단 관심에 해당하는 공통 기능의 코드를 의미하며 독립된 클래스의 메서드로 작성된다. 언제 동작할지는 스프링 설정에서 결정.

* before
* after
* after-returning
* after-throwing
* around

### 위빙(Weaving)

: 횡단 관심 메서드가 삽입되는 과정을 의미한다. 이 위빙을 통해 비지니스 메소드를 수정하지 않고 횡단 관심에 해당하는 기능을 추가하거나 변경 가능.

### 애스팩트(Aspect) 또는 어드바이저(Advisor)

: 애스펙트는 포인트컷과 어드바이스의 결합 => 어떤 포인트컷 메서드에 어떤 어드바이스 메소드를 실행할지 결정한다.

---
## AOP 엘리먼트
### `<aop:config>` 엘리먼트

스프링 설정 파일 내 여러번 사용 가능,

``` xml
<beans xmlns="http://www.springframework.org/schema/beans" ...>
    <aop:config>
        <aop:pointcut …/>
        </aop:aspect …></aop:aspect>
    </aop:config>
</beans>
```

### `<aop:pointcut>` 엘리먼트

포인트컷을 지정하기 위해 사용 `<aop:config>`이나 `<aop:aspect>`의 자식 엘리먼트로 사용
`<aop:aspect>` 하위에 설정된 포인트컷은 해당 `<aop:aspect>`에서만 사용할 수 있다.
여러개 정의 가능

``` xml
<aop:config>
    <aop:pointcut id="allPointcut" expression="execution(* io.icednut.spring.exercise..*Impl.*(..))"/>
    <aop:aspect ref="log">
        <aop:before pointcut-ref="allPointcut" method="printLog"/>
    </aop:aspect>
</aop:config>
```

"allPointcut" 이라는 포인트컷은  io.icednut.spring.exercise 패키지로 시작하는 클래스 중 Impl로 끝나는 클래스의 모든 메소드를 포인트컷으로 설정, 애스펙트 설정에서 app:before 엘리먼트의 pointcut-ref 속성으로 포인트컷을 참조

### `<aop:aspect>` 엘리먼트
해당 관심에 해당하는 포인트컷 메소드와 횡단 관심에 해당하는 어드바이스 메소드를 결합하기 위해 사용. 어떻게 설정하냐에 따라 위빙 결과가 달라지므로 AOP에서 가장 중요한 설정

### `<aop:advisor>` 엘리먼트
aspect와 같은 기능, 트랜잭션 설정 같은 몇몇 특수한 경우는 어드바이저를 사용해야 함.
AOP에서 애스펙트로 설정하기 위해서는 어드바이스의 아이디와 메소드 이름을 알아야하지만 이를 모를 경우 사용할 수 없다.

``` xml
<!-- Transaction 설정 -->
<bean id="txManager" class="org.springframework.orm.jap.JpaTransactionManager">
    <property name="entityManagerFactory" ref="entityManagerFactory"></property>
</bean>
<tx:advice id="txAdvice" transaction-manager="txManager">
    <tx:attributes>
        <tx:method name="get*" read-only="true"/>
        <tx:method name="*"/>
    </tx:attributes>
</tx:advice>
<aop:config>
    <aop:pointcut id="allPointcut" expression="execution(* io.icednut.spring.exercise..*Impl.*(..))"/>
    <aop:advisor pointcut-ref="allPointcut" advice-ref="txAdvice"/>
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
</aop:config>
```
스프링 컨테이너는 `<tx:advice>` 엘리먼트를 해석하여 트랜잭션 관리 기능의 어드바이스 객체를 생성한다. txAdvice 설정 방법에 따라 동작은 달라짐.
=> 문제는 어드바이스의 아이디는 확인되지만 메소드 이름은 확인할 방법이 없다. 이럴 때 `<aop:advisor>`를 써야함.

---
## 포인트컷 표현식

### `execution(*io.icednut.spring.exercise..*Impl.get*(..))`

| execution( | * | io.icdenut.spring.exercise.. | *Impl | . | get*(..) | ) |
|------------|----------|------------------------------|----------|---|-------------------|---|
|  | 리턴타입 | 패키지경로 | 클래스명 |  | 메소드명 매개변수 |  |

#### 리턴 타입

| 타입 | 설명 |
|--------------------------------|--------------------------------------------------------------------------------|
| `*` | 모든 리턴타입허용 |
| `void` | 리턴타입 void인 메소드 선택 |
| `!void` | void가 아닌 메소드 선택 |

#### 패키지 경로

| 패키지경로 | 설명 |
|--------------------------------|--------------------------------------------------------------------------------|
| `io.icednut.spring.exercise` | 정확하게 패키지 선택 |
| `io.icednut.spring.exercise..` | 패키지로 시작하는 모든 패키지 선택 |
| `io.icednut.spring..impl` | io.icednut.spring으로 시작하면서 마지막 패키지 이름이 Impl로 끝나는 패키지 선택 |

#### 클래스명

| 클래스명 | 설명 |
|--------------------------------|--------------------------------------------------------------------------------|
| BoardServiceImpl | 정확한 클래스 선택 |
| *Impl | 이름이 Impl로 끝나는 클래스 선택 |
| BoardService+ | 해당 클래스로 파생된 모든 자식 클래스 선택, 인터페이스 구현 된 모든 클래스 선택 |

#### 메소드명, 매개변수

| 메소드명, 매개변수 | 설명 |
|--------------------------------|--------------------------------------------------------------------------------|
| `*` | 모든 메소드 선택 |
| `get*(..)` | 이름이 get으로 시작하는 모든 메소드 선택 |


---
## 어드바이스 동작 시점
`<aop:aspect>` 아래 삽입

* Before : 비지니스 메소드 실행 전 동작 ; `<aop:before>`
* After
    * After Returning : 비지니스 메소드가 성공적으로 리턴되면 동작 ; `<aop:after-returning>`
    * After Throwing : 비지니스 메소드 실행 중 예외가 발생하면 동작(try~catch 블록에서 catch 블록에 해당) ; `<aop:after-throwing>`
    * After : 실행 된 후 무조건 실행(try~catch~finally 블록에서 finally 블록에 해당) ; `<aop:after>`
* Around : 비지니스 메소드 실행 전후에 처리할 로직을 삽입할 수 있음 `<aop::after-around>`


### Before 어드바이스

``` xml
<bean id="before" class="io.icednut.spring.exercise.beforeAdvice"/>
<aop:config>
    <aop:pointcut id="allPointcut" expression="execution(* io.icednut.spring.exercise..*Impl.*(..))"/>
    <aop:aspect ref="before">
        <aop:before pointcut-ref="allPointcut" method="beforeLog"/>
    </aop:aspect>
</aop:config>
```

### After Returning 어드바이스

``` xml
<bean id="afterReturning" class="io.icednut.spring.exercise.AfterRetruningAdvice"/>
<aop:config>
    <aop:pointcut id="getPointcut" expression="execution(* io.icednut.spring.exercise..*Impl.get*(..))"/>
    <aop:aspect ref="before">
        <aop:after-returning pointcut-ref="getPointcut" method="afterLog"/>
    </aop:aspect>
</aop:config>
```

### After Throwing 어드바이스

``` xml
<bean id="afterThrowing" class="io.icednut.spring.exercise.AfterThrowingAdvice"/>
<aop:config>
    <aop:pointcut id="allPointcut" expression="execution(* io.icednut.spring.exercise..*Impl.*(..))"/>
    <aop:aspect ref="afterThrowing">
        <aop:after-throwing pointcut-ref="allPointcut" method="exceptionLog"/>
    </aop:aspect>
</aop:config>
```

### After 어드바이스

``` xml
<bean id="after" class="io.icednut.spring.exercise.AfterAdvice"/>
<aop:config>
    <aop:pointcut id="allPointcut" expression="execution(* io.icednut.spring.exercise..*Impl.*(..))"/>
    <aop:aspect ref="afterThrowing">
        <aop:after-throwing pointcut-ref="allPointcut" method="exceptionLog"/>
    </aop:aspect>

    <aop:aspect ref="after">
        <aop:after pointcut-ref="allPointcut" method="finallyLog"/>
    </aop:aspect>
</aop:config>
```

동시에 지정하게 되면 예외가 발생한 상황에서도 `<aop:after>`로 설정한 finallyLog가 먼저 실행되고 exceptionLog()가 실행되는것을 확인할 수 있다.

### Around 어드바이스

``` xml
<bean id="after" class="io.icednut.spring.exercise.AfterAdvice"/>
<aop:config>
    <aop:pointcut id="allPointcut" expression="execution(* io.icednut.spring.exercise..*Impl.*(..))"/>
    <aop:aspect ref="after">
        <aop:around pointcut-ref="allPointcut" method="aroundLog"/>
    </aop:aspect>
</aop:config>
```

---
## JoinPoint와 바인드 변수

예외가 발생한 비즈니스 메소드 이름이 무엇인지, 그 메소드가 속한 클래스와 패키지 정보는 무엇인지 알아야 정확한 예외 처리 로직을 구현할 수 있다.
=> JoinPoint 인터페이스 제공

Signature getSignature() : 클라이언트가 호출한 메소드의 시그니처(리턴타입, 이름, 매개변수) 정보 리턴
* String getName() : 메서드 이름 리턴
* String toLongString() : 메서드 리턴 타임, 이름, 매개변수 패키지경로까지 포함해서 리턴
* String toShorString() :  메[소드 시그니처를 축약한 문자열로 리턴
Object getTarget() : 클라이언트가 호출한 비지니스 메소드를 포함하는 비즈니스 객체 리턴
Object[] getArgs() : 클라이언트가 메소드를 호출할 때 넘겨준 인자 목록 리턴

사용하려면 어드바이스 메소드 매개변수로 JoinPoint를 선언하면 된다. => 스프링 컨테이너에서 JoinPoint 객체를 생성

실습 보기

---
## 어노테이션 기반 AOP

`<aop:aspectj-autoproxy/>`
를 스프링 설정 파일에 입력하면 알아서 처리해준다.

이후
`@Service` 어노테이션 설정으로 bean등록을 대신한다

``` java
@Pointcut("execution(* io.icednut.spring.exercise..*Impl.*(..))") // 포인트컷
public void allPointcut() {} // 참조 메서드
```

``` java
@Before("allPointcut()") // 어드바이스 설정
public void printLogging() {
    System.out.println("[공통 로그-Log4j 비지니스 로직 수행 전 동작");
}
```

###  애스펙트 설정 (포인트컷과 어드바이스 결합)
`@Aspect`가 설정된 애스펙트 객체에는 반드시 포인트컷과 어드바이스를 결합하는 설정이 있어야한다.


``` java
package io.icednut.spring.exercise.common;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.AfterThrowing;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Service;

import java.sql.SQLException;

@Service
@Aspect
public class Log4jAdvice {
    @Pointcut("execution(* io.icednut.spring.exercise..*Impl.*(..))")
    public void allPointcut() {}

    @Pointcut("execution(* io.icednut.spring.exercise..*Impl.get*(..))")
    public void getPointcut() {}

    @Before("allPointcut()")
    public void printLogging() {
        System.out.println("[공통 로그-Log4j 비지니스 로직 수행 전 동작");
    }

    @AfterThrowing(pointcut = "allPointcut()", throwing = "exceptObj")
    public void exceptionLogging(JoinPoint jp, Exception exceptObj) {
        String method = jp.getSignature().getName();
        System.out.println(method + "() 메소드 수행 중 예외 발생!");

        if (exceptObj instanceof SQLException) {
            System.out.println("SQL 수행 중 예외 발생");
        } else {
            System.out.println("기타 예외 발생");
        }
    }
}
```

—
Referencing

: 채규태 (2016), 스프링 퀵 스타트, 경기 부천: 루비페이퍼
