---
title: Spring JPA의 application.yml 설정하는 프로퍼티 항목

categories:
- spring

tags:
- spring
- jpa

toc: true
toc_sticky: true
toc_label: Contents
---

## Spring JPA의 application.yml 설정하는 프로퍼티 항목

```yaml
spring:  
  datasource:  
    url: <database_url>  
    username: <database_username>  
    password: <database_password>  
    driver-class-name: <database_driver_class_name>  
  jpa:  
    hibernate:  
      ddl-auto: <create/drop/create-drop/update/validate/none>  
    properties:  
      hibernate:  
        dialect: <database_dialect>  
        show_sql: <true/false>  
        format_sql: <true/false>  
        use_sql_comments: <true/false>  
        jdbc:  
          batch_size: <batch_size_value>  
          fetch_size: <fetch_size_value>  
          named_query_check_regions: <named_query_check_regions_value>  
        cache:  
          use_second_level_cache: <true/false>  
          use_query_cache: <true/false>  
          region:  
            factory_class: <cache_region_factory_class>  
```
-   `spring.datasource.url`: 데이터베이스 URL
-   `spring.datasource.username`: 데이터베이스 사용자 이름
-   `spring.datasource.password`: 데이터베이스 비밀번호
-   `spring.datasource.driver-class-name`: 데이터베이스 드라이버 클래스 이름
-   `spring.jpa.hibernate.ddl-auto`: Hibernate가 애플리케이션 시작 시에 데이터베이스 스키마를 생성/수정/검증하는 방법 설정 (create, create-drop, update, validate, none)
-   `spring.jpa.properties.hibernate.dialect`: Hibernate가 사용할 데이터베이스 방언(Dialect)
-   `spring.jpa.properties.hibernate.show_sql`: Hibernate가 실행한 SQL 쿼리를 로그에 출력할지 여부
-   `spring.jpa.properties.hibernate.format_sql`: Hibernate가 실행한 SQL 쿼리를 포맷팅하여 로그에 출력할지 여부
-   `spring.jpa.properties.hibernate.use_sql_comments`: Hibernate가 실행한 SQL 쿼리에 주석을 추가할지 여부
-   `spring.jpa.properties.jdbc.batch_size`: JDBC 배치 사이즈
-   `spring.jpa.properties.jdbc.fetch_size`: JDBC 페치 사이즈
-   `spring.jpa.properties.named_query_check_regions`: Named 쿼리 검증 여부
-   `spring.jpa.properties.cache.use_second_level_cache`: Hibernate 두 번째 레벨 캐시 사용 여부
-   `spring.jpa.properties.cache.use_query_cache`: Hibernate 쿼리 캐시 사용 여부
-   `spring.jpa.properties.cache.region.factory_class`: Hibernate 두 번째 레벨 캐시 구현체 팩토리 클래스