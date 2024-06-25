---
title: Failed to validate connection
categories:
- spring
tags:
- spring
- java
- database

toc: true
toc_sticky: true
toc_label: Contents
---

애플리케이션을 실행할 때마다, 아래와 같은 로그가 `warning` 으로 발생했습니다.

```
 09:54:39.244 [http-nio-8080-exec-1] WARN  com.zaxxer.hikari.pool.PoolBase - HikariPool-1 - Failed to validate connection org.mariadb.jdbc.MariaDbConnection@1f104f9 (Connection.setNetworkTimeout cannot be called on a closed connection). Possibly consider using a shorter maxLifetime value.
 09:54:39.246 [http-nio-8080-exec-1] WARN  com.zaxxer.hikari.pool.PoolBase - HikariPool-1 - Failed to validate connection org.mariadb.jdbc.MariaDbConnection@d140ab (Connection.setNetworkTimeout cannot be called on a closed connection). Possibly consider using a shorter maxLifetime value.
 09:54:39.248 [http-nio-8080-exec-1] WARN  com.zaxxer.hikari.pool.PoolBase - HikariPool-1 - Failed to validate connection org.mariadb.jdbc.MariaDbConnection@19e4884 (Connection.setNetworkTimeout cannot be called on a closed connection). Possibly consider using a shorter maxLifetime value.
 ...
```

결론부터 말하자면 로그에 나와있는데로

**기본 커넥션 풀인, `hikari-pool` 의 maxLifeTime을 짧게 하면 됩니다.**

Maria DB 커넥션의 `wait_timeout` 값은, 이 시간이 지나면 연결을 종료시킵니다.

```sql
show variables like 'wait_time%';
```

Spring 커넥션(hikari-pool)의 `max-lifetime` 값은, 이 시간이 지나면 새로운 커넥션을 만드려 시도합니다.

```yaml
spring:
  datasource:
    hikari:
			max-lifetime: 1800000 # 30분
```

즉, **DB가 연결을 끊기 전에 서버에서 커넥션을 끊어 새로운 커넥션을 시도하게 되는 것이 이상적입니다.**

hikari 측에서는 max-lifetime 값을 wait-timeout보다 2~3초 짧게 줄 것을 권고하고 있습니다.

예시)

- wait-timeout: 28_800
- max-lifetime: 25_800

> 참고:

https://gksdudrb922.tistory.com/228
>
