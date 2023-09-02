---
title: Kotlin Springboot에서 Datasource 정보 암호화하기

categories:
- spring

tags:
- spring
- kotlin

toc: true
toc_sticky: true
toc_label: Contents
---

보통 DB connection을 하기위한 정보를 `application.properties` 혹은 `application.yaml` 파일에 저장해서 해당 설정으로 커넥션을 생성한다.
```yaml
spring:  
  datasource:  
    url: jdbc:mysql://localhost:3306/test
    username: username
    password: password
    driver-class-name: com.mysql.cj.jdbc.Driver
```
근데, 또 이런 정보들을 형상관리하기 위해 git을 이용하는데 해당 정보가 그대로 올라간다는 점이다.
그럴 경우, 2가지 문제가 있을 수 있는데 
1. github에서는 해당 파일에 대한 보안성이 떨어진다고 분석하면서 경고를 낸다. 커밋 또한 제한됨
2. public 레포지토리 일 경우, DB 커넥션 정보가 그대로 유출된다. (오남용 가능성)

그러면 어떻게 해야할까? 서칭을 통해서 알아본 방법은 **Jasypt** 를 이용하는 방법이다.
**Jasypt** 는 Java Simplified Encryption의 약자로서, 자바 영역에서 간단하게 암호화를 제공해준다.
해당 github: https://github.com/ulisesbocchio/jasypt-spring-boot

## 1. 먼저 Jasypyt  디펜던시 적용
```yaml
implementation("com.github.ulisesbocchio:jasypt-spring-boot-starter:3.0.5")
```
현재 기준으로 3.0.5 버전이 latest version이다.

## 2. application 설정에 암호화 방식을 정의한다.
```yaml
jasypt:  
  encryptor:  
    bean: jasyptStringEncryptor   # java 빈 이름  
    algorithm: PBEWithMD5AndDES   # 사용 알고리즘  
    pool-size: 2    # 암호화 요청을 담을 pool 크기 설정 (2 권장)  
    string-output-type: base64    # 암호화 이후 받을 타입  
    key-obtention-iterations: 1000    # 암호화를 수행할 반복할 해시 횟수  
    password: password    # 암호화 키 (노출 x)
```
해당 설정값들에 대한 내용은 공식 깃헙이나 위 설정의 주석을 참고하자.

## 3. 프로퍼티 변수화
```kotlin
// EncryptProperties.kt

@ConstructorBinding  
@ConfigurationProperties(prefix = "jasypt.encryptor")  
data class EncryptProperties(  
    val bean: String,  
    val algorithm: String,  
    val poolSize: Int,  
    val stringOutputType: String,  
    val keyObtentionIterations: Int,  
    val password: String  
)
```
보통 application에 설정된 값들은 `@Value` 로 가져오는 경우가 많은데, 
설정값이 많거나 Relaxed Binding을 지원이 필요한 경우 `@ConfigurationProperties` 를 사용할 수 있다.
`@ConfigurationProperties` 를 사용하는 방법은 [여기](obsidian://open?vault=obsidian&file=Spring%2FKotlin%20ConfigurationProperties%20%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0)에 좀 더 자세하게 정리했다.

## 4.  빈에 등록할 암호화 설정
```kotlin
// Encryptor.kt

@Configuration  
@EnableEncryptableProperties  
class Encryptor(val encryptProperties: EncryptProperties) {  
  
    @Bean("jasyptStringEncryptor")  
    fun jasyptEncryptor(): PooledPBEStringEncryptor {  
        val pooledPBEStringEncryptor = PooledPBEStringEncryptor()  
        return pooledPBEStringEncryptor.apply {  
            setAlgorithm(encryptProperties.algorithm)  
            setPoolSize(encryptProperties.poolSize)  
            setStringOutputType(encryptProperties.stringOutputType)  
            setKeyObtentionIterations(encryptProperties.keyObtentionIterations)  
            setPassword(encryptProperties.password)  
        }  
    }  
  
}
```
일반적인 setter를 적용하면된다.

## 5. 암호화하여 application 설정에 적용
```kotlin
// applicationTest.kt

@Autowired  
@Qualifier(value = "jasyptStringEncryptor")  
lateinit var encryptor: PooledPBEStringEncryptor;  
  
@Test  
fun getEncrypt() {  
   println(encryptor.encrypt("jdbc:mysql://localhost:3306/simple"))
}
```
![](https://i.imgur.com/S9BsDWT.png)
위와 같은 결과를 얻을 수 있는데, 이를 application.yaml에 적용하면 된다.
적용할 때는, `ENC(${암호화된 데이터})` 로 넣어주어야 한다.
```yaml
spring:  
  datasource:  
    url: ENC(VvwNibGd5OiLOsIh8BOZEfrh9y5u1uwoCB9qWL48txOmhQmsHY3TKcCHGzSWgfcA)  
    username: ...
    password: ...
    driver-class-name: com.mysql.cj.jdbc.Driver
```

```ad-warning
application.yaml에 있는 Jasypt.encryptor.password는 노출되지 말아야 한다. (노출되면 암호화를 적용하는게 의미가 없어짐)
그렇기에 password 정보는 별도의 파일로 관리하여 .gitignore로 관리하는 것이 좋을 듯 하다.
```