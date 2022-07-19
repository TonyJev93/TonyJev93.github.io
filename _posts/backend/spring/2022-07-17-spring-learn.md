---
title: "[Spring] 학습내용 정리"
last_modified_at: 2022-07-18T23:50:00+09:00
categories:
    - Back End
    - Spring
tags:
    - Back End
    - Spring
toc: true
toc_sticky: true
toc_label: "목차"
---

Spring : Spring과 관련된 다양한 지식들을 정리한다.
{: .notice--info}

# Spring Security

## CORS 해결하기

### 1) WebMvcConfigurer addCorsMappings

- WebMvcConfigurer addCorsMappings 구현 (in Spring)
```java
@RequiredArgsConstructor
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

    private final LoginUserIdArgumentResolver loginUserIdArgumentResolver;

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
            .allowedOrigins("http://localhost:3000")
            .allowedMethods("OPTIONS", "GET", "POST", "PUT", "DELETE");
    }
}
```

- Preflight Request 허용 (in Spring)
```java
@RequiredArgsConstructor
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
            .mvcMatchers(HttpMethod.OPTIONS, "/**").permitAll() // Preflight Request 허용해주기
            .antMatchers("/api/v1/**").hasAnyAuthority(USER.name());
    }
}
```

### 2) React Proxy를 사용해서 SOP 해결하기

- http-proxy-middleware 모듈 설치
```bash
$ yarn add http-proxy-middleware
```

- setProxy.js 설정
  - `http://localhost:3000/api/v1 -> (proxy) -> http://localhost:8080/api/v1`

```javascript
// /src/setProxy.js
const { createProxyMiddleware } = require('http-proxy-middleware');

module.exports = function (app) {
  app.use(
    createProxyMiddleware('/api/v1', {
      target: 'http://localhost:8080', // Server 의 도메인과 일치하는 URL
      changeOrigin: true,
    })
  );
};
```

- Proxy가 적용되지 않는 경우
  - yarn.lock node_modules 캐시문제일 수 있으므로 제거

```bash
$ rm -rf yarn.lock node_modules
$ yarn install
```

### 참고

- [[Spring] Spring Security, React를 사용하면서 CORS 허용하는 방법](https://devlog-wjdrbs96.tistory.com/429)

# Data Source

### 참고
- [[Spring Boot] JavaConfig로 Datasource 설정하기](https://blog.jiniworld.me/69)