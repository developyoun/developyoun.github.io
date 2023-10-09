---
title: Thymeleaf 자동 리로드 (Live Reload)

categories:
- spring

tags:
- spring
- thymeleaf

toc: true
toc_sticky: true
toc_label: Contents
---



클라이언트 구성을 하다보면, 소스를 꽤 빈번히 변경하고 확인하는 작업을 하게된다.  
요즘 SPA들은 코드가 변경되면 자동으로 렌더링되는 구조로 되어있는데,  
타임리프는 기본 설정이라면, 서버를 재시작함으로서 정적 파일 템플릿을 생성한뒤 수정을 검수해야 한다.

근데 **프로퍼티 및 IDE 설정을 통해 서버 재시작 없이 브라우저 새로고침을 통해 수정을 반영할 수 있게 할 수 있다.**  
간단하다.

1. 타임리프의 프로퍼티 설정에서 cache 설정을 `false` 처리한다.
```yaml
spring:
	thymeleaf:
		cache: false
```

2. IDE (intelliJ) 실행 액션 설정하기
![](https://i.imgur.com/BLZL8fa.png)
on 'Update' action 및 on frame deactivation 설정을 **Update classes and resources** 로 설정

---

위와 같이 설정하면 이제 브라우저 새로고침을 통해 클라이언트 수정을 즉각적으로 확인할 수 있다.  
(다만, 정적파일로 제너레이팅 되는 딜레이가 있으니 약간 시간 텀을 두고 확인해야함)