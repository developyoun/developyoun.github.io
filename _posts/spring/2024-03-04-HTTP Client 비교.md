---
title: HTTP Client 비교

categories:
- spring

tags:
- spring
- java
- http

toc: true
toc_sticky: true
toc_label: Contents
---

# HTTP Client 비교

### **개요**

신규 프로젝트와 채팅 서버가 통신하기 위해서 API를 주고 받는 과정이 있다. 기존 채팅 시스템에서는 `Retrofit2` 을 사용했으나, 런칭한지도 꽤 되었고 그 사이에 다른 라이브러리나 오픈소스가 생겼을 것 같아 조사해보고 검토한 뒤에 적용해 볼까하는 생각으로 조사하였음.

### **종류**

### 1. RestTemplate

`RestTemplate` 은 Spring Framework에서 제공하며, 동기식 HTTP 통신을 위한 클라이언트이다.

- 스프링 3.0에서 추가되었다.
- 주로 Spring MVC 기반의 애플리케이션에서 사용
- 다양한 HTTP 메서드(GET, POST, PUT, DELETE)를 지원
- HttpClient를 추상화하여 제공한다.
- 동기식으로 동작하기 때문에, 응답을 받을때까지 대기하며 스레드를 차단함. (성능 이슈 가능성)
- Spring Framework 5.0부터 RestTemplate은 **유지관리모드**(제거되지 않는 상태)에 있다
- **대안 기술로 WebClient를 고려 해보라고 한다.**

### 2. WebClient

Spring Framework 5.0부터 도입된 Http 클라이언트 라이브러리로, 비동기/논블로킹 방식으로 API 호출이 가능하다. (동기식도 가능)

- 비동기 및 리액티브 스트림 처리를 지원
- 대량의 요청을 효과적으로 처리할 수 있다.
- RestTemplate의 대안으로 거론됨
- 웹플럭스(webflux)를 추가해야만 사용 가능함 (= webflux 모듈 의존성이 추가된다)

### 3. OkHttp & Retrofit

OkHttp는 Squre사에서 개발한, Android 및 Java 전용 type-safe Http 클라이언트 라이브러리이다.

Retrofit은 OkHttp를 조금 더 추상화하여 직관적이고 편리하게 사용할 수 있다. (즉, retrofit 또한 내부적으로 okhttp를 사용)

- HTTP API를 자바 인터페이스 변환해주는 선언적 웹 서비스 클라이언트이다.
- 간단한 API 인터페이스를 통해 안정적인 네트워크 요청을 처리할 수 있다.
- 성능이 우수하고 사용이 간편하다.
- Spring Framework의 기본 지원 범위가 아니고, 별도로 라이브러리를 추가해야 한다. (호환성 검토 필요)

### 4. (Spring Cloud) OpenFeign

Feign은 Netflix에서 개발한 HTTP 클라이언트 binder이다.

HTTP 클라이언트를 쉽게 작성할 수 있는 *선언적 HTTP 클라이언트*이다.

> 현재 Feign은 Spring Cloud echo 시스템에 통합되었고 Netflix에서 사용을 중단하면서 OpenFeign이라는 오픈 소스 커뮤니티의 프로젝트로 이전되었다.
>
- annotation을 통해 다양한 옵션을 간단히 처리할 수 있다.
- 구현 코드 없이 로직을 명확히 이해 가능하도록 유도 (인터페이스 작성을 통한 구현체 자동 생성)
- 다른 Spring Cloud 기술들과 통합이 쉽다.
- 기본적으로 동기로 동작하므로, 선언적 구문을 통한 비동기를 활용하려면 ***Feign Reactive*** 사용