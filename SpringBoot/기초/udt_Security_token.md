# 토큰 활용하여 로그인 구현하기

## 필요 클래스
- SecurityFilterChain + PasswordEncoder
- UserDetailsService (DB 조회 후 Details 반환) + UserDetails (유저 정보 객체)
- JwtUtil (create, valid, parse)
- JwtAuthenticationFilter (헤더에서 토큰 검증 후 Auth 객체를 ContextHolder에 저장)

