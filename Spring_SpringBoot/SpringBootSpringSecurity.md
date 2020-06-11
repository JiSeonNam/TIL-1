# 스프링 부트 개념과 활용 Spring Security
[Reference](https://docs.spring.io/spring-security/site/docs/current/reference/html5/)
## Starter-Security 
- 스프링 부트 애플리케이션에 spring-boot-starter-security 의존성만 추가하면 spring security가 기본적으로 적용된다. 
<br>

### 스프링 시큐리티
- 웹 시큐리티
- 메소드 시큐리티
- 다양한 인증 방법 지원
    * LDAP, 폼 인증, Basic 인증, OAuth, ...
<br>

### 스프링 부트 시큐리티 자동 설정
- SecurityAutoConfiguration
- UserDetailsServiceAutoConfiguration
    * 자동으로 랜덤한 user를 생성해주는 자동 설정
    * 스프링 부트 애플리케이션이 구동 될 때 기본적으로 user객체 하나를 inMemoryUserDetailsManager를 만들어서 제공해준다.
    * 이 설정은 AuthenticationManager, AuthenticationProvider, UserDetailsService 가 없는 경우에만 적용된다.
    * 보통 스프링 시큐리티를 적용하는 프로젝트들은 각 프로젝트만의 UserDetailsService를 등록하게 되어 있어서 사실상 UserDetailsServiceAutoConfiguration 설정은 거의 쓸 일이 없다.
- spring-boot-starter-security
    * 스프링 시큐리티 5.* 의존성 추가
- 모든 요청에 인증이 필요함.
- 기본 사용자 생성
    * Username: user
    * Password: 애플리케이션을 실행할 때 마다 랜덤 값 생성 (콘솔에 출력 된다)
    * spring.security.user.name
    * spring.security.user.password
- 인증 관련 각종 이벤트 발생
    * DefaultAuthenticationEventPublisher 빈 등록
    * 다양한 인증 에러 핸들러 등록 가능
<br>

### Spring-Security 실습
- Thymeleaf 의존성 추가
```html
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```
- Controller 생성
```java
@Controller
public class HomeController {

    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }

    @GetMapping("/my")
    public String my() {
        return "my";
    }
}
```
- src/resource/templates에 index.html, hello.html, my.html 생성
    * hello, my는 그냥 각각 hello, my만 출력하도록 생성
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Title</title>
</head>
<body>
<h1>Welcome</h1>
<a href="/hello">hello</a>
<a href="/my">my page</a>
</body>
</html>
```
- Controller Test 코드 만들기
```java
@RunWith(SpringRunner.class)
@WebMvcTest(HomeController.class)
public class HomeControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void hello() throws Exception {
        mockMvc.perform(get("/hello"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(view().name("hello"));
    }

    @Test
    public void my() throws Exception {
        mockMvc.perform(get("/my"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(view().name("my"));
    }
}
```
- **Spring-Security 추가하기**
    * 의존성 추가
    * 추가만 하고 실행하면 오류가 난다.
    * 모든 요청이 인증을 필요로 하게 된다.
```html
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```
- 가짜 유저 인증정보 적용
    * @WithMockUser를 적용하면 가짜 유저 인증정보를 적용하여 돌려준다.
    * 메소드마다 등록 할 수 있고 클래스에 등록해 클래스의 모든 메소드에 등록할 수도 있다.
    * 인증정보가 있으면 테스트 성공, 인증정보가 없으면 status 401에 Unauthorized 메시지를 응답한다.
```java
@RunWith(SpringRunner.class)
@WebMvcTest(HomeController.class)
public class HomeControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    @WithMockUser
    public void hello() throws Exception {
        mockMvc.perform(get("/hello"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(view().name("hello"));
    }

    @Test
    public void hello_without_user() throws Exception {
        mockMvc.perform(get("/hello"))
                .andDo(print())
                .andExpect(status().isUnauthorized());
    }

    @Test
    @WithMockUser
    public void my() throws Exception {
        mockMvc.perform(get("/my"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(view().name("my"));
    }

}
```
<br>

## Spring-Security 설정 커스터마이징
- 스프링부트가 제공하는 기본설정 두가지
    * WebSecurityConfigureAdapter 설정
        - Form인증과 Basic Authentication을 활성화 시켜줌
        - 모든 요청이 다 Authentication이 필요하다고 정의
    * UserDetailsService 설정
        - 기본으로 랜덤한 유저를 생성해주는 건 UserDetailsServiceAutoConfiguration
<br>

### Web Security Configuration(웹 시큐리티 설정)
- hello나 index페이지는 아무나 접근하고 my 페이지만 인증을 한 사용자가 방문하도록
- WebSecurityConfigurerAdapter의 bean을 등록하면 SpringBoot가 제공하는 SecurityAutoConfiguration은 사용이 안된다.
- 정의 하는대로 동작한다.
- thymeleaf, security 의존성이 추가되어있고, controller와 3개의 html 파일이 생성되어 있는 상황에서 SecurityConfig 생성
    * index와 hello는 인증을 예외해서 접근이 가능하게 한다.
    * my는 Accept Header에 html이 있으므로 formLogin에 걸려서 로그인 인증요청 화면으로 이동하게 한다.
    * html이 없는 경우에는 httpBasic에 걸린다. 
```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/","/hello").permitAll()
                .anyRequest().authenticated()
                .and()
            .formLogin()
                .and()
            .httpBasic();
    }
}
```
<br>

### UserDetailsServie 구현
- 스프링 부트에서 자동으로 유저를 만들어주는 것이 아닌 직접 Account를 관리 하도록 한다.
- JPA와 H2 의존성 추가
```html
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
```
- Account 클래스 생성
```java
@Entity
public class Account {

    @Id @GeneratedValue
    private Long id;
    private String username;
    private String password;

    ...getter and setter...
}
```
- AccountRepository 생성
    * JPA 설정이 되었고 H2를 추가하였으므로 자동으로 H2를 사용하도록 설정된다.
    * 이렇게 하면 일단 JPA쪽은 완성
```java
public interface AccountRepository extends JpaRepository<Account, Long> {

}
```
- AccountService 생성
```java
@Service
// 보통 유저 정보를 관리하는 AccountService에다가 UserDetailsService 인터페이스를 구현
// 중요한건 UserDetailsService 타입의 bean이 등록되는 것이다.
public class AccountService implements UserDetailsService {

    @Autowired
    private AccountRepository accountRepository;

    public Account createAccount(String username, String password) {
        Account account = new Account();
        account.setUsername(username);
        account.setPassword(password);
        return accountRepository.save(account);
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Optional<Account> byUsername = accountRepository.findByUsername(username);
        Account account = byUsername.orElseThrow(() -> new UsernameNotFoundException(username));
        return new User(account.getUsername(), account.getPassword(), authorities());
    }

    private Collection<? extends GrantedAuthority> authorities() {
        return Arrays.asList(new SimpleGrantedAuthority("ROLE_USER"));
    }
}
```
- AccountRunner 생성
    * 로그인할 테스트 유저를 임의로 생성하기 위해 Runner 클래스를 생성.
```java
@Component
public class AccountRunner implements ApplicationRunner {

    @Autowired
    AccountService accountService;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Account hayoung = accountService.createAccount("hayoung", "1234");
        System.out.println(hayoung.getUsername() + " password: " + hayoung.getPassword());
    }
}
```
<br>

### PasswordEncoder 설정 및 사용
- 위의 로직을 다 구현하고 테스트 해보면 오류가 난다.(password 인코딩을 안했기 때문)
- 스프링 시큐리티 버전이 올라가면서 다양한 인코딩을 지원 하느라 password 포맷이 `{noop}password`여야 아무것도 인코딩하지 않는 패스워드 인코더로 디코딩을 시도한다.
- 아무것도 없는 경우 위와 같이 예외가 발생
- 실습 진행을 위해 사용하면 안될 방법 사용(그냥 보여만 주는 것)
    * 실제로는 사용하면 안되지만 예외적으로 회피할 수 있는 NoOpPasswordEncoder로 로직 처리
    * 스프링 시큐리티가 사용할 Encoder가 더이상 기본 패스워드가 아닌 NoOpPasswordEncoder 로 변환된다. 
    * 패스워드 앞에 있는 prefix 값을 보고 어떤 Encoding인지 확인 한 다음에 Encoding Decoding하는 똑똑한 패스워드가 아니라
    * 일부러 그 빈을 Encoding Decoding 아무것도 하지 않는 NoOp 그런 패스워드 Encoder로 바꾼 것이다. 
```java
@Bean
public PasswordEncoder passwordEncoder() {
    return NoOpPasswordEncoder.getInstance();
}
```
- 권장하는 PasswordEncoder
    * 스프링 시큐리티가 권장하는 PasswordEncoder으로 설정
```java
@Bean
public PasswordEncoder passwordEncoder() {
    return PasswordEncoderFactories.createDelegatingPasswordEncoder();
}
```
- SecurityConfig에 PasswordEncoder 빈으로 등록하도록 설정 추가
```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/", "/hello").permitAll()
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                .httpBasic();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }
}
```
- AccountService PasswordEncoder를 사용하도록 로직 수정
    * PasswordEncoder을 주입을 받아서 Encoding 된 password를 저장하도록 수정
```java
@Autowired
private PasswordEncoder passwordEncoder;

public Account createAccount(String username, String password) {
    Account account = new Account();
    account.setUsername(username);
    account.setPassword(passwordEncoder.encode(password));
    return accountRepository.save(account);
}
```
<br>