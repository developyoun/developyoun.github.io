---
title: OpenFeign-Client 헤더 추가하기

categories:
- spring

tags:
- http-client
- spring

toc: true
toc_sticky: true
toc_label: Contents
---

## 1. 헤더 1개 추가

```java
@FeignClient(name = "testClient", url = "http://localhost:8080")
public interface TestClient() {
	@GetMapping(value = "/test", headers = "key1=value1")
	Response getTest();
}
```

## 2. 헤더 2개 추가

```java
@FeignClient(name = "testClient", url = "http://localhost:8080")
public interface TestClient() {
	@GetMapping(value = "/test", headers = {"key1=value1", "key2=value2"})
	Response getTest();
}
```

## 3. 모든 Feign 요청에 헤더 추가

모든 요청에서 헤더를 추가하고 싶으면 `RequestInterceptor` 인터페이스 사용.

```java
// RequestInterceptor 구현
public class CustomRequestInterceptor implements RequestInterceptor {
	
	@Value("${token}")
	private final token;

	@Override
	public void apply(RequestTemplate template) {
		template.header("Authorization", token);
	}
}
```

```java
// FeignConfig 설정 (빈 등록)
@Configuration
public class FeignConfig {

	@Bean
	public CustomRequestInterceptor feignInterceptor() {
		return new CustomRequestInterceptor();
	}
}
```

```java
// Feign Client 설정
@FeignClient(name = "test", url = "http://localhost:8080", configuration = FeignConfig.class)
public interface Test() {
	
	@GetMapping(value = "/test")
	Response getTest();
}

```

> 참고: [https://rudaks.tistory.com/entry/openfeign에서-header에-값-추가하는-방법](https://rudaks.tistory.com/entry/openfeign%EC%97%90%EC%84%9C-header%EC%97%90-%EA%B0%92-%EC%B6%94%EA%B0%80%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95)
>
