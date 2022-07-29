---
title: "[Spring] Spring Security - 원리"
last_modified_at: 2022-07-27T23:50:00+09:00
categories:
    - Back End
    - Spring
    - Spring Security
tags:
    - Back End
    - Spring
    - Spring Security
toc: false
---

Spring : Spring Security 원리를 이해한 대로 정리해보자.
{: .notice--info}

# FilterChainProxy

- `getFilters()` : Filter 리스트 조회

<br>

# AbstractAuthenticationProcessingFilter

- `doFilter()` : 필터의 로직을 수행한다.
- `attemptAuthentication()`
  - `doFilter()` 내에서 수행 되며 추상 메소드로 되어있다. 
  - UsernamePasswordAuthenticationFilter에 구현되어 있어 이를 호출한다.

<br>

# UsernamePasswordAuthenticationFilter

- Security 의 Default 인증 필터
- Session/Cookie 방식
- AbstractAuthenticationProcessingFilter를 상속하고 있다.

```java
@Override
public class UsernamePasswordAuthenticationFilter extends AbstractAuthenticationProcessingFilter {
    // ...
    
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response)
            throws AuthenticationException {
        if (this.postOnly && !request.getMethod().equals("POST")) {
            throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
        }
        String username = obtainUsername(request);
        username = (username != null) ? username.trim() : "";
        String password = obtainPassword(request);
        password = (password != null) ? password : "";
        UsernamePasswordAuthenticationToken authRequest = UsernamePasswordAuthenticationToken.unauthenticated(username,
                password);
        // Allow subclasses to set the "details" property
        setDetails(request, authRequest);
        return this.getAuthenticationManager().authenticate(authRequest);
    }
}
```

## UsernamePasswordAuthenticationToken
- Authentication을 상속하고 있는 `AbstractAuthenticationToken`클래스를 상속하고 있다.
- 즉, 최종적으로 SecurityContextHolder.getContext()에 등록될 Authentication를 상속한 객체이다.
- 2개의 생성자가 존재한다.

```java
public class UsernamePasswordAuthenticationToken extends AbstractAuthenticationToken {
    // 1. 미인증 Authentication 객체 생성자
    public UsernamePasswordAuthenticationToken(Object principal, Object credentials) {
        super(null);
        this.principal = principal;
        this.credentials = credentials;
        setAuthenticated(false);
    }

    // 2. 인증 Authentication 객체 생성자
    public UsernamePasswordAuthenticationToken(Object principal, Object credentials,
                                               Collection<? extends GrantedAuthority> authorities) {
        super(authorities);
        this.principal = principal;
        this.credentials = credentials;
        super.setAuthenticated(true); // must use super, as we override
    }
}
```

- UsernamePasswordAuthenticationFilter의 `attemptAuthentication()` 에서는 `1. 미인증 Authentication 객체 생성자`를 먼저 수행하여 DTO 로서 사용한다.
- 추후 인증이 완료 되면 `2. 인증 Authentication 객체 생성자`를 통해 인증된 Authentication을 반환한다.

## getAuthenticationManager()

- `AuthenticationManager`(AbstractAuthenticationProcessingFilter에 존재)를 가져오는 함수이다.
 
### AuthenticationManager
- `AuthenticationProvider` 객체를 관리하는 역할을 수행하는 **인터페이스**이다.
- `UsernamePasswordAuthenticationFilter`에서의 `AuthenticationManager` 구현체는 `ProviderManager`이다.

```java
public interface AuthenticationManager {
    Authentication authenticate(Authentication authentication) throws AuthenticationException;
}
```

```java
public class ProviderManager implements AuthenticationManager, MessageSourceAware, InitializingBean {
    //...
    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        Class<? extends Authentication> toTest = authentication.getClass();
        // ...
        for (AuthenticationProvider provider : getProviders()) {
            if (!provider.supports(toTest)) {
                continue;
            }
            // ...
            try {
                result = provider.authenticate(authentication);

                // ...
            } catch (AccountStatusException | InternalAuthenticationServiceException ex) {
                // ...
            }
            // ...
        }
        //...
    }
}
```

- for 문을 통해 각각의 AuthenticationProvider 의 인증절차를 수행한다.
- `provider.supports(toTest)` : 인자로 넘긴 `toTest(= Authentication 확장)가 Provider에 사용되는 Authentication 타입인지 확인
- `provider.authenticate(authentication)` : 인증 수행 및 결과반환


### AuthenticationProvider

- 실제 인증 로직이 담겨있는 객체(= 최종적으로 실질적인 인증을 수행하는 역할)

```java
public interface AuthenticationProvider {

    // 인증 수행 및 결과(Authentication) 반환
    Authentication authenticate(Authentication authentication) throws AuthenticationException;

    // 인자로 받은 authentication 가 Provider에 사용되는 Authentication 클래스 타입인지 확인
    boolean supports(Class<?> authentication);
}
```

- `boolean supports(Class<?> authentication)`: 인자로 전달받은 authentication이 Provider에서 사용되는 적합한 객체인지 검증

### AbstractUserDetailsAuthenticationProvider

- `ProviderManager`에서 `AuthenticationProvider`의 구현체이다. 

```java
public abstract class AbstractUserDetailsAuthenticationProvider {
    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        // ..
        if (user == null) {
            cacheWasUsed = false;
            try {
                user = retrieveUser(username, (UsernamePasswordAuthenticationToken) authentication);
            } catch (UsernameNotFoundException ex) {
                // ..
            }
            // ..
        }
        try {
            // ..
            additionalAuthenticationChecks(user, (UsernamePasswordAuthenticationToken) authentication);
        }
        catch (AuthenticationException ex) {
            // ..
        }
        // ..
        return createSuccessAuthentication(principalToReturn, authentication, user);
    }
}
```

- `retrieveUser()`
  - UserDetailsService 객체를 통해 UserDetails 객체를 리턴해주는 추상 메소드이다.
  - DaoAuthenticationProvider에서 구현하여 사용한다.
- `additionalAuthenticationChecks(UserDetails userDetails, UsernamePasswordAuthenticationToken authentication)` 
  - 입력받은 id, password 와 실제 id, password 비교
- `createSuccessAuthentication()` : 인증완료 후 수행
  - UsernamePasswordAuthenticationToken 생성자 중 **인증된 토큰을 반환**하는 생성자 호출
  - 인증 완료 후 `AbstractAuthenticationProcessingFilter`의 doFilter()로 반환
  - 이후 AbstractAuthenticationProcessingFilter의 successfulAuthentication 수행

### AbstractAuthenticationProcessingFilter

- successfulAuthentication() 
  - Authentication 객체를 Security Context에 저장

# 참고

- [[스프링] Spring Security 인증은 어떻게 이루어질까? - 소스레벨 분석](https://cjw-awdsd.tistory.com/45)