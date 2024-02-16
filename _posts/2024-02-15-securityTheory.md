---
layout: single
title: "시큐리티 기론 이론과 로그인"
categories: spring
---

- **스프링 시큐리티(Spring Security)**: Spring 기반의 어플리케이션의 보안을 담당하는 스프링 하위 프레임워크

그럼 시큐리티에서 중요한 인증(Authentication)과 인가(Authorization)는 무엇인가?

- **인증(Authentication)**

인증은 사용자가 누구인지 확인하는 과정. 사용자가 로그인 할 때 사용자 이름과 비밀번호를 제공하고, 스프링 시큐리티는 이 정보를 사용하여 사용자의 신원을 확인

- **인가(Authorization)**

인가는 인증된 사용자가 특정 리소스에 접근할 수 있는 권한을 가지고 있는지 확인하는 과정

![기본로직](/images/security.png)

위 표는 스프링 **시큐리티의 역할**을 한눈에 파악할 수 있게 해준다. 

스프링 시큐리티는 클라이언트의 모든 **Http요청을 가로챈다.**

![처리과정](/images/securityLogic.png)

위 이미지는 스프링 시큐리티의 **처리과정**이다.

**`AuthenticationFilter`** 는 **`SecurityFilterChain`** 에 포함되어 있다.

이미지에 대해서 설명하자면,

1. 사용자가 로그인 폼을 통해 사용자 이름과 비밀번호를 입력하고 로그인을 시도하면, **`UsernamePasswordAuthenticationFilter`** 가 이 HTTP 요청을 가로챈다.

2. **`UsernamePasswordAuthenticationToken`** 생성: 필터는 요청에서 사용자 이름과 비밀번호를 추출하여, 아직 인증되지 않은 **`UsernamePasswordAuthenticationToken`** 객체를 생성한다. 이 객체는 사용자의 인증 정보를 담고 있으며, 인증 과정에서 사용된다. 초기 생성 시점에서는 이 객체가 인증되지 않은 상태이며, 사용자의 권한 정보는 비어 있다.

3. 인증 과정: 생성된 **`UsernamePasswordAuthenticationToken`** 객체는 **`AuthenticationManager`** 에게 전달된다. **`AuthenticationManager`** 는 등록된 **`AuthenticationProvider`** 을 사용하여 사용자의 인증 정보를 검증한다. 이 과정에서 주로 **`DaoAuthenticationProvider`** 가 사용되며, 이는 내부적으로 **`UserDetailsService`** 를 사용하여 데이터베이스나 다른 저장소에서 사용자 정보를 조회하고 비밀번호가 일치하는지 확인한다.

4. 인증 성공: 사용자 인증이 성공하면, **`AuthenticationManager`** 는 권한 정보가 포함된 새로운 **`UsernamePasswordAuthenticationToken`** 객체를 반환한다. 이제 이 객체는 인증된 상태로 간주되며, 사용자의 신원과 권한 정보를 담고 있다.

5. 보안 컨텍스트에 인증 정보 저장: 인증된 **`UsernamePasswordAuthenticationToken`** 객체는 스프링 시큐리티의 **`보안 컨텍스트(SecurityContext)`** 에 저장된다.


