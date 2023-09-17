---
title: Kotlin LocalDateTime

categories:
- kotlin

tags:
- kotlin
- localdatetime

toc: true
toc_sticky: true
toc_label: Contents
---

### 1. LocalDateTime now()

현재 로컬 컴퓨터의 날짜와 시간을 반환합니다. 타입은 LocalDateTime 입니다.

```
val nowDateTime = LocalDateTime.now();
// Result: 2023-04-24T21:00:56.043102
```

현재 결과는 로컬 컴퓨터 시간(Aisa/seoul)로 설정되어 있지만, 유동적으로 변경할 수 있습니다.바로, `ZoneId` 를 사용하는 방법이 있습니다.

```
val nowDateTime = LocalDateTime.now(ZoneId.of("America/Chicago"))
// Result: 2023-04-24T07:28:19.598154
```

아래 이미지는 지원하는 ZoneId 목록입니다. (코틀린 기준으로 Asia/seoul은 없군요..)\
![](https://i.imgur.com/ClotM3c.png)

### 2. LocalDateTime 생성

LocalDateTime에서 직접 시간을 생성하여 인스턴스를 생성할 수 있습니다.초, 밀리세컨드 단위는 생략할 수 있습니다.

```kotlin
val localDateTime = LocalDateTime.of(2023, 1, 1, 12, 20)
// Result: 2023-01-01T12:20
val localDateTime = LocalDateTime.of(2023, 1, 1, 12, 20, 30)  
// Result: 2023-01-01T12:20:30
val localDateTime = LocalDateTime.of(2023, 1, 1, 12, 20, 30, 223)
// 2023-01-01T12:20:30.000000223
```

또한, 월(month)는 Enum 타입을 이용해서 생성할 수도 있습니다.`Month` enum 타입을 사용하는데, 1월부터 12월까지 제공하고 있습니다.

```kotlin
val localDateTime = LocalDateTime.of(2023, Month.JANUARY, 1, 12, 20)
// Result: 2023-01-01T12:20
```

### 3. Formatter를 사용하여 원하는 출력사용

`DateTimeFormatter.ofPattern()` 을 사용하여 LocalDateTime을 원하는 시간 포맷으로 변경하여 사용할 수 있습니다.

* yyyy: 년도
* MM: 월
* dd: 일
* HH: 시
* mm: 분
* ss: 초

```kotlin
val localDateTime = LocalDateTime.now()
    
val formatter1 = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")
val localDateTime1 = localDateTime.format(formatter1)
// Result: 2023-04-24 21:46:45

val formatter2 = DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss")
val localDateTime2 = localDateTime.format(formatter2)
// Result: 2023/04/24 21:46:45

val formatter3 = DateTimeFormatter.ofPattern("yyyy MM dd HH:mm")
val localDateTime3 = localDateTime.format(formatter3)
// Result: 2023 04 24 21:46
```

참고로, 이렇게 포매터로 인해 변환된 날짜 및 시간들은 “문자열" 타입으로 취급되기 때문에, 이후 처리는 문자열 처리로 작업해야 합니다.\


### 4. LocalDateTime의 각각의 값 조회

```kotlin
val localDateTime = LocalDateTime.now()
println(localDateTime)
// Result: 2023-04-24T21:53:00.444246
println(localDateTime.year)
// Result: 2023
println(localDateTime.month)
// Result: APRIL
println(localDateTime.monthValue)
// Result: 4
println(localDateTime.dayOfMonth)
// Result: 24
println(localDateTime.hour)
// Result: 21
println(localDateTime.minute)
// Result: 53
println(localDateTime.second)
// Result: 0
println(localDateTime.nano)
// Result: 444246
```

### 5. LocalDateTime 연산

LocalDateTime 타입은 Plus / Minus 연산을 할 수 있다.

```kotlin
// Plus
println(localDateTime.plusDays(2))
// Result: 2023-04-26T22:00:08.241569
println(localDateTime.plusMonths(10))
// Result: 2024-02-24T22:00:08.241569

// Minus
println(localDateTime.minusHours(6))
// Result: 2023-04-24T16:00:08.241569
println(localDateTime.minusWeeks(2))
// Result: 2023-04-10T22:00:08.241569
```

### 6. LocalDateTime 비교

LocalDateTime은 일반 연산 말고도 비교 연산도 가능하다.

* CompareTo
* isAfter
* isBefore
* isEqual

```kotlin
val localDateTime1 = LocalDateTime.of(2023, 3, 24, 0, 0, 0)
val localDateTime2 = LocalDateTime.of(2023, 4, 24, 0, 0, 0)

println(localDateTime2.isAfter(localDateTime1))  // true
println(localDateTime2.isBefore(localDateTime1)) // false
println(localDateTime2.isEqual(localDateTime2))  // true

println(localDateTime1.compareTo(localDateTime2)) // -1
println(localDateTime2.compareTo(localDateTime1)) // 1
println(localDateTime1.compareTo(localDateTime1)) // 0
```

### 7. LocalDateTime 변환

LocalDateTime을 LocalDate 혹은 LocalTime으로 변환할 수 있다.

```kotlin
val localDateTime = LocalDateTime.now()

println(localDateTime)
// Result: 2023-04-24T22:12:06.614821
println(localDateTime.toLocalDate())
// Result: 2023-04-24
println(localDateTime.toLocalTime())
// Result: 22:12:06.614821
```

이렇게 보면, LocalDateTime으로 생성해서 다른 시간타입으로 변환해 사용하는 것이 좋을 듯 합니다.



> 레퍼런스\
> [https://covenant.tistory.com/255](https://covenant.tistory.com/255)