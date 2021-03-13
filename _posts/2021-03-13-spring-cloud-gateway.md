---
layout: post
title: "Spring Cloud Gateway(SCG)"
tags: [SpringCloudGatway]
comments: true
date: 2021-03-13
---

# spring-cloud-gateway

## Spring Cloud Gateway란

- API Gateway 중의 하나이고, 모든 유입이 통하기 때문에 API라우팅 및 보안, 모니터링/메트릭 등의 기능을 간단한 방식으로 제공
- Spring Boot 2.x, Spring WebFlux 및 Project Reactor를 기반으로 동작
    - Asynchronous / Non-Blocking 방식의 Netty 기반 Runtime 으로 동작하기 때문에 WAR로 빌드된 경우에는 작동하지 않음

![No image](/assets/posts/20210313/BlockingIO.png)

![No image](/assets/posts/20210313/NonBlockingIO.png)

![No image](/assets/posts/20210313/Framework.png)

## 용어

1. Route

- 목적지 URI, 조건부(Predicate) 집합, Filter 집합으로 정의되며, 조건부에 일치해야 해당 경로로 라우팅 됨

2. Predicate

- 라우팅를 결정하기 위한 조건
- AND 조건으로 조합할 수 있음
- Path : 주로 쓰는 predicate로 이 옵션에 지정한 path로 들어오는 경우, 해당 라우터로 라우팅
- 많은 종류의 Predicate Factory가 있기 때문에 아래 문서 참고하여 필요한 Predicate 검색
- [https://cloud.spring.io/spring-cloud-gateway/reference/html/#gateway-request-predicates-factories](https://cloud.spring.io/spring-cloud-gateway/reference/html/#gateway-request-predicates-factories)
- [https://woooongs.tistory.com/55](https://woooongs.tistory.com/55)

3. Filter

- 요청 전/후의 Request, Response 를 수정할 수 있음. (필터는 해당 라우터에 한정)
- 특정 factory 로 구성된 Spring Framework GatewayFilter 인스턴스
- AddResponseHeader GatewayFilter Factory, AddRequestParameter GatewayFilter Factory 등등
- 많은 종류의 Filter Factory 가 있기 때문에 아래 문서 참고하여 필요한 Filter 검색
- [https://cloud.spring.io/spring-cloud-gateway/reference/html/#gatewayfilter-factories](https://cloud.spring.io/spring-cloud-gateway/reference/html/#gatewayfilter-factories)
- [https://woooongs.tistory.com/55](https://woooongs.tistory.com/55)

![No image](/assets/posts/20210313/SCGflow.png)

## 소스

- 라우팅하기 위한 소스를 application.yml 에 작성할 수도 있고, Code 레벨에 작성할 수도 있음
- 동일한 route를 application.yml 파일, Code 레벨 양쪽에 작성할 경우 Code 레벨을 우선시 함
    - Code 레벨에 작성 시 RouteLocatorBuilder 를 통해 route 생성
- URI에 포트를 정의하지 않으면, 기본 포트인 http(80), https(443) 으로 연결

```java
// kotlin
@Bean
fun customRouteLocator(builder: RouteLocatorBuilder): RouteLocator? {
    return builder.routes()
            .route("default") { r: PredicateSpec ->
                r.path("/**")
                        .uri("http://localhost:8082") // 8082 포트로 라우팅함
            }
            .build()
}

// application.yml
spring:
  cloud:
    gateway:
      routes:
        - id: default
          uri: http://localhost:8081  // 무시됨
          predicates:
            - Path=/**
```

### Short Cut 방식 vs Fully Expanded 방식

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: server1
          uri: http://localhost:8081
          predicates:
            - Path=/server1/**
          filters:
						# Short Cut 방식 
            # - AddRequestParameter=seq, 13579
						# - Cookie=mycookie,mycookievalue

						# Fully Expanded 방식. filter종류에 따라 args의 name이 달라짐
            - name: AddRequestParameter
              args:
                name: seq
                value: 13579
						- name: Cookie
		          args:
		            name: mycookie
		            regexp: mycookievalue
```

## Filter Pre/Post 적용 순서

```java
@Component
class TestGlobalFilter: GlobalFilter, Ordered {
		// order가 낮은 것이 우선
    override fun getOrder(): Int {
        return -1
    }

    override fun filter(exchange: ServerWebExchange?, chain: GatewayFilterChain?): Mono<Void> {
				// 라우팅 요청 전의 filter 수행
        println("TestGlobalFilter first PRE filter")

        return chain!!.filter(exchange)
                .then(Mono.fromRunnable {
										// Mono.fromRunnable을 통해 라우팅 요청 후의 filter를 처리함 
                    println("TestGlobalFilter first POST filter")
                })
    }
}

@Component
class TestGlobalFilter2: GlobalFilter, Ordered {
    override fun getOrder(): Int {
        return 0
    }

    override fun filter(exchange: ServerWebExchange?, chain: GatewayFilterChain?): Mono<Void> {
        println("TestGlobalFilter second PRE filter")

        return chain!!.filter(exchange)
                .then(Mono.fromRunnable {
                    println("TestGlobalFilter second POST filter")
                })
    }
}

// result
TestGlobalFilter first PRE filter
TestGlobalFilter second PRE filter
TestGlobalFilter second POST filter
TestGlobalFilter first POST filter
// => 요청 전의 filter는 order가 낮은 것부터 수행되고, 요청 후의 filter는 order의 역순으로 진행됨
```

## yaml 설정

```yaml
server:
  port: 8080

management:
  endpoints:
    web:
      exposure:
        include:
          - "gateway"
  endpoint:
    gateway:
      enabled: true # spring actuator 설정

spring:
  cloud:
    gateway:
      ## 전역 필터
      ## code 레벨에 작성한 라우팅에서는 먹히지 않음.. default-filters 를 모든 라우팅에 적용하려면 applicaion.yml 에 모두 설정해야할 듯
      default-filters:
        - AddRequestParameter=req, shin

      ## 전역 timeout 설정 -> timeout 지나면 504 리턴받음
      httpclient:
        connect-timeout: 1000
        response-timeout: 4s # java.time.Duration 타입으로 설정
      routes:
        - id: server1
          uri: http://localhost:8081
          predicates:
            - Path=/server1/**
          filters:
            - name: AddRequestParameter
              args:
                name: seq
                value: 13579
          ## 특정 route 에 대한 timeout 설정 -> timeout 지나면 504 리턴받음
          ## 전역 timeout 도 설정되어 있다면 route 에 설정된 timeout 우선
          metadata:
            connect-timeout: 1000
            response-timeout: 2000  # 전역 timeout 과는 다르게 ms 단위로 설정해야 함
```

timeout 발생시 

- 504 Gateway Timeout 응답
- PRE filter는 수행되지만 POST filter 수행되지 않음

![No image](/assets/posts/20210313/timeout.png)

## AccessLog

- Reactor Netty 액세스 로그를 활성화하려면 Spring Boot 속성이 아닌 Java System 속성을 변경해야함

```java
fun main(args: Array<String>) {
		// accessLogEnabled 속성 true로 변경 
    System.setProperty("reactor.netty.http.server.accessLogEnabled", "true")
    runApplication<DemoApplication>(*args)
}
```

![No image](/assets/posts/20210313/log.png)

## 참고

- [https://cloud.spring.io/spring-cloud-gateway/reference/html](https://cloud.spring.io/spring-cloud-gateway/reference/html)

