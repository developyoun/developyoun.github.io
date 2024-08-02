---
title: Can't configure mvcMatchers after anyRequest 이슈 처리
categories:
- spring
tags:
- spring
- java
- security

toc: true
toc_sticky: true
toc_label: Contents
---

Spring Boot 3.xx 버전에서 스프링 시큐리티를 사용하면서 Request로 들어오는 요청에 권한을 처리할 때, 조건을 걸어 동적으로 패턴을 처리하고자 했다.

```java
.authorizeHttpRequests(authorizeRequest -> {
    authorizeRequest.requestMatchers("/heartbeat").permitAll()
            .anyRequest().authenticated();
    if (permittedPatterns.length > 0) {
        authorizeRequest.requestMatchers(permittedPatterns).permitAll();
    }
})
```

위와 같이 마지막에 조건을 걸면, `Can't configure mvcMatchers after anyRequest` 키워드의 예외가 발생했는데

이 문제의 원인은, **`.anyRequest` 이후에는 추가적인 `.requestMatchers` 를 사용할 수 없기 때문이다.**

```java
.authorizeHttpRequests(authorizeRequest -> {
    authorizeRequest.requestMatchers("/heartbeat").permitAll()

    if (permittedPatterns.length > 0) {
        authorizeRequest.requestMatchers(permittedPatterns).permitAll();
    }

    authorizeRequest.anyRequest().authenticated();
})
```

이렇게 변경하여 사용하면 정상적으로 요청을 처리할 수 있다.
