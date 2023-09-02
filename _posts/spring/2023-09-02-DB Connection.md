---
title: DB Connection

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

## DB Connection?
DB connection이란, 어플리케이션과 데이터베이스의 연결을 의미한다.
-   DB connection은 Database Driver와 Database 연결 정보를 담은 URL이 필요하다.
-   Java의 DB connection은 JDBC를 주로 이용한다.

> **JDBC ?**  
> JDBC는 Java Database Connectivity의 약자로서, 자바에서 데이터베이스로 접근하기 위한 API이다.  
> JDBC를 이용하면 자바 어플리케이션에서 데이터베이스와 통신하여 데이터를 읽고 쓸 수 있다.  
> 일반적으로 JDBC 드라이버라는 클래스를 사용하여 데이터베이스와 연결하는데, DBMS에 맞게 JDBC 드라이버를 다운로드하고 설치해야 한다.

### JDBC를 이용한 어플리케이션의 구조
![](https://i.imgur.com/YLFOj3B.png)


### JDBC의 실행 순서
![](https://i.imgur.com/BNyLWVE.png)

1.  **JDBC 드라이버 로딩**
	-   자바에서 데이터베이스와 연결하기 위해 JDBC 드라이버를 로딩하는 과정
	-   일반적으로 JDBC 드라이버는 `Class.forName()` 메서드를 사용하여 로딩한다.
2.  **DB Connection 생성 (데이터베이스 연결)**
	-   JDBC 드라이버를 로딩한 후, `DriverManager.getConnection()` 메서드를  호출해 데이터베이스와 연결을 시도한다.
	-   연결 문자열(URL)을 인자로 전달하여 데이터베이스 정보를 지정한다.
3.  **Statement 생성**
	-   Connection 객체에서 `createStatement()` 메서드를 호출하여 Statement 객체를 생성한다.
	-   Statement 객체는 SQL문을 실행하는 데 사용한다.
4.  **SQL 실행**
	-   Statement 객체에서 `executeQuery()` 메서드나, `executeUpdate()` 메서드를 호출하여 SQL문을 실행한다.
	-   `executeQuery()` 메서드는 SELECT문을 실행한다
	-   `executeUpdate()` 메서드는 INSERT, UPDATE, DELETE 문을 실행하고 영향받은 행의 수를 반환한다.
5.  **결과 처리 (결과 받기)**
	-   `executeQuery()` 메서드로 반환된 결과 집합은 ResultSet 객체로 처리된다.
	-   ResultSet 객체에서 `next()` 메서드를 호출하여 다음행으로 이동하고, `getXXX()` 메서드를 이용해 각 컬럼의 값을 가져올 수 있다.
6.  **연결 해제**
	-   SQL 문 실행이 끝나면 Statement, ResultSet, Connection 객체 등의 자원을 명시적으로 해제한다.
	-   이때, `close()` 메서드를 사용한다.

위와 같은 과정을 거쳐 JDBC를 사용하여 데이터베이스와 통신할 수 있는데, 이러한 작업은 매번 반복적으로 수행해야 하기에, 효율적인 방법으로 JDBC를 사용하기 위해 **DB Connection Pool** 과 같은 방법을 사용해야 한다.

> 레퍼런스  
> [https://steady-coding.tistory.com/564](https://steady-coding.tistory.com/564)