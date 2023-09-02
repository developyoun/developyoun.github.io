---
title: Kotlin ConfigurationProperties 설정하기

categories:
- spring

tags:
- spring
- kotlin

toc: true
toc_sticky: true
toc_label: Contents
---

보통 스프링(부트)을 개발하면서, `application.yaml` 혹은 `application.properties` 파일로 설정값들을 저장하는 경우가 많다.
```yaml
spring:  
  datasource:  
    url: jdbc:mysql://localhost:3306/simple  
    username: root  
    password: password
###  생략
```

그래서 이러한 설정값을 런타임 시점에서 사용하는 경우가 있는데, 한 두개의 값이면 많이 사용하는것은 `@Value` 어노테이션을 사용하기도 한다.
```kotlin
class ConfigForSetup {
	@Value("\${spring.datasource.url}")  val url: String = ""
}
```

하지만 이러한 설정값들이 다수인 경우에는 모든 필드에 `@Value` 로 선언해서 쓰는건 매우 비효율적으로 보인다.
그래서 한번에 모든 프로퍼티 설정들을 가져와서 변수로 사용할 수 있게 해주는 방법을 찾아보았다.

바로, `@ConfigurationProperties` 어노테이션이다. 이를 적용하면 다량의 프로퍼티 설정들을 한번에 클래스 내부로 옮길 수 있으며, 이에 대한 유지보수성이 증가할 수 있다.
차근차근 시작해보자.

## 1. 디펜던시 설치
```yaml
annotationProcessor("org.springframework.boot:spring-boot-configuration-processor")   // ConfigurationProperties 어노테이션 사용
```
정확히 스프링부트 몇 버전인지는 모르겠지만, 일정 버전이후부터는 이 디펜던시가 있어야 오류가 나지 않는다고 한다. (2.2 였나..?)

## 2. 어플리케이션 클래스에 어떤 프로퍼티 클래스를 스캔할 것인지 정의하기
```kotlin
// MyAppApplication.kt

@SpringBootApplication  
@EnableConfigurationProperties(SimpleProperties::class)  
class MyAppApplication

fun main(args: Array<String>) {  
   runApplication<MyAppApplication>(*args)  
}
```
어플리케이션 클래스에 내가 저장할 프로퍼티 설정 클래스를 스캔할 수 있도록 `@EnableConfigurationProperties` 를 정의한다.
→ 참고로, `@EnableConfigurationProperties` 어노테이션 대신에, `@ConfigurationPropertiesScan` 을 사용할 수도 있는데, `@ConfigurationProperties` 가 붙은 모든 클래스를 스캔하는 방식이다. (이게 더 편한듯?)

## 3. (본적격인) 프로퍼티 클래스 정의
```kotlin
@ConstructorBinding  
@ConfigurationProperties(prefix = "spring.datasource")  
data class DbProperties(  
    val url: String,
    val username: String,
    val password: String
) {}
```
`@ConfigurationProperties` 로 가져올 프로퍼티 경로를 지정해준다.
여기서 중요한 건 `@ConstructorBinding` 어노테이션인데, 이는 **생성자 방식으로 변수를 만들어주기에 불변 객체를 만들 수 있게 합니다**
→ 즉, `var` 혹은 `lateinit var` 로 프로퍼티 변수를 생성하는 것이 아닌, `val` 로 변수를 생성하는 것이 가능할 수 있게 해주는 것. (매우 권장됨)

## 4. 프로퍼티 변수를 타 클래스에서 사용하기
```kotlin
@Configuration  
class ConfigForSetup(val dbProperties: DbProperties) {  
  
    @Bean  
    fun test() {  
        println(encryptProperties.url)
        println(encryptProperties.username)
        println(encryptProperties.password)  
    }  
}
```
이렇게 다수의 프로퍼티 값을 한번에 변수로 만들어 사용할 수 있다.

