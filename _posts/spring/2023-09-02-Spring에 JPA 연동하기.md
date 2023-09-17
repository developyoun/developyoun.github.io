---
title: Springboot에 JPA 연동하기

categories:
- spring

tags:
- spring
- jpa

toc: true
toc_sticky: true
toc_label: Contents
---

## 1. 프로젝트 의존성 추가

```groovy
// build.gradle.kts  
dependencies {  
	implementation("org.mariadb.jdbc:mariadb-java-client")                      // MariaDB 사용시  
    implementation('mysql:mysql-connector-java')            // MySQL 사용시 (✔️)  
    implementation('org.springframework.boot:spring-boot-starter-data-jpa')  
}  

```
---
## 2. application.yml 설정
```yml
spring:  
  // ... 생략  
  datasource:  
    # MySQL 설정  
    driver-class-name: com.mysql.cj.jdbc.Driver  
    # DB Source URL  
    url: jdbc:mysql://localhost:3306/testdb?useSSL=false&useUnicode=true&serverTimezone=Asia/Seoul  
    # DB username  
    username: root  
    # DB password  
    password: 1234  
    jpa:  
      # DDL(create, alter, drop) 정의시 DB의 고유 기능을 사용할 수 있다.  
      hibernate.ddl-auto: update  
      # JPA의 구현체인 Hibernate가 동작하면서 발생한 SQL의 가독성을 높여준다.  
      properties.hibernate.format_sql: true  
```

> **spring.jpa.properties.hibernate.format_sql ?**  
> 하이버네이트가 SQL 쿼리를 로깅할 때, 해당 쿼리를 보기 좋게 포맷팅해주는 옵션(true/false). 이 옵션은 SQL 쿼리를 보기 좋게 출력하여 해당 쿼리의 문제점을 파악하기 쉽게 해준다.  
> **spring.jpa.properties.hibernate.show_sql ?** 하이버네이트가 실행하는 SQL 쿼리를 콘솔에 출력할지 여부를 지정하는 옵션(true/false).  
> 이 옵션을 사용하면 하이버네이트가 실행하는 SQL 쿼리를 실시간으로 살펴볼 수 있어 문제가 발생했을 때 해당 쿼리를 분석하고 디버깅하는 데 좋지만, 실제 운영 환경에서는 보안상의 이유로 이 옵션을 사용해서는 안된다.  
> **spring.jpa.properties.hibernate.use_sql_comments ?** 하이버네이트가 생성하는 SQL 쿼리에 주석을 추가할지 여부를 지정하는 옵션(true/false)  
> 이 값이 활성화된 경우, 하이버네이트는 생성하는 SQL 쿼리에 엔티티, 컬럼 등과 같은 객체와 매핑된 정보를 주석으로 추가한다. 이를 통해 생성된 SQL 쿼리를 읽는 것이 쉬워지고, 유지 보수나 디버깅이 용이해진다.  
>   
> **spring.datasource.hibernate.ddl-auto ?**  
> 하이버네이트가 어플리케이션 시작시에, 데이터베이스 스키마를 생성, 수정, 검증하는 방법을 지정하는 옵션이다.

`Spring.jpa.hibernate.ddl-auto` 속성은 아래와 같은 값들이 있다.
-   `none` : Hibernate가 데이터베이스 스키마를 수정하지 않음 (운영용)
-   `create` : 어플리케이션 시작시에 데이터베이스 스키마를 생성함 (개발용)
-   `create-drop` : 어플리케이션 시작 시에 데이터베이스 스키마를 생성하고, 어플리케이션 종료시에 데이터베이스 스키마를 삭제함 (개발용)
-   `update` : 어플리케이션 시작 시에 데이터베이스 스키마를 검증하고, 검증한 내용에 따라 수정한다. (어플리케이션 업그레이드, DB 마이그레이션용)
-   `validate` : 어플리케이션 시작 시에 데이터베이스 스키마를 검증한다. (DB 스키마 검증용)

none을 제외한 모든 옵션은 데이터베이스 스키마를 생성 또는 수정할 때 주의해서 사용해야 한다.

---
## 3. 제대로 연결 했는지 확인하기

![](https://i.imgur.com/yL2nk8w.png)

제대로 연결됨을 확인할 수 있다.
> 참고로 WARN⁠ 으로 발생한 로그가 하나 있는데, 이는 **spring.jpa.open-in-view** 옵션이다.  
> 이는, Spring Data JPA에서  기본적으로 활성화된 옵션으로, 하나의 HTTP 요청이 처리될 때 영속성 컨텍스트를 자동으로 유지해주는 기능을 수행한다.  
> 이를 통해 지연 로딩(Lazy Loading)등의 문제를 해결할 수 있으나, 영속성 컨텍스트를 유지하면서 불필요한 데이터베이스 쿼리가 발생할 수도 있다.