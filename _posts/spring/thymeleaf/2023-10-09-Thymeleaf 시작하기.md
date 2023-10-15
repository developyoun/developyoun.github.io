---
title: Thymeleaf 시작하기

categories:
- spring

tags:
- spring
- thymeleaf

toc: true
toc_sticky: true
toc_label: Contents
---



타임리프(Thymeleaf)란 웹 및 독립형 환경에서 사용하는 Java 진형의 서버 템플릿 엔진이다.  

> 템플릿 엔진?
> 템플릿 양식과 특정 데이터 모델에 따른 입력 자료를 합성하여 결과 문서를 출력하는 소프트웨어 컴포넌트를 통칭한다.

API 작업을 위해서 back단쪽 작업을 하는데, 간단한 클라이언트 환경을 구성하고 테스트하기 위해 타임리프를 적용하기로 함.

## 타임리프 적용

1. gradle 의존성 라이브러리 추가
```java
implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
```

2. application.properties 혹은 application.yaml 설정
*application.yaml*
```java
spring:  
  thymeleaf:  
    prefix: classpath:/templates/  
    suffix: .html
```
- 템플릿 경로를 설정하는 것인데, 타임리프를 의존성 처리하면 자동으로 생성된다고 함. (나 같은 경우는 수동으로 추가해주었음)
- ex) /templates/home.html

3. templates 디렉토리에 html을 생성하기  



![](https://i.imgur.com/K8DQF33.png)

```html
<!DOCTYPE html>  
<html lang="en" xmlns:th="http://www.thymeleaf.org">  
<head>  
  <meta charset="UTF-8">  
  <title>Title</title>  
</head>  
<body>  
  Hello World  
</body>  
</html>Í
```
- html 태그에 `xmlns:th="http://www.thymeleaf.org"` 를 추가해주어야 타임리프 문법이 가능하다.

4. controller 생성하기
```java
@Controller  
public class IndexController {  
  
    @GetMapping("home")  
    public String home() {  
        return "home";  
    }
}
```
- Controller에서 해당 템플릿을 호출할 수 있게, 설정한다.
- return으로 해당 템플릿 지정시 `.html` 을 붙히지 않는걸 권고한다.

---

## thymeleaf 고급 설정하기
application.yaml을 설정할 때, 간단하게 suffix와 prefix를 지정했지만, 사실 타임리프의 프로퍼티 설정은 다양한 것들이 존재한다.

1. 타임리프 템플릿 캐시 설정
```yaml
spring:
	thymeleaf:
		cache: true # false
```
- 템플릿 캐싱 활성화 또는 비활성화 설정

2. 타임리프 템플릿 모드 설정
```yaml
spring:
	thymeleaf:
		mode: HTML5 # LEGACYHTML5, XHTML, XML
```
- 템플릿 모드 구성 설정

3. 타임리프 템플릿 파일 위치 설정
```yaml
spring:
	thymeleaf:
		prefix: classpath:/templates/
```
- 템플릿 파일이 위치하는 디렉토리 설정

4. 타임리프 템플릿 파일 확장자 설정
```yaml
spring:
	thymeleaf:
		suffix: .html
```
- 템플릿 파일의 확장자 설정

5. 타임리프의 문자셋 설정
```yaml
spring:
	thymeleaf:
		encoding: UTF-8
```
- 템플릿의 문자셋 설정

6. 타임리프의 템플릿 리졸버 우선순위 설정
```yaml
spring:
	thymeleaf:
		template-resolver-order: 1
```
- 템플릿 리졸버의 우선순위 설정

7. 타임리프 다국어 및 국제화 설정
```yaml
spring:
	thymeleaf:
		enabled: true # false
```
- 다국어 및 국제화 지원 활성화 또는 비활성화

8. 타임리프의 스프링 시큐리티 설정
```yaml
spring:
	thymeleaf:
		servlet:
			content-accessible: true
```
- 스프링 시큐리티와 함께 사용 시 리소스 엑세스 설정

9. 타임리프의 템플릿 캐시 TTL(Time To Live) 설정
```yaml
spring:
	thymeleaf:
		cache-ttl-ms: 360000 # ms
```
- 템플릿 캐시의 TTL 설정



10. 타임리프의 사용자 정의 프로퍼티 설정
```yaml
spring:
	thymeleaf:
		properties:
			prop1: value1
			prop2: value2
```



> 뷰 리졸버 (View Resolver) ?  
> 스프링 백엔드에서 데이터를 처리하거나 가지고 왔다면, 이 데이터를 View의 영역으로 전달을 해야 한다. 이때 View를 어떤 것을 사용할지 자유롭게 설정을 할 수 있는데 이 설정 역할을 하는 것이 View Resolver라고 생각하면 된다.  
> ref: https://needneo.tistory.com/204