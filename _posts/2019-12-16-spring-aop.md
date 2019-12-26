---
layout: post
title: "Spring CustomAnnotation And AOP"
tags: [spring]
comments: true
date: 2019-12-16
---

## Annotation
- 어노테이션은 자바 소스코드에 추가적인 정보를 제공하는 메타데이터 용도로 많이 쓰이며 컴파일 혹은 런타임에 해석된다.
- 자바에서 기본적으로 제공하는 대표적인 어노테이션에는 @Override, @Deprecated 등이 있다.

## Meta Annotation
- 메타 어노테이션은 어노테이션을 위한 메타데이터라고 볼 수 있다. 이 메타 어노테이션을 이용해 커스텀 어노테이션을 만들 수 있다.

## Meta Annotation 종류
1. @Retention - 어노테이션의 유효 범위를 결정
- RetentionPolicy.SOURCE : 컴파일 전까지만 어노테이션이 유효. 컴파일 이후에는 사라짐
- RetentionPolicy.CLASS : 컴파일러가 클래스를 참조할 때 까지 유효.
- RetentionPolicy.RUNTIME : 컴파일 이후에도 JVM에 의해 계속 참조 가능(리플렉션 사용가능)

2. @Target - 어노테이션을 적용시킬 수 있는 위치 결정
- ElementType.TYPE : class, interface, enum 선언시
- ElementType.FIELD : 멤버 변수 선언시
- ElementType.METHOD : 메서드 선언시
- ElementType.PARAMETER : 파라미터 선언시
- ElementType.CONSTRUCTOR : 생성자 선언시
- ElementType.LOCAL_VARIABLE : 지역변수 선언시
- ElementType.ANNOTATION_TYPE : 어노테이션 타입에만 적용
- ElementType.PACKAGE : 패키지 선언시
- ElementType.TYPE_PARAMETER : 제네릭 타입변수 (자바8 부터 적용가능)
- ElementType.TYPE_USE : 어떤 타입이던지 가능(extends, implements에 상속받을 타입에도 사용가능) (자바8 부터 적용 가능)

3. @Inherited
- 이 어노테이션이 붙어있는 인터페이스를 상속받을 경우 자식은 이 어노테이션을 상속 받는다.

4. @Documented
- 해당 어노테이션을 Javadoc에 포함시킨다.

## 커스텀 어노테이션 
- 메타 어노테이션을 이용해 커스텀 어노테이션을 생성할 수 있다.
- 커스텀 어노테이션과 리플렉션을 이용하면 여러 가지 기능을 만들 수 있는데 자주 사용하는 로그찍는 AOP를 구현해보자. 클라이언트의 요청을 받아 서버단에서 요청을 처리할 때 어떤 요청이 들어왔는지 로그를 찍는 AOP 기능이다.

## AOP 기본 개념
[Filter, InterCeptor, AOP 차이점](https://shinkwangwon.github.io/spring-filter-interceptor-aop/)

## AOP 어드바이스 수행 시점
- @Before : 타겟 메소드가 호출되기 전에 어드바이스 기능 수행
- @After : 타겟 메소드의 결과에 관계없이(성공, 예외 관계없이) 타겟 메소드가 완료되면 어드바이스 기능 수행
- @AfterReturning : 타겟 메소드가 성공적으로 결과값을 반환 후에 어드바이스 기능 수행
- @AfterThrowing : 타겟 메소드가 수행중 예외를 던지게 되면 어드바이스 기능 수행
- @Around : 어드바이스가 타겟메소드를 감싸서 타겟 메소드 호출전과 호출 후에 어드바이스 기능을 수행 - proceed()메서드 호출시점을 기준으로 메소드 전과 후로 구분함
> 타겟메소드란, 실제 로직을 담고 있는 메소드를 말한다. 즉, 타겟 메소드는 기존로직을 수행하는 메소드를 말하고 타겟 메소드를 기준으로 특정 시점에 공통 기능을 가진 어드바이스를 수행시키는 것이 AOP다.

```java
// 커스텀 어노테이션 생성
@Retention(RetentionPolicy.RUNTIME) // 런타임시에도 참조할 수 있다.
@Target(ElementType.TYPE)   // 클래스, 인터페이스단에 적용하겠다.
public @interface TestLogging { // @interface 형식으로 어노테이션을 생성한다.
}
```

```java
@Aspect // @Around와 같은 AspectJ 문법을 사용하기 위해 @Aspect 어노테이션을 부여한다.
@Component
public class TestLoggingAspectj {
    // TestLogging 어노테이션이 붙은 클래스 안의 모든 메소드를 호출할 때마다 호출 직전에 이 메소드가 수행된다.
    @Around("@within(com.common.annotation.TestLogging)")
    public Object testLoggingAspectJ(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("메소드 수행 전 한번 로그한번 남긴다.");
        
        // 이 위로 로직 작성하면 메소드 호출 전에 수행
        Object returnValue = joinPoint.proceed(); // 실제 메소드 수행
        // 이 아래로 로직 작성하면 메소드 호출 후에 수행

        return returnValue;
    }
}
```

```java
@Component
@TestLogging
public class TestLoggingClient {
    public void test1() {
        System.out.println("TestLoggingClient.test1() 호출");
    }
    public void test2() {
        System.out.println("TestLoggingClient.test2() 호출");
    }
}
```

```java
@Autowired
private TestLoggingClient testLoggingClient;

@Test
public void test() {
    // 실제호출
    testLoggingClient.test1();
    testLoggingClient.test2();

    // 결과
    // 메소드 수행 전 한번 로그한번 남긴다.
    // TestLoggingClient.test1() 호출
    // 메소드 수행 전 한번 로그한번 남긴다.
    // TestLoggingClient.test2() 호출


    // 참고로 아래처럼 클래스를 직접 선언해서 호출하면 AOP가 제대로 동작하지 않는다.
    // 이유는 AOP는 프록시 방식이기 때문에 Spring에게 오브젝트 주입 권한을 위임해서 사용한다(@Autowired)
    // TestLoggingClient tlc = new TestLoggingClient();
    // tlc.test1();
    // tlc.test2();
}
```



#### 참고
- <https://advenoh.tistory.com/21>