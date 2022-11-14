---
layout: post
title: Spring Security - 2편(Security Config)
# description: >
#   Howdy! This is an example blog post that shows several types of HTML content supported in this theme.
sitemap: false
hide_last_modified: true
---
spring security 관련 강의를 들으며 작성한 글입니다.








# Spring Security - 2

## Remember Me 인증 과정

1. RememberMeAuthenticationFilter 가 작동
    - 인증 객체가 없는 경우
        
        인증을 받은 사용자의 Authentication 인증 객체는 SecurityContext에 저장
        
        세션 안에서 SecurityContext를 못 찾고, Authentication 객체도 null일 경우에
        
    - Remember-me 쿠키를 가져오는 경우
2. RememberMeService
    
    토큰 추출
    
    - TokenBasedRememberMeServices
    - PersistentTokenBasedRememberMeServices
3. 사용자가 가지고 있는 Token이 Rememberme Token 존재하는가 확인
4. Token을 Decode한 후, 사용자의 값과 서버의 Token이 서로 일치하는지 확인
5. User 정보를 통해서 DB에 User가 존재하면, 새로운 Authentication 생성
    
    그 인증객체를 AuthenticationManager에 전달해서 인증 과정 진행
    

## AnonymousAuthenticationFilter

인증을 받지 않은 사용자는 null로 처리를 한다. 하지만 이 필터는 null로 판단하지 않고, 익명 사용자용 인증 객체를 만들어서 처리

익명이어도 한 번 인증 객체를 생성하면 일반 인증 사용자처럼 SecurityContext에 Authentication 객체를 저장한다.

### 구현 방법

- isAnnonymous() 와 isAuthenticated()로 구분해서 사용
- 인증 객체를 세션에 저장하지는 않는다.

## 동시 세션 제어

같은 사용자가 로그인 여러 번 시도할 경우에, 최대 세션 허용 개수 초과문제 전략 2가지

- 이전 사용자 세션 만료
- 현재 사용자 인증 실패

```java
http.sessionManagement()
        .maximumSessions(1) // 최대 허용 가능 세션수, -1: 무제한 로그인 세션 허용
        .maxSessionsPreventsLogin(true) // 동시 로그인 차단함(새로운 세션 실패하도록), false: 기존 세션 만료(default)
        .expiredUrl("/expired");        // 세션이 만료된 경우 이동할 페이지
```

## 세션 고정 보호

- 공격자가 서비스에 접속하고, 받은 JSESSIONID를 사용자에게 전달한다.
- ✏️사용자는 공격자의 세션 쿠키로 서비스에 로그인 시도하고, 로그인 성공
- 공격자 쿠키 값으로 인증되어 있기 때문에 로그인을 따로 하지 않아도, **공격자는 사용자의 정보를 공유하게 된다**

### 💡 어떻게 방지할까?

인증할 때마다 새로운 세션, 쿠키 생성 → 공격자는 공격자가 받은 쿠키로 접속이 불가하다

```java
http.sessionManagement()
        .sessionFixation().changeSessionId();
```

### ✏️ 세션 정책

```java
http.sessionManagement()
        .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED)
```

- SessionCreationPolicy.Always → 스프링 시큐리티 항상 세션 생성
- SessionCreationPolicy.If_Required → 스프링 시큐리티가 필요 시 생성(기본값)
- SessionCreationPolicy.Never → 스프링 시큐리티가 생성하지 않지만 이미 존재하면 사용
- SessionCreationPolicy.Stateless → 스프링 시큐리티가 생성하지 않고 존재해도 사용하지 않음
    
    세션을 아예 생성하지 않는 인증 방식을 도입하고자 할 때: 대표적인 예(JWT)
    

## Security의 Filter

### 1. SessionManagermentFilter

- **세션 관리**
    
    인증 시 사용자의 세션정보를 등록, 조회, 삭제 등의 세션 이력 관리
    
- **동시적 세션 제어**
    
    동일 계정으로 접속이 허용되는 최대 세션 수를 제한
    
- **세션 고정 보호**
    
    인증할 때마다 세션쿠키를 새로 발급하여 공격장의 쿠키 조작 방지
    
- **세션 생성 정책**
    
    Always, If_Required, Never, Stateless
    

### 2. ConcurrentSessionFilter

이전 사용자(세션이 이미 있는)의 request를 처리

- 매 요청마다 현재 사용자의 세션 만료 여부 체크
- 세션이 만료되었을 경우 즉시 만료 처리
    
    session.isExpired() == true → 로그아웃 처리, 즉시 오류 페이지 응답(This session has been expired)
    

## 권한 설정

### 선언적 방식

- URL
    - http.antMatchers(”/users/**”).hasRole(”USER”)
- Method
    
    ```java
    @PreAuthorize("hasRole('USER')")
    public void user() { System.out.println("user") }
    ```
    

### 동적 방식 - DB 연동 프로그래밍

- URL
- Method

### Code

```java
http.antMatcher("/shop/**")
        .authorizeRequests()
        .antMatchers("/shop/login", "/shop/users/**").permitAll()
        .antMatchers("/shop/mypage").hasRole("USER")
        .antMatchers("/shop/admin/pay").access("hasRole('ADMIN')")
        .antMatchers("/shop/admin/**").access("hasRole('ADMIN') or hasRole('SYS')")
        .anyRequest().authenticated(); //모든 요청에는 인증을 받은 사용자만이 access 가능
```

`antMatcher`를 통해 설정한 url의 request들은 보안동작을 하게 된다. (만약 antMatcher를 설정하지 않으면 모든 url에 대해 보안 동작)

‼️ **설정 시 구체적인 경로가 먼저 오고, 그것보다 크 범위이 경로가 뒤에 오도록 해야한다.**

## ExceptionTranslationFilter

### AuthenticationException

- 인증 예외 처리
    1. AuthenticationEntryPoint 호출
        
        로그인 페이지 이동, 401 오류 코드 전달 등
        
    2. 인증 예외가 발생하기 전의 요청 정보를 저장
        
        RequestCache: 사용자의 이전 요청 정보를 세션에 저장하고, 이를 꺼내 오는 캐시 메카니즘
        
        - SavedRequest: 사용자가 요청했던 request 파라미터 값들, 그 당시 헤더값 등이 저장

### AccessDeniedException

- 인가 예외 처리
    
    AccessDeniedHandler에서 예외 처리하도록 제공
    

```java
http.formLogin()
        .successHandler(new AuthenticationSuccessHandler() {
            @Override
            public void onAuthenticationSuccess(HttpServletRequestrequest,HttpServletResponseresponse,Authenticationauthentication) throws IOException, ServletException {
RequestCacherequestCache = new HttpSessionRequestCache();
SavedRequestsavedRequest = requestCache.getRequest(request, response);
                String redirectUrl = savedRequest.getRedirectUrl();
                response.sendRedirect(redirectUrl);
            }
        })
http.exceptionHandling()
        .authenticationEntryPoint(new AuthenticationEntryPoint() {
            @Override
            public void commence(HttpServletRequestrequest,HttpServletResponseresponse, AuthenticationException authException) throws IOException, ServletException {
                response.sendRedirect("/login");
            }
        })
        .accessDeniedHandler(new AccessDeniedHandler() {
            @Override
            public void handle(HttpServletRequestrequest,HttpServletResponseresponse, AccessDeniedException accessDeniedException) throws IOException, ServletException {
                response.sendRedirect("/denied");
            }
        });
```

## CSRF(사이트 간 요청 위조)

1. 사용자: 로그인 후 쿠키를 발급 받음
2. 공격자: 링크를 이용자에게 전달
3. 사용자: 링크를 클릭하며 공격용 웹페이지에 접속하고, 그 공격용 페이지를 열면, 브라우저는 이미지 파일을 받아오기 위해 공격용 URL을 연다
4. 사용자의 승인이나 인지 없이 배송지가 등록됨으로써 공격이 완료된다.

### CsrfFilter

- 모든 요청에 **랜덤하게 생성된 토큰(csrf토큰)을 HTTP 파라미터로 요구**
- 요청 시 전달되는 토큰 값과 서버에 저장된 실제 값과 비교한 후 만약 일치하지 않으면 요청 실패