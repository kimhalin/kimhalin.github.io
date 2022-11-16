---
layout: post
title: Spring Security - 4편
# description: >
#   Howdy! This is an example blog post that shows several types of HTML content supported in this theme.
sitemap: false
hide_last_modified: true
---
spring security 관련 강의를 들으며 작성한 글입니다.








# Spring Security - 4

## Authentication

사용자의 인증 정보를 저장하는 토큰 개념

- 인증 시 id와 password를 담고, 인증 검증을 위해 전달되어 사용
- 인증 후 최종 인증 결과(user 객체, 권한 정보)를 담고 SecurityContext에 저장되어 전역적으로 참조가 가능하다

```java
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
```

### 구조

- principal: 사용자 아이디 혹은 User객체 저장
- credentials: 사용자 비밀번호
- authorities: 인증된 사용자의 권한 목록
- details: 인증 부가 정보
- Authenticated: 인증 여부

### Authentication 흐름

1. `authRequest`(UsernamePasswordAuthenticationToken)을 `Authenticationmanager`한테 맡긴다.
2. 인증 받은 후에 결과로 return 받은 Authentication객체를 SecurityContext에 넣는다.

## SecurityContext

`Authentication` 객체가 저장되는 보관소 → 필요 시 언제든지 Authentication 객체를 꺼내어 쓸 수 있도록 제공되는 클래스

- `ThreadLocal`에 저장되어 아무 곳에서나 참조가 가능하도록 설계함
- 인증이 완료되면 `HttpSession`에 저장되어 어플리케이션 전반에 걸쳐 전역적인 참조 가능

### SecurityContextHolder

- SecurityContext 객체 저장 방식
    - MODE_THREADLOCAL: 스레드당 SecurityContext 객체 할당, 기본값
        
         ex) 만약 메인 thread와 자식 thread가 생성되었을 때, 각각 다른 SecurityContext를 ThreadLocal에 가지고 있다.
        
    - MODE_INHERITABLETHREADLOCAL: 메인 스레드와 자식 스레드에 관하여 동일한 SecurityContext를 유지
    - MODE_GLOBAL: 응용 프로그램에서 단 하나의 SecurityContext를 저장

```java
SecurityContextHolder.clearContext() → SecurityContext 기존 정보 초기화
```

💡 `SecurityContextHolder`가 `SecurityContext`를 저장하고 있고, `SecurityContext`안에 `Authentication` 객체를 저장하고 있다.

**SecurityContextHolder → ThreadLocal → SecurityContext → Authentication**

## SecurityContextPersistenceFilter

SecurityContext 객체의 생성, 저장, 조회

- 익명 사용자
    - **새로운 SecurityContext 객체 생성** → **SecurityContextHolder에 저장**
    - AnonymousAuthenticationFilter에서 AnonymousAuthenticationToken 객체를 SecurityContext에 저장
- 인증 시
    - **새로운 SecurityContext 객체 생성** → **SecurityContextHolder에 저장**
    - UsernamePasswordAuthenticationFilter에서 인증 성공 후 SecurityContext에 UsernamePasswordAuthentication객체를 SecurityContext에 저장
    - 인증이 최종 완료되면 Session에 SecurityContext를 저장
- 인증 후
    - **Session에서 SecurityContext 꺼내어** **SecurityContextHolder에서 저장**
    - SecurityContext 안에 Authentication 객체가 존재하면 계속 인증을 유지
- 최종 응답 시 공통
    - SecurityContextHolder.clearContext();

## Authentication Flow

1. **Client**가 Request Login → **UsernamePasswordAuthenticationFilter**는 Authentication(id + pwd) 를 authenticate(Authentication) 함수 호출을 통해 **AuthenticationManager**로 Authentication (인증 전) 객체 전달
2. **AuthenticationManager**는 직접 인증 과정에 참여 X, authenticate(Authenticate) 함수 호출을 통해 적절한 **AuthenticationProvider**에게 위임
3. **AuthenticationProvider**는 loadUserbyUsername(username) 함수 호출해서 **UserDetailsService**에서 실제 인증 처리 역할(유저 유효성 검증)을 하도록, 한다.
4. **UserDetailsService**는 findById() 함수를 통해 이 유저 객체가 DB에 있는지 확인한 후에 UserDetails 타입 객체로 **AuthenticationProvider**에 반환
5. **AUthenticationProvider**는 인증 후인 토큰 객체(UserDetails + authorities) 생성해서 **AuthenticationManager**에게 return
6. **AuthenticationManager**는 다시 **UsernamePasswordAuthenticationFilter**에게 Authentication(인증 후) 객체 return
7. **UsernamePasswordAuthenticationFilter**는 **SecurityContext**에 인증 객체 저장

📌  결론

**Client → UsernamePasswordAuthenticationFilter → AuthenticationManager → AuthenticationProvider → UserDetailsService→ Repository → UserDetailsService → AuthenticationProvider → AuthenticationManager → UsernamePasswordAuthenticationFilter → Client**

## AuthenticationManager

AuthenticationProvider 목록 중에서 인증 처리 요건에 맞는 AuthenticationProvider를 찾아 인증처리를 위임

부모 ProviderManager를 설정하여 AuthenticationProvider를 계속 탐색 가능

## AuthenticationProvider

- **authenticate(authentication)**
    1. ID 검증: UserDetailsService → UserNotFoundException
    2. password 검증: BadCredentialException
    3. 추가 검증: Authentication(user, authorities) → AuthenticationManager

## Authorization

당신에게 무엇이 허가 되었는지 증명하는 것 ⇒ ROLE 확인

### 스프링 시큐리티가 지원하는 권한 계층

- 웹 계층: URL 요청에 따른 메뉴 혹은 화면 단위의 레벨 보안
- 서비스 계층: 화면 단위가 아닌 메소드 같은 기능 단위의 레벨 보안
- 도메인 계층(Access Control List, 접근제어목록): 객체 단위의 레벨 보안

### FilterSecurityInterceptor

마지막에 위치한 필터, 인증된 사용자에 대해 **특정 요청의 승인/거부 여부를 최종적으로 결정**

권한 제어 방식 중 HTTP 자원의 보안 처리 필터

권한 처리를 AccessDecisionManager에게 맡김

### AccessDecisionManager

인증 정보, 요청 정보, 권한 정보를 이용 → 사용자의 지원 접근을 허용 또는 거부할 것인지 최종 결정 주체

여러 개의 Voter들을 가질 수 있고, Voter들로부터 접근허용, 거부, 보류에 해당하는 각각의 값을 리턴받고 판단 및 결정

### 접근결정 유형

- **AffirmativeBased**: 여러 개의 Voter 클래스 중 하나라도 접근 허가로 결론을 내면 접근 허가로 판단
- **ConsensusBased**: 다수결, 동수일 경우 기본은 접근허가 allowIfEqualGrantedDeniedDecisions를 false로 설정할 경우 접근 거부로 결정
- **UnanimmousBased**: 모든 Voter가 만장일치로 접근을 승인해야 승인

### AccessDecisionVoter

판단 심사

**권한 부여 과정에서 판단하는 자료(AccessDecisionManager가 Voter에게 전달하는 정보)**

- Authentication - 인증 정보(User)
- FilterInvocation - 요청 정보(antMatcher(”/user”))
- ConfigAttributes - 권한 정보(hasRole(”USER”))