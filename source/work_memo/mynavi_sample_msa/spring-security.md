# AWSで作るマイクロサービス（Spring Securityを使ったログイン処理実装まで）
## 実施したこと概要
[AWSで作るマイクロサービス](https://news.mynavi.jp/itsearch/series/devsoft/aws_2.html)の第3回〜第4回を実施

## 実施したこと詳細
1 frontend用プロジェクトの作成
 * groupId: org.hanedan.msexample.frontend
 * artifactId: ms-sample-frontend
 * dependency
   + spring-boot-starter-security
   + spring-boot-starter-thymeleaf
   + spring-boot-starter-web
   + lombok

2 SecurityConfig.javaの作成
```java
@EnableWebSecurity // Spring Security設定クラスと認識させるためのアノテーション
public class SecutiryConfig extends WebSecurityConfigurerAdapter { // Spring Securiyの基本設定を継承

    @Autowired
    PasswordEncoder passwordEncoder;

    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().antMatchers("/static/**");
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/favicon.ico").permitAll()
                .antMatchers("/webjars/**").permitAll()
                .antMatchers("/static/**").permitAll()
                .antMatchers("/timeout").permitAll()
                .anyRequest().authenticated()
                .and()
                .csrf().disable()
                .formLogin()
                .loginProcessingUrl("/authenticate")
                .loginPage("/login")
                .successHandler(loginSuccessHandler())
                .failureUrl("/login")
                .usernameParameter("username")
                .passwordParameter("password")
                .permitAll()
                .and()
                .exceptionHandling()
                .authenticationEntryPoint(authenticationEntryPoint())
                .and()
                .logout()
                .logoutSuccessUrl("/login")
                .permitAll();
    }

    @Bean
    public LoginSuccessHandler loginSuccessHandler(){
        return new LoginSuccessHandler();
    }

    @Bean
    AuthenticationEntryPoint authenticationEntryPoint() {
        return new SessionExpiredDetectingLoginUrlAuthenticationEntryPoint("/login");
    }

    @Bean
    public PasswordEncoder passwordEncoder(){
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }

    @Override
    protected UserDetailsService userDetailsService(){
        return new CustomUserDetailsService();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService())
                .passwordEncoder(passwordEncoder);
    }
    
}
```
3 CustomUserDetailsService.javaの作成
```java
public class CustomUserDetailsService implements UserDetailsService {

    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        return CustomUserDetails.builder()
                .authorities(AuthorityUtils.createAuthorityList("ROLE_USER"))
                .build();
    }
}
```
4 CustomUserDetails.javaの作成
```java
@Data
@Builder
@AllArgsConstructor
public class CustomUserDetails  implements UserDetails {

    private final Collection<? extends GrantedAuthority> authorities;

    @Override
    public Collection getAuthorities() {
        return authorities;
    }

    @Override
    public String getPassword(){
        return "{noop}test"; // {noop}はNoOpPasswordEncoder（ハッシュ化も暗号化もしない）を使用する場合に付与
    }

    @Override
    public String getUsername(){
        return "test";
    }

    @Override
    public boolean isAccountNonExpired(){
        return true;
    }

    @Override
    public boolean isAccountNonLocked(){
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled(){
        return true;
    }
}
```
5 LoginSuccessHandler.java
```java
public class LoginSuccessHandler implements AuthenticationSuccessHandler {

    @Override
    public void onAuthenticationSuccess(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication)
            throws IOException, ServletException {
        httpServletResponse.sendRedirect("/frontend/portal");

    }
}
```

6 SessionExpiredDetectingLoginUrlAuthneticationEntryPoint
```java
public class SessionExpiredDetectingLoginUrlAuthenticationEntryPoint extends LoginUrlAuthenticationEntryPoint {

    public SessionExpiredDetectingLoginUrlAuthenticationEntryPoint(String loginFormUrl) {
        super(loginFormUrl);
    }

    @Override
    protected String buildRedirectUrlToLoginPage(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) {
        String redirectUrl = super.buildRedirectUrlToLoginPage(request, response, authException);

        if(isRequestedSessionInvalid(request)){
            redirectUrl = "/timeout";
        }
        return redirectUrl;
    }

    private boolean isRequestedSessionInvalid(HttpServletRequest request){
        return Objects.nonNull(request.getRequestedSessionId())
                && !request.isRequestedSessionIdValid();
    }
}
```
7 SampleController
```java
@Controller
public class SampleController {

    @GetMapping("/login")
    public String login(){
        return "login";
    }

    @GetMapping("/portal")
    public String portal(){
        return "portal";
    }

    @GetMapping("/timeout")
    public String timeout(){
        return "timeout";
    }

}
```
8 MvcConfig.java
```java
@ComponentScan("org.hanedan.msexample.frontend.webapp.app.web")
@Configuration
public class MvcConfig implements WebMvcConfigurer {
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/static/**")
                .addResourceLocations("classpath:/static/");
    }

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("forward:/login");
    }
}
```
9 ログインページ､ポータルトップ、セッションタイムアウトページ作成
省略

10 application.yml
```
server:
  servlet:
    context-path: /frontend
```

11 その他
 * ログアウトせずに複数回loginページにアクセスするとタイムアウト画面に遷移してしまう。/のときはタイムアウト画面に遷移させないようにする
