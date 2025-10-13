# 세션 활용하여 로그인 구현하기

## 필요 클래스
- SecurityConfig (PasswordEncoder)
- CustomUserDetails
- CustomUserDetailsService (UserDetails 반환)

## 설명
- name과 password 값으로 로그인, HTML id 값 통일해줘야 함 form input

### Config
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    private static final RequestMatcher[] PUBLIC_MATCHERS = {
            new AntPathRequestMatcher("/", "GET"),
            new AntPathRequestMatcher("/signup", "POST"),
            new AntPathRequestMatcher("/login", "POST"),
            new AntPathRequestMatcher("/posts", "GET"),
            new AntPathRequestMatcher("/posts/**", "GET")
    };

    private static final RequestMatcher[] USER_MATCHERS = {
            new AntPathRequestMatcher("/mypage", "GET"),
            new AntPathRequestMatcher("/posts", "POST")
    };

    private static final RequestMatcher[] ADMIN_MATCHERS = {
            new AntPathRequestMatcher("/admin/**"),
            new AntPathRequestMatcher("/posts/**", "DELETE")
    };

    @Bean
    public BCryptPasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests(this::configureAuthorizationRules);
        
        http
                .formLogin((auth) -> auth
                        .usernameParameter("name")
                        .passwordParameter("password")
                        .loginPage("/login") //GET 
                        .defaultSuccessUrl("/", true)
                        .loginProcessingUrl("/login-proc") //POST
                        .permitAll()
                );

        http.csrf(AbstractHttpConfigurer::disable);

        return http.build();
    }

    private void configureAuthorizationRules(
            AuthorizeHttpRequestsConfigurer<HttpSecurity>.AuthorizationManagerRequestMatcherRegistry auth) {
        auth
                .requestMatchers(PUBLIC_MATCHERS).permitAll()
                .requestMatchers(USER_MATCHERS).hasAnyRole("USER", "ADMIN")
                .requestMatchers(ADMIN_MATCHERS).hasRole("ADMIN");
        
        auth.anyRequest().authenticated();
    }

}
```


### CustomUserDetails
```java
@RequiredArgsConstructor
public class CustomUserDetails implements UserDetails {
    private final User user;
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        Collection<GrantedAuthority> authorities = new ArrayList<>();
        authorities.add(() -> String.valueOf(user.getRole())); // ENUM ROLE
        return authorities;
    }


    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getName();
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}

```


### CustomUserDetailsService
```java
@Service
@RequiredArgsConstructor
public class CustomUserDetailsService implements UserDetailsService {
    private final UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) {
        User user = userRepository.findByName(username)
                .orElseThrow(() -> new UsernameNotFoundException("해당 사용자를 찾을 수 없습니다: " + username));

        return new CustomUserDetails(user);
    }
}
```

