---
layout: post
title: "Spring Cloud Gateway RateLimiterFilter"
tags: [SpringCloudGatway]
comments: true
date: 2021-03-13
---


# Rate Limiter Filter

## RequestRateLimiter 란

- RequestRateLimiter GatewayFilter Factory는 초당 요청 속도를 제한할 수 있는 Factory입니다. RateLimiter 구현을 사용하여 사용자가 설정한 값을 초과할 경우 HTTP 429 - Too Many Requests 상태를 반환합니다.

## KeyResolver

- RequestRateLimiter Factory 생성 시, 요청을 제한할 때 사용할 키를 설정하는 KeyResolver를 지정해 주어야 합니다. 만약 KeyResolver가 키를 찾지 못한다면 디폴트 정책으로 모든 요청을 막도록 되어 있습니다. (HTTP 403 Forbidden)

- 키를 찾지 못한 결과 코드로 403이 아닌 다른 상태코드를 내려주고 싶을 때 spring.cloud.gateway.filter.request-rate-limiter.empty-key-status-code 속성을 통해 변경할 수 있습니다.

- 또한 KeyResolver가 키를 찾지 못했을 때 요청을 막지 않고 허용하도록 spring.cloud.gateway.filter.request-rate-limiter.deny-empty-key 속성을 false 로 변경할 수도 있습니다. 당연한 얘기지만 이런 경우에는 RateLimiter가 적용되지 않은 상태로 요청을 전달합니다.

```yaml
spring:
  ...

  cloud:
    gateway:
      filter:
        request-rate-limiter:
          # 키를 찾지 못한 경우 요청 거부
          deny-empty-key: true
          # 키를 찾지 못한 경우 403 Forbidden 대신 429 Too Many Request로 응답
          empty-key-status-code: 429
  ...
```

 

## Redis RateLimiter

- Spring Cloud Gateway 에서는 Redis를 이용한 RequestRateLimiter를 지원하고 있습니다. Redis RateLimiter는 Token Bucket Algorithm을 통해 RateLimiter를 구현합니다. 

- Redis를 사용하기 위해 org.springframework.boot:spring-boot-starter-data-redis-reactive 의존성 추가가 필요합니다.

### 속성

- redis-rate-limiter.replenishRate : 초당 허용할 요청 수. 토큰 버킷이 채워지는 비율을 의미

- redis-rate-limiter.burstCapacity : 1초에 수행할 수 있는 최대 요청 수. 즉 토큰 버킷이 보유할 수 있는 최대 토큰 수를 의미. 이 값을 0으로 설정하면 모든 요청 차단

- redis-rate-limiter.requestedTokens : 각 요청에 필요한 토큰 수 (default 1개)

- replenishRate와 burstCapacity를 같은 값으로 설정하면 일정한 비율로 요청을 제한합니다. burstCapacity를 더 높게 설정한 경우 일시적으로 요청 수를 늘릴 수 있습니다. (토큰 버킷에 토큰이 남아있다면)

- 쉽게 말해 replenishRate=3 & burstCapacity=5 로 설정한 경우, 초당 3개의 토큰이 버킷에 쌓이고 최대 5개까지 쌓일 수 있습니다. 이때 초당 요청이 3개 미만으로 들어오다가 갑자기 1초에 요청이 10개가 들어오는 경우, 토큰이 3개 이상 남아있기 때문에 일시적으로 최대 burstCapacity만큼 요청을 처리할 수 있습니다.

- RedisRateLimiter 의 응답값은 log level 을 DEBUG 로 변경하면 요청마다 로그를 확인할 수 있습니다.

- 아래 예제는 로컬환경에서 Redis를 띄워놓은 상태에서 진행했습니다. 

```yaml
docker pull redis
docker run -d -p 6379:6379 --name rate_limiter redis
```

```yaml
logging:
  level:
    root: DEBUG

spring:
  # Redis 연동을 위한 설정
  redis:
    host: localhost
    port: 6379

  cloud:
    gateway:
      filter:
        request-rate-limiter:
          # 키를 찾지 못한 경우 요청 거부
          deny-empty-key: true
          # 키를 찾지 못한 경우 403 Forbidden 대신 429 Too Many Request로 응답
          empty-key-status-code: 429

      routes:
        - id: server2
          uri: http://localhost:8082
          predicates:
            - Path=/server2/**
          filters:
            - name: RequestRateLimiter
              args:
                key-resolver: "#{@userKeyResolver}"
                redis-rate-limiter.replenishRate: 3
                redis-rate-limiter.burstCapacity: 5
```

```java
@Bean(name = ["userKeyResolver"])
fun userKeyResolver(): KeyResolver {
    return KeyResolver { exchange ->
        val userId = exchange.request.queryParams.getFirst("userId")

        if(userId == null) {
            Mono.just("paylater")
        } else {
            Mono.just(userId)
        }
    }
}
```

![No image](/assets/posts/20210313/RateLimiterKey.png)

![No image](/assets/posts/20210313/RateLimiterResult.png)

참고

- [https://cloud.spring.io/spring-cloud-gateway/reference/html/](https://cloud.spring.io/spring-cloud-gateway/reference/html/)

- [https://woooongs.tistory.com/56](https://woooongs.tistory.com/56)

- [https://www.mimul.com/blog/about-rate-limit-algorithm/#2-token-bucket](https://www.mimul.com/blog/about-rate-limit-algorithm/#2-token-bucket)
