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
toc: true
toc_sticky: true
toc_label: "목차"
---

Spring : Spring Security 원리를 이해한 대로 정리해보자.
{: .notice--info}

# Spring Security

- Filter 리스트 조회 in FilterChainProxy.getFilters()

- Security 의 Default 인증 필터 = Session/Cookie 방식 in UsernamePasswordAuthenticationFilter

- UsernamePasswordAuthenticationFilter = AbstractAuthenticationProcessingFilter 를 상속

- AbstractAuthenticationProcessingFilter
  - doFilter() : 필터의 로직이 들어있음. UsernamePasswordAuthenticationFilter의 attemptAuthentication()를 호출
  - attemptAuthentication() 추상메소드

- attemptAuthentication() in UsernamePasswordAuthenticationFilter
  - UsernamePasswordAuthenticationToken: AbstractAuthenticationToken을 상속
    - AbstractAuthenticationToken: Authentication을 상속 = SecurityContextHolder.getContext() 에 등록될 객체
    - UsernamePasswordAuthenticationToken 생성자 2개 존재
        1. setAuthenticated(false); // <- attemptAuthentication()에서는 해당 생성자 사용
            - 처음에는 미인증된 Authentication 생성하고 추후 인증완료 되면 인증된 Authentication 생성
        2. setAuthenticated(true);
  - getAuthenticationManager(): 상속한 AbstractAuthenticationProcessingFilter의 AuthenticationManager를 가져오는 함수
    - AuthenticationManager 란: AuthenticationProvider라는 클래스 객체를 관리
      - AuthenticationProvider: 실제 인증 로직이 담겨있는 객체
      - Authentication authenticate(Authentication authentication): 인증로직을 통한 결과를 Authentication으로 반환
      - AuthenticationManager의 구현체 (in UsernamePasswordAuthenticationFilter) = ProviderManager

- ProviderManager(AuthenticationManager)
  - authenticate(): AuthenticationProvider을 for문을 통해 인증수행
  - AuthenticationProvider
    - Authentication authenticate(Authentication authentication): 실제 인증로직 수행
    - boolean supports(Class<?> authentication): 인자로 전달받은 authentication가 Provider에서 사용되는 적합한 객체인지 검증
    - AuthenticationProvider 구현체 = AbstractUserDetailsAuthenticationProvider in ProviderManager

- AbstractUserDetailsAuthenticationProvider
  - retrieveUser()
    - 추상 메소드(DaoAuthenticationProvider 에서 구현)
      - DaoAuthenticationProvider
        - retrieveUser(): UserDetailsService 객체를 통해 UserDetails 가져옴
        - additionalAuthenticationChecks() : 입력받은 id, password 와 실제 id, password 비교 
    - UserDetails 객체를 리턴
  - createSuccessAuthentication(): 인증완료 후 수행
    - UsernamePasswordAuthenticationToken 생성자 중 인증된 토큰을 반환하는 생성자 호출
    - 인증 완료 후 AbstractAuthenticationProcessingFilter의 doFilter()로 반환
    - AbstractAuthenticationProcessingFilter의 successfulAuthentication 수행

- successfulAuthentication() 
  - Authentication 객체를 Security Context에 저장

# 참고

- [[스프링] Spring Security 인증은 어떻게 이루어질까? - 소스레벨 분석](https://cjw-awdsd.tistory.com/45)