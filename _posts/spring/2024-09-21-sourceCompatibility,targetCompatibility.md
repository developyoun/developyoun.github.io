---
title: sourceCompatibility / targetCompatibility

categories:
- spring

tags:
- java
- gradle

toc: true
toc_sticky: true
toc_label: Contents
---

## sourceCompatibility

Java 버전과 일치하는 값으로, 컴파일에서 사용하는 JDK 버전

소스 코드에서 사용할 수 있는 Java 버전을 해당 버전 값으로 제한

## targetCompatibility

생성된 클래스 파일의 버전을 제어

프로그램에서 실행할 수 있는 가장 낮은 Java 버전을 의미

만약 아래와 같이 설정한다면,

```yaml
java {
	sourceCompatibility = 1.6
	targetCompatibility = 1.8
}
```

소스코드에서는 람다 표현식이나, Java 6에서 사용할 수 없는 기능은 포함될 수 없으며, Java 8 이상에서 실행되어야 하는 클래스 파일이 생성됨.

> Spring Boot 프로젝트 생성 시, sourceCompatibility는 자동으로 추가되나, targetCompatibility는 추가되지 않음.
> 

***참고자료:***

https://dlee0129.tistory.com/265

https://www.baeldung.com/gradle-sourcecompatiblity-vs-targetcompatibility
