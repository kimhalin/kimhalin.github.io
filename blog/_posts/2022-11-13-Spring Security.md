---
layout: post
title: Spring Security - 1편(Security Config)
# description: >
#   Howdy! This is an example blog post that shows several types of HTML content supported in this theme.
sitemap: false
hide_last_modified: true
---









## Form 인증

- 사용자가 /home url을 요청하면, 인증이 됐는지, 안됐는지 확인
- 인증이 안되면 로그인 페이지로 redirect
- security context를 session에 저장, 인증 토큰 생성/저장
- 사용자는 세션에 저장된 인증 토큰으로 접근이 가능함

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    publicSecurityFilterChainfilterChain(HttpSecurity http) throws
            Exception {
	http
	        .formLogin()
	        .loginPage("/login.html")    // 사용자 정의 로그인 페이지(Spring security는 기본적으로 제공하는 페이지가 존재
	        .defaultSuccessUrl("/home")  // 로그인 성공 후 이동 페이지
	        .failureUrl("/login.html?error=true")  // 로그인 실패 후 이동 페이지
	        .usernameParameter("username")  //아이디 파라미터명 설정
	        .passwordParameter("password")  //패스워드 파라미터명 설정 
	        .loginProcessingUrl("/login")   //로그인 Form Action Url
	        .successHandler(loginSuccessHandler())  // 로그인 성공 후 핸들러
	        .failureHandler(loginFailureHandler()); // 로그인 실패 후 핸들러
					.permitAll();  //loginPage는 인증을 받지 않아도 접근이 가능하게 해야한다.
        return http.build();
    }

}
```

‼️ 주의할 점은 loginPage는 인증을 받기 위한 페이지이므로, 인증을 받지 않아도 접근이 가능하게 해야한다.

→ permitAll() 추가

## Form Login 진행 과정

1. UsernamePasswordAuthenticationFilter 통과
    
    요청 정보가 매칭되는지 확인
    
2. AntPathRequestMatcher(/login)
3. Authentication(Username + Password) 객체 생성
4. AuthenticationManager가 받아서 AuthenticationProvider에 인증 위임
5. AuthenticationProvider에서 인증 성공하면 AuthenticationManager에 Authentication 객체 전달
    
    실패하면 AuthenticationException
    
6. Authentication 객체에는 User와 Authorities가 들어있고, 이를 SecurityContext에 저장
7. SuccessHandler 호출

## Logout 처리

1. request (/logout)
2. 세션 무효화, 인증토큰 삭제, 쿠키정보 삭제, 로그인 페이지로 redirect

```java
http.logout()
        .logoutUrl("/logout")
        .logoutSuccessUrl("/login")
        .deleteCookies("JSESSIONID", "remember-me")
        .addLogoutHandler(new LogoutHandler() {
            @Override
            public void logout(HttpServletRequestrequest,HttpServletResponseresponse,Authenticationauthentication) {

            }
        })
        .logoutSuccessHandler(new LogoutSuccessHandler() {
            @Override
            public void onLogoutSuccess(HttpServletRequestrequest,HttpServletResponseresponse,Authenticationauthentication) throws IOException, ServletException {

            }
        });
```

## Logout 진행 과정

1. LogoutFilter
2. AntPathRequestMatcher(/logout)
3. Authentication 객체를 SecurityContext에서 꺼내온다
4. SecurityContextLogoutHandler
    - 세션 무효화
    - 쿠키 삭제
    - SecurityContextHolder.clearContext()
5. 로그아웃 성공 시 SimpleUrlLogoutSuccessHandler → /login으로 redirect

<aside>
💡 Handler는 여러 개를 가지고 있고, 순차적으로 handler의 로직을 각각 밟아나간다.

</aside>

## Remember Me 인증

- 세션이 만료되고 웹 브라우저가 종료된 후에도 어플리케이션이 사용자를 기억하는 기능
- Remember-Me 쿠키에 대한 Http 요청을 확인 → 토큰 기반 인증을 사용해 유효성을 검사하고 토큰이 검증되면 사용자는 로그인된다.

```java
http.rememberMe()
        .rememberMeParameter("remember"). //파라미터명
        .tokenValiditySeconds(3600)  //토큰 유효기간 (단위: 초)
        .alwaysRemember(true)        //설정 안해도 항상 Remember 기능 활성화
        .userDetailsService(userDetailService);
```