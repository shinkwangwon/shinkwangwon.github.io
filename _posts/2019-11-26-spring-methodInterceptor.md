---
layout: post
title: "Spring MethodInterCeptor & HandlerMethodArgumentResolver"
tags: [spring]
comments: true
date: 2019-11-26
---


## HandlerMethodArgumentResolver
- 스프링의 HandlerMethodArgumentResolver는 클라이언트의 요청이 Controller에 도달하기 전에 그 요청의 파라미터를 수정할 수 있도록 해준다.
- 컨트롤러에 들어오는 파라미터에 무언가 값을 추가해주어야 한다거나, 변경해줘야 할때 쓰면 좋다.
- HandlerMethodArgumentResolver 를 구현하려면 2가지의 메소드를 반드시 구현해야한다.  
- HandlerMethodArgumentResolver 구현한 클래스는 스프링 설정정보에 지정을 해주어야 함.  
- 아래 XML에 등록했으므로 TestHandlerMethodArguementResolver 클래스에는 따로 빈 스캐닝을 위한 어노테이션을 부여하지 않아도 됨.  

```java
public class TestHandlerMethodArguementResolver implements HandlerMethodArgumentResolver {
    // 리졸버가 사용가능한지 확인하는 메소드 : true를 리턴하면 resolveArgument 메소드 수행, false를 리턴하면 resolveArgument 메소드 수행 안함
    boolean supportsParameter(MethodParameter parameter) {
        // return true or false;
    }

    // 클라이언트의 요청 파라미터와 기타 정보를 받아서 실제 컨트롤러에서 쓰일 파라미터를 리턴 
    // 즉, 이 메소드에서 클라이언트의 요청파라미터를 가로채서 가공한다음에 컨트롤러에게 가공된 파라미터로 요청을하게 됨
    Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        // 컨트롤러에 전달하고 싶은 파라미터 가공 작업
        // return 컨트롤러에 전달할 파리미터;
    }
}

```

```xml
<mvc:annotation-driven>
    <mvc:argument-resolvers>
        <bean class="com.test.TestHandlerMethodArguementResolver">
    </mvc:argument-resolvers>
</mvc:annotation-driven>
```



## MethodInterCeptor
- 스프링의 MethodInterCeptor는 공통기능을 하는 메소드를 만들고, AOP를 이용하여 원하는 포인트컷을 지정하여 수행시킬 수 있다.
- MethodInterCeptor을 구현하려면 1개의 메소드를 반드시 구현해야 한다.
- Object invoke(MethodInvocation invocation) throws Throwable;
- @Component 어노테이션으로 스프링 빈으로 등록되도록 함  
```java
@Component
public class TestMethodInterceptor implements MethodInterceptor {

    @Override
	public Object invoke(MethodInvocation invocation) throws Throwable {
        // 여기서 대상 객체 메소드 수행 전에 할 작업을 수행
        // 작업~~~

        // invocation.proceed(); 메소드를 호출해줘야 대상 객체 메소드를 호출 한다.
        return invocation.proceed();
    }
}
```
