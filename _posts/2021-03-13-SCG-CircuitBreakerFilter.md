---
layout: post
title: "Spring Cloud Gateway CircuitBreakerFilter"
tags: [SpringCloudGatway]
comments: true
date: 2021-03-13
---

# Circuit Breaker Filter

### Circuit Breaker Filter
- 서비스 장애시 쓰레드들이 요청을 기다리게 되면서 장애가 전파될 수 있는데, 이러한 장애 전파를 방지하기 위해 MSA에서 사용하는 Circuit Breaker 패턴을 사용한 Gateway Filter 입니다.

- Spring Cloud Gateway 에서는 Resilience4j 를 이용하여 Circuit Breaker 기능을 제공합니다. Netflix Hystrix 도 있지만 Hystrix는 더 이상 개발되지 않고 유지보수만 하겠다고 했기 때문에 Spring Cloud Gateway에서는 Resilience4j 를 추천하고 있습니다.

```yaml
implementation("org.springframework.cloud:spring-cloud-starter-circuitbreaker-reactor-resilience4j")
```

- ReactiveResilience4JCircuitBreakerFactory 를 통한 Customize Bean을 작성하여 Circuit Breaker를 만들 수 있습니다.

- 아래 예제는 timeout 1초를 초과하면 요청한 곳으로 전달하지 않고 Circuit Breaker Filter에 설정한 fallbackUri 로 요청을 하는 예제입니다. 현재 fallbackUri 에는 forward: 스킴만 지원하고 있습니다. 

- fallbackUri에 설정된 URI는 기본적으로 Spring Cloud Gateway 내부 컨트롤러의 경로를 의미합니다.

```java
@Bean
fun defaultCircuitBreaker(): Customizer<ReactiveResilience4JCircuitBreakerFactory> {
    return Customizer<ReactiveResilience4JCircuitBreakerFactory> { factory ->
        factory.configureDefault { id ->
            Resilience4JConfigBuilder(id)
                    .circuitBreakerConfig(CircuitBreakerConfig.ofDefaults())
                    .timeLimiterConfig(TimeLimiterConfig.custom().timeoutDuration(Duration.ofMillis(1000)).build())
                    .build()
        }
    }
}
```

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: server2
          uri: http://localhost:8082
          predicates:
            - Path=/server2/**
          filters:
            - name: CircuitBreaker
              args:
                name: defaultCircuitBreaker
                fallbackUri: forward:/fallback
```
