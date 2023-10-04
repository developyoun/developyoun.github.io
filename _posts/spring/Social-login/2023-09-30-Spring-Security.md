---
title: Spring Security (스프링 시큐리티)

categories:
- spring

tags:
- spring-security
- spring
- java

toc: true
toc_sticky: true
toc_label: Contents
---

## 스프링 시큐리티?
스프링 기반 어플리케이션의 인증과 권한을 담당하는 스프링 하위 프레임워크

## 인증 (Authentication), 인가 (Authorization)
- 인증: 해당 사용자가 본인이 맞는지를 확인하는 절차
- 인가: 인증된 사용자가 요청된 자원에 접근가능한가를 결정하는 절차

## 아키텍처 (Architecture)
![](https://i.imgur.com/ACZmWkJ.png)

1. **Http Request 수신**
    - 사용자가 로그인 정보와 함께 인증 요청
2. **유저 자격을 기반으로 인증 토큰 생성**
    - AuthenticationFilter가 요청을 가로채고, 이를 통해 UsernamePasswordAuthenticationToken의 인증용 객체 생성
3. **Filter를 통해 AuthenticationToken을 AuthenticationManager로 위임**
    - AuthenticationManager의 구현체인 ProviderManager에게 생성한 UsernamePasswordToken 객체를 전달
4. **AuthenticationProvider의 목록으로 인증을 시도**
    - AuthenticationManager는 등록 AuthenticationProvider들을 조회하여 인증을 요구
5. **UserDetailsService의 요구**
    - 실제 DB에서 사용자 인증정보를 가져오는 UserDeatilsService에 사용자 정보를 넘겨준다.
6. **UserDetails를 이용해 User 객체에 대한 정보 탐색**
    - 넘겨받은 사용자 정보를 통해 DB에서 찾은 사용자 정보인 UserDetails 객체를 만든다
7. **User 객체의 정보들을 UserDeatils가 UserDetailsService(LoginService)로 전달**
    - AuthenticationProvider들은 UserDetails를 넘겨받고 사용자 정보를 비교한다
8. **인증 객체 or AuthenticationException**
    - 인증이 완료되면 권한 등의 사용자 정보를 담은 Authentication 객체를 반환한다
9. **인증 끝**
    - 다시 최초의 AuthenticationFilter에 Authentication 객체가 반환
10. **SecurityContext에 인증 객체를 설정**
    - Authentication 객체를 Security Context에 저장

---

## Spring Security의 주요 모듈
### 1. SecurityContextHolder, SecurityContext, AUthentication

```java
public interface Authentication extends Principal, Serializable {
	// 현재 사용자의 권한 목록을 가져옴
	Collection<? extends GrantedAuthority> getAuthorities();
    
    // credential을 가져옴
	Object getCredentials();
	
	Object getDetails();

	// Principal 객체를 가져옴
	Object getPrincipal();

	// 인증 여부를 가져옴
	boolean isAuthenticated();

	// 인증 여부를 결정함
	void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
 
}
```

세 가지 클래스 모두, 스프링 시큐리티의 주요 컴포턴트이다.

유저의 아이디와 패스워드를 사용자 정보에 넣고 실제 가입된 사용자인지 체크한 후 인증에 성공하면, 사용자의 principal과 credential 정보를 Authentication 안에 담는다 →  
Authentication을 SecurityContext에 보관한다.  →  
SecurityContext를 SecurityContextHolder에 담아 보관한다.

즉, SecuritycontextHolder를 통해 SecurityContext에 접근하고, SecruityContext를 통해 Authentication에 접근할 수 있다.
```java
SecurityContextHolder.getContext().getAuthentication().getPrinciple(); // 사용자의 정보를 얻을 수 있다.
```


### 2. UsernamePasswordAuthenticationToken
Authentication 인터페이스의 구현체로서, AbstractAUthenticationtoken의 하위 클래스이다.  
사용자 ID가 Principal 역할을 하고, Password가 Credential의 역할을 한다.  
UsernamePasswordAuthenticationToken의 첫 번째 생성자는 인증 전의 객체를 생성하고, 두 번재는 인증이 완료된 객체를 생성한다.

```java
public abstract class AbstractAuthenticationToken implements Authentication, CredentialsContainer {}
 
public class UsernamePasswordAuthenticationToken extends AbstractAuthenticationToken {
 
	private static final long serialVersionUID = SpringSecurityCoreVersion.SERIAL_VERSION_UID;
 
	// 주로 사용자의 ID에 해당
	private final Object principal;
 
	// 주로 사용자의 PW에 해당
	private Object credentials;
 
	// 인증 완료 전의 객체 생성
	public UsernamePasswordAuthenticationToken(Object principal, Object credentials) {
		super(null);
		this.principal = principal;
		this.credentials = credentials;
		setAuthenticated(false);
	}
 
	// 인증 완료 후의 객체 생성
	public UsernamePasswordAuthenticationToken(Object principal, Object credentials,
			Collection<? extends GrantedAuthority> authorities) {
		super(authorities);
		this.principal = principal;
		this.credentials = credentials;
		super.setAuthenticated(true); // must use super, as we override
	}
}
```

### 3. AuthenticationManager
인증에 대한 부분은 AuthenticationManager를 통해서 처리된다.  
실질적으로는 AuthenticationManager에 등록된 AuthenticationProvider에 의해 처리된다.  
인증에 성공하면 두 번째 생성자를 이용해 객체를 생성하여 SecurityContext에 저장한다.

```java
public interface AuthenticationManager{
	Authentication authentication(Authentication authentication) throws AuthenticationException;
}
```

### 4. AuthenticationProvider
AuthenticationProvider는 실제 인증에 대한 부분을 처리하는 작업을 진행한다.  
인증 전에 Authentication 객체를 받아 인증이 완료된 객체를 반환하는 역할을 하고, 인터페이스를 구현해 Custom한 AuthenticationProvider를 작성하고 AuthenticationManager에 등록하면 된다.

```java
public interface AuthenticationProvider {
 
	Authentication authenticate(Authentication authentication) throws AuthenticationException;
 
	boolean supports(Class<?> authentication);
 
}
```

### 5. ProviderManager
AuthenticationManager를 구현한 ProviderManager는 AuthenticationProvider를 구성하는 목록을 갖는다.

```java
public class ProviderManager implements AuthenticationManager, MessageSourceAware, InitializingBean {
	
    public List<AuthenticationProvider> getProviders() {
		return this.providers;
	}
    
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
		Class<? extends Authentication> toTest = authentication.getClass();
		AuthenticationException lastException = null;
		AuthenticationException parentException = null;
		Authentication result = null;
		Authentication parentResult = null;
		int currentPosition = 0;
		int size = this.providers.size();
        
        // for문으로 모든 provider를 순회하여 처리하고 result가 나올때까지 반복한다.
		for (AuthenticationProvider provider : getProviders()) { ... }
	}
}
```

### 6. UserDetailsService
UserDetailsService는 UserDetails 객체를 반환하는 하나의 메서드만을 가지고 있는데, 일반적으로 이를 구현한 클래스에서 UserRepository를 주입받아 DB와 연결하여 처리한다.

```java
public interface UserDetailsService {
	UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

### 7. UserDetails
인증에 성공하여 생성된 UserDetails 객체는 Authentication객체를 구현한 UsernamePasswordAuthenticationToken을 생성하기 위해 사용된다.  
UserDetails를 implements하여 처리할 수 있다.

```java
public interface UserDetails extends Serializable {
 
	// 권한 목록
	Collection<? extends GrantedAuthority> getAuthorities();
 
	String getPassword();
 
	String getUsername();
 
	// 계정 만료 여부
	boolean isAccountNonExpired();
 
	// 계정 잠김 여부
	boolean isAccountNonLocked();
 
	// 비밀번호 만료 여부
	boolean isCredentialsNonExpired();
 
	// 사용자 활성화 여부
	boolean isEnabled();
 
}
```

### 8. SecurityContextHolder
보안 주체의 세부 정보를 포함하여 응용프로그램의 현재 보안 컨텍스트에 대한 세부 정보가 저장된다.

### 9. SecurityContext
Authentication을 보관하는 역할을 하며, SecurityContext를 통해 Authentication을 저장하거나 꺼낼 수 있다.

```java
SecurityContextHolder.getContext().setAuthentication(authentication);
SecurityContextHolder.getContext().getAuthentication(authentication);
```

### 10. GrantedAuthority
현재 사용자(principal)가 가지고 있는 권한을 의미하며, `ROLE_ADMIN`이나 `ROLE_USER`와 같이 `ROLE_*`의 형태로 사용한다.  
GrantedAuthority 객체는 UserDetailsService에 의해 불러올 수 있고, 특정 자원에 대한 권한이 있는지를 검사하여 접근 허용 여부를 결정한다.

---

## 회고
명성이 자자했던 스프링 시큐리티에 대해서 한번 정리를 해봤는데, 명불허전,,, 쉽지 않은 듯 하다.  
물론 한번에 와닿는 사람도 있겠지만, 아직 부족한 나한텐 쉽지 않은듯 하다.   
스프링 시큐리티를 구현해 볼 기회가 생긴하면 반드시 이 글을 참고해서 한번 더 공부 해보고 싶다.


>**ref**.
> [참조 문서1](https://dev-coco.tistory.com/174)
> [참조 문서2](https://velog.io/@hope0206/Spring-Security-%EA%B5%AC%EC%A1%B0-%ED%9D%90%EB%A6%84-%EA%B7%B8%EB%A6%AC%EA%B3%A0-%EC%97%AD%ED%95%A0-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0)