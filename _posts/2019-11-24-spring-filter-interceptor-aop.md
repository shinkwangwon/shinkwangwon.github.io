---
layout: post
title: "Filter, InterCeptor, AOP 차이점"
categories: spring
date: 2019-11-24
---

## Filter & Interceptor & Aop
![No image](/assets/posts/20191124/spring_filter.png)

## Filter
- 필터는 DispatcherServlet 바깥에서 실행된다. 즉, 스프링 컨텍스트 외부에 존재하게 된다.
- 주로 DispatcherServlet 안으로 들어오기전에 처리할 인코딩, 시큐어 설정등을 해놓는다.
- 필터는 스프링과 무관하게 지정된 자원에 대해 동작한다.

~~~
- init() - 필터 인스턴스 초기화
- doFilter() - 전/후 처리
- destroy() - 필터 인스턴스 종료
~~~


## Interceptor
- DispatcherServlet 안쪽에서 실행되며, 클라이언트의 요청으로 컨트롤러로 들어올 때 요청을 가로채서 어떤 작업을 수행한 후 컨트롤러로 다시 보낼 수 있고 서버에서 클라이언트로 응답할 때 응답을 가로채서 어떠한 작업을 수행한 후 클라이언트로 전송할 수 있다.
- 인터셉터는 DispatcherServlet 안쪽이므로 스프링 컨텍스트 내부에 존재하기 때문에 스프링 내의 모든 객체(bean)에 접근이 가능하다.

~~~
preHandler() - 컨트롤러 메서드가 실행되기 전
postHanler() - 컨트롤러 메서드 실행직 후 view페이지 렌더링 되기 전
afterCompletion() - view페이지가 렌더링 되고 난 후
~~~


## AOP (Aspect Oriented Programming)
- AOP란 중복된 기능을 공통기능으로 만들고, 개발자가 직접 원하는 위치에서 수행시킬 수 있는 프로그래밍
- AOP는 Proxy형태로 사용된다 -> 어떤 메소드를 호출하면 proxy객체를 호출하게 되고 proxy객체는 설정해둔 공통기능도 하고 호출한 메소드 수행도 함
- joinPoint 중에서 조건을 만족하는 pointCut을 찾아 advice를 weaving 한다.
- joinPoint : pointcut의 후보 리스트
- pointCut : 실제로 공통 기능을 수행시킬 위치
- advice : 공통 기능으로 사용할 구현체(즉, 실제 사용할 메소드)
- weaving : advice가 pointCut 위치에 삽입되는 과정
- Aspect : 구현하고자 하는 횡단 관심사의 기능을 의미(로깅, 트랜잭선 등). 1개 이상의 pointCut과 adivce의 조합으로 만들어 진다. (Advisor)
  * 즉 Aspect는 여러 객체에 공통으로 적용되야할 공통 관점 사항이라는 추상적인 개념이고 실제 구현한 기능을 advice라하며, 그것을 적용시키는 시점을 pointCut이라 한다.
