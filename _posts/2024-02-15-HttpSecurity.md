---
layout: single
title: "HttpSecurity에 관하여..."
categories: spring
---

**`HttpSecurity`** 는 스프링 시큐리티의 설정 중 하나로, 웹 보안 설정을 위한 다양한 메서드를 제공한다. 

이 객체를 사용하여 **인증 방식**, **URL 별 접근 권한**, **로그인 및 로그아웃 처리**, **CSRF 보호 활성화/비활성화** 등 웹 보안과 관련된 세부 설정을 구성할 수 있다.

먼저 각각에 대해서 설명하자면,

1. **URL 별 접근 제어**: **`authorizeHttpRequests()`** 메서드를 사용하여 특정 경로에 대한 접근을 제어한다. 예를 들어, 일부 경로는 **`permitAll()`** 을 통해 모두에게 접근을 허용하고, 그 외의 요청은 **`authenticated()`** 을 통해 인증된 사용자에게만 접근을 허용한다.

2. **폼 기반 로그인 설정**: **`formLogin()`** 메서드를 통해 로그인 페이지, 로그인 처리 URL, 로그인 성공 시 리다이렉션할 URL, 로그인 실패 처리 핸들러 등을 설정한다.

3. **로그아웃 설정**: **`logout()`** 메서드를 사용하여 로그아웃 관련 설정을 한다. 로그아웃 URL, 로그아웃 성공 시 리다이렉션 URL, 세션 무효화 및 인증 정보 클리어 등의 작업을 포함할 수 있다.

4. **CSRF 보호 설정**: **`csrf()`** 메서드를 통해 **`CSRF(Cross-Site Request Forgery)`** 공격 방지 기능을 비활성화한다. 웹 애플리케이션에 따라 CSRF 보호 기능이 필요하지 않을 때 비활성화할 수 있다.

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
	
	@Bean
	public SecurityFilterChain filterChain(HttpSecurity http) throws Exception{
        http.authorizeRequests()
                .antMatchers("/login").permitAll()
                .antMatchers("/detail").hasAnyRole("ADMIN")
                .antMatchers("/main").hasAnyRole("USER")
                .anyRequest().authenticated();
    }
}
```

- **antMatchers**

특정 리소스에 대해서 권한을 설정

- **permitAll**

설정한 리소스의 접근을 인증 없이 허용한다는 의미

- **hasAnyRole**

admin으로 시작하는 모든 URL을을 인증후 ADMIN 레벨의 권한을 가진 사용자만 접근을 허용

- **anyRequest**

인증 후 특정 권한을 가진 사용자만 접근가능한 리소스를 설정하고 그외 나머지 리소스들은 무조건 인증을 완료해야 접근이 가능

```java
http.formLogin()
    .loginPage("/login")
    .loginProcessingUrl("/login")
    .defaultSuccessUrl("/main")
    .successHandler(new CustomAuthenticationSuccessHandler("/main"))
    .failureUrl("login-fail")
    .failureHandler(new CustomAuthenticationFailureHandler("/login-fail"))
```

- **http.formLogin()**  

로그인 폼 페이지와 로그인 처리 성공 실패 등을 사용

- **loginPage("/login")** 

로그인 페이지

- **loginProcessingUrl("/login")** 

로그인시 `/login` 을 호출하여 필터가 호출되어 인증처리 수행. 즉 `UsernamePasswordAuthenticationFilter`가 실행

- **defaultSuccessUrl("/main")**

로그인 성공 후 페이지 이동

- **successHandler(new CustomAuthenticationSuccessHandler("/main"))**

커스텀 핸들러를 생성하여 등록하면 인증성공 후 사용자가 추가한 로직을 수행하고 성공 페이지로 이동

- **failureUrl("/login-fail")**

로그인 실패 시 페이지 이동

- **failureHandler(new CustomAuthenticationFailureHandler("/login-fail"))**

커스텀 핸들러를 생성하여 등록하면 인증실패 후 사용자가 추가한 로직을 수행하고 실패 페이지로 이동

```java
http.logout((logout) -> logout
    .logoutUrl("/logout") 
    .logoutSuccessUrl("/login") 
    .invalidateHttpSession(true) 
    .clearAuthentication(true)
    .permitAll()
    );
```

- **logoutUrl("/logout")**

로그아웃 처리 URL 설정

- **logoutSuccessUrl("/login")**

로그아웃 성공 시 리다이렉션될 URL

- **invalidateHttpSession(true)**

현재 사용자의 HTTP세션 무효화

- **clearAuthentication(true)**

현재 사용자의 인증 정보를 SecurityContext에서 제거

- **permitAll()**

모든 사용자에게 접근 허용






**CSRF에 대해서는 추후에 공부를 하고 작성해보겠다!**
