## 5. 스프링 시큐리티와 OAuth2.0으로 로그인 기능 구현하기

### 5-1. 구글 서비스와 OAuth2.0 사용
- build.gradle에 의존성 추가
```gradle
dependencies {
    // ...
    compile('org.springframework.boot:spring-boot-starter-security')
    compile('org.springframework.boot:spring-boot-starter-oauth2-client')
    // ...
}
```
- src/main/resources에 application-oauth.properties 파일 생성
```properties
spring.security.oauth2.client.registration.google.client-id={clientId}
spring.security.oauth2.client.registration.google.client-secret={clientSecret}
spring.security.oauth2.client.registration.google.scope=profile,email
```
- application.properties에 코드 추가
```properties
spring.profiles.include=oauth
```
<br>

### 5-2. 구글 로그인 연동하기
- User 클래스 생성
    * 사용자 정보를 담당할 도메인
    * `@Enumerated(EnumType.STRING)`
        - JPA로 이용해 데이터베이스로 저장할 때 Enum 값을 어떤 형태로 저장할지를 결정한다.
        - 기본적으로 int로 된 숫자가 저장된다.
        - 숫자로 저장되면 데이터베이스로 확인할 때 그 값이 무슨 코드를 의미하는 지 알 수가 없다.
        - 따라서 문자열(EnumType.STRING)로 저장될 수 있도록 선언한다.
```java
@Getter
@NoArgsConstructor
@Entity
public class User extends BaseTimeEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private String email;

    @Column(nullable = false)
    private String picture;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private Role role;

    @Builder
    public User(String name, String email, String picture, Role role) {
        this.name = name;
        this.email = email;
        this.picture = picture;
        this.role = role;
    }

    public User update(String name, String picture) {
        this.name = name;
        this.picture = picture;

        return this;
    }

    public String getRoleKey() {
        return this.role.getKey();
    }
}
```
- Enum 클래스 Role 생성
    * 각 사용자의 권한을 관리한 Enum 클래스
```java
@Getter
@RequiredArgsConstructor
public enum Role {

    GUEST("ROLE_GUEST", "손님"),
    USER("ROLE_USER", "일반 사용자");

    private final String key;
    private final String title;
}
```
- UserRepository 생성
    * User의 CRUD를 담당한다.
    * `findByEmail()`
        - 소셜 로그인으로 반환되는 값 중 email을 통해 이미 생성된 사용자인지 처음 가입하는 사용자인지 판단하기 위한 메소드
```java
public interface UserRepository extends JpaRepository<User, Long> {

    Optional<User> findByEmail(String email);
}
```
- SecurityConfig 클래스 생성
    * `@EnableWebSecurity`
        - Spring Security 설정들을 활성화시켜 준다.
    * `authorizeRequests()`
        - URL별 권한 관리를 설정하는 옵션의 시작점이다.
        - `authorizeRequests()`가 선언되어야 `antMatchers()` 옵션 사용 가능하다.
    * `antMatchers`
        - 권한 관리 대상을 지정하는 옵션이다.
        - URL, HTTP 메소드별로 관리가 가능하다.
        - "/"등 지정된 URL들은 `permitAll()` 옵션을 통해 전체 열람 권한을 준다.
    * `anyRequest()`
        - 설정된 값들 이외 나머지 URL들을 나타낸다.
        - `authenticated()`를 추가해 나머지 URL들은 모두 인증된 사용자들에게만 허용하도록 한다.
    * `logout().logoutSuccessUrl("/")`
        - 로그아웃 기능에 대한 여러 설정의 진입점이다.
        - 로그아웃 성공 시 "/" 주소로 이동한다.
    * `oauth2Login()`
        - OAuth2 로그인 기능에 대한 여러 설정의 진입점이다.
    * `userInfoEndpoint()`
        - OAuth2 로그인 성공 이후 사용자 정보를 가져올 때의 설정들을 담당한다. 
    * `userService()`
        - 소셜 로그인 성공 시 후속 조치를 진행할 UserService 인터페이스의 구현체를 등록한다.
        - 리소스 서버(즉, 소셜 서비스들)에서 사용자 정보를 가져온 상태에서 추가로 진행하고자 하는 기능을 명시할 수 있다.
```java
@RequiredArgsConstructor
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private  final CustomOAuth2UserService customOAuth2UserService;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        CharacterEncodingFilter filter = new CharacterEncodingFilter();
        http
                .csrf().disable() // h2-console 화면을 사용하기 위해 해당 옵션을 disable
                .headers().frameOptions().disable()
                .and()
                    .authorizeRequests()
                    .antMatchers("/", "/css/**", "/images/**", "/js/**", "/h2-console/**").permitAll()
                    .antMatchers("/api/v1/**").hasRole(Role.USER.name())
                    .anyRequest().authenticated()
                .and()
                    .logout()
                        .logoutSuccessUrl("/")
                .and()
                    .oauth2Login()
                        .userInfoEndpoint()
                            .userService(customOAuth2UserService);
    }
}
```
- CustomOAuth2UserService 클래스 생성
    * 구글 로그인 이후 가져온 사용자의 정보들을 기반으로 가입 및 정보수정, 세션 저장 등의 기능을 지원한다.
    * `registrationId`
        - 현재 로그인 진행 중인 서비스를 구분하는 코드이다.
        - 네이버 로그인 연동 시에 네이버 로그인인지, 구글 로그인인지 구분하기 위해 사용한다.
    * `userNameAttributeName`
        - OAuth2 로그인 진행 시 키가 되는 필드값을 이야기한다.(Primary Key와 같은 의미)
        - 구글의 경우 기본적으로 코드를 지원하지만, 네이버 카카오 등은 기본 지원하지 않는다.
        - 이후 네이버 로그인과 구글 로그인을 동시 지원할 때 사용한다.
    * `OAuthAttributes`
        - OAuth2UserService를 통해 가져온 OAuth2User의 attribute를 담을 클래스
        - 네이버 등 다른 소셜 로그인도 이 클래스를 사용한다.
    * `SessionUser`
        - 세션에 사용자 정보를 저장하기 위한 Dto 클래스
```java
@RequiredArgsConstructor
@Service
public class CustomOAuth2UserService implements OAuth2UserService<OAuth2UserRequest, OAuth2User> {
    private final UserRepository userRepository;
    private final HttpSession httpSession;

    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
        OAuth2UserService delegate = new DefaultOAuth2UserService();
        OAuth2User oAuth2User = delegate.loadUser(userRequest);

        String registrationId = userRequest.getClientRegistration().getRegistrationId();
        String userNameAttributeName = userRequest.getClientRegistration().getProviderDetails()
                .getUserInfoEndpoint().getUserNameAttributeName();

        OAuthAttributes attributes = OAuthAttributes.of(registrationId, userNameAttributeName, oAuth2User.getAttributes());

        User user = saveOrUpdate(attributes);
        httpSession.setAttribute("user", new SessionUser(user));

        return new DefaultOAuth2User(
                Collections.singleton(new SimpleGrantedAuthority(user.getRoleKey())),
                attributes.getAttributes(),
                attributes.getNameAttributeKey());
    }


    private User saveOrUpdate(OAuthAttributes attributes) {
        User user = userRepository.findByEmail(attributes.getEmail())
                .map(entity -> entity.update(attributes.getName(), attributes.getPicture()))
                .orElse(attributes.toEntity());

        return userRepository.save(user);
    }
} 
```
- OAuthAttributes 클래스 생성
    * `of()`
        - OAuth2User에서 반환하는 사용자 정보는 Map이기 때문에 값 하나하나를 변환해야 한다.
    * `toEntity()`
        - User 엔티티를 생성한다.
        - OAuthAttributes에서 엔티티를 생성하는 시점은 처음 가입할 때이다.
```java
@Getter
public class OAuthAttributes {
    private Map<String, Object> attributes;
    private String nameAttributeKey;
    private String name;
    private String email;
    private String picture;

    @Builder
    public OAuthAttributes(Map<String, Object> attributes, String nameAttributeKey, String name, String email, String picture) {
        this.attributes = attributes;
        this.nameAttributeKey = nameAttributeKey;
        this.name = name;
        this.email = email;
        this.picture = picture;
    }

    public static OAuthAttributes of(String registrationId, String userNameAttributeName, Map<String, Object> attributes) {
        return ofGoogle(userNameAttributeName, attributes);
    }

    private static OAuthAttributes ofGoogle(String userNameAttributeName, Map<String, Object> attributes) {
        return OAuthAttributes.builder()
                .name((String) attributes.get("name"))
                .email((String) attributes.get("email"))
                .picture((String) attributes.get("picture"))
                .attributes(attributes)
                .nameAttributeKey(userNameAttributeName)
                .build();
    }

    public User toEntity() {
        return User.builder()
                .name(name)
                .email(email)
                .picture(picture)
                .role(Role.GUEST) // 가입할 때 기본 권한을 GUEST
                .build();
    }
} 
```
- SessionUser 클래스 생성
```java
@Getter
public class SessionUser {
    private String name;
    private String email;
    private String picture;

    public SessionUser(User user) {
        this.name = user.getName();
        this.email = user.getEmail();
        this.picture = user.getPicture();
    }
}
```
<br>

#### 5-2-1. 로그인 테스트
- index.mustache에 로그인 버튼 추가
    * `{{#userName}}`
        - mustache는 다른 언어와 같은 if문을 제공하지 않는다. (true / false 여부만 판단한다.)_
        - mustache에 항상 최종 값을 넘겨줘야 한다.
        - userName이 있다면 userName을 노출하도록 구성한다.
    * `a href="/logout"`
        - 스프링 시큐리티에서 기본적으 제공하는 로그아웃 URL이다.
        - 개발자가 별도로 URL에 해당하는 컨트롤러를 만들 필요가 없다.
        - SecurityConfig 클래스에서 URL을 변경할 순 있지만 기본 URL을 사용해도 충분하다.
    * `{{^userName}}`
        - mustache에서 해당 값이 존재하지 않는 경우 ^를 사용
        - userName이 없다면 로그인 버튼을 노출하도록 구성
    * `a href="/oauth2/authorization/google"`
        - 스프링 시큐리티에서 기본적으로 제공하는 로그인 URL
        - 로그아웃 URL과 마찬가지로 개발자가 별도의 컨트롤러를 생성할 필요가 없다.
```mustache
{{>layout/header}}

    <h1>스프링 부트로 시작하는 웹 서비스</h1>
    <div class="col-md-12">
        <div class="row">
            <div class="col-md-6">
                <a href="/posts/save" role="button" class="btn btn-primary">글 등록</a>
                {{#userName}}
                    Logged in as: <span id="user">{{userName}}</span>
                    <a href="/logout" class="btn btn-info active" role="button">Logout</a>
                {{/userName}}
                {{^userName}}
                    <a href="/oauth2/authorization/google" class="btn btn-success active" role="button">Google Login</a>
                {{/userName}}
            </div>
        </div>

        <!-- 목록 출력 영역-->
        // ...
```
- IndexController에 userName을 model에 저장하는 코드 추가
```java
@RequiredArgsConstructor
@Controller
public class IndexController {

    private final PostsService postsService;
    private final HttpSession httpSession;

    @GetMapping("/")
    public String index(Model model) {
        model.addAttribute("posts", postsService.findAllDesc());
        // 로그인 성공 시 값을 가져온다.
        SessionUser user = (SessionUser) httpSession.getAttribute("user");

        if(user != null) { // 세션에 저장된 값이 있을 경우
            // 세션에 저장된 값이 있으면 model엔 아무런 값이 없는 상태이므로 로그인 버튼이 보이게 된다.
            model.addAttribute("userName", user.getName());
        }
        return "index";
    }
}
```
<br>

### 5-3. 어노테이션 기반으로 개선하기
- IndexCotroller에서 세션값을 가져오는 부분 애노테이션 기반으로 개선하기
- @LoginUser 애노테이션 생성
    * `@Target(ElementType.PARAMETER)`
        - 어노테이션이 생성될 수 있는 위치를 지정한다.
        - PARAMETER로 지정했으므로 메서소의 파라미터로 선언된 객체에서만 사용될 수 있다.
        - 이 외에도 클래스 선언시 사용할 수 있는 TYPE 등이 있다.
```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface LoginUser {

}
```
- LoginUserArgumentResolver 생성
    * `supportParameter()`
        - 컨트롤러 메소드의 특정 파라미터를 지원하는지 판단한다.
        - 파라미터에 @LoginUser 어노테이션이 붙어있고, 파라미터 클래스 타입이 SessionUser.class인 경우 true를 반환
    * `resolveArgument()`
        - 파라미터에 전달할 객체를 생성한다.
        - 아래 코드에서는 세션에서 객체를 가져온다.
```java
@RequiredArgsConstructor
@Component
public class LoginUserArgumentResolver implements HandlerMethodArgumentResolver {

    private final HttpSession httpSession;

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        boolean isLoginUserAnnotation = parameter.getParameterAnnotation(LoginUser.class) != null;
        boolean isUserClass = SessionUser.class.equals(parameter.getParameterType());

        return isLoginUserAnnotation && isUserClass;
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        return httpSession.getAttribute("user");
    }
}
```
- WebConfig 클래스 생성
    * LoginUserArgumentResolver를 스프링에서 인식할 수 있게 해준다.
```java
@RequiredArgsConstructor
@Configuration
public class WebConfig implements WebMvcConfigurer {
    private final LoginUserArgumentResolver loginUserArgumentResolver;

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
        argumentResolvers.add(loginUserArgumentResolver);
    }
}
```
- IndexController의 반복되는 부분 `@LoginUser` 애노테이션으로 개선
    * 이제 어느 컨트롤러든지 `@LoginUser` 애노테이션만 사용하면 세션 정보를 가져올 수 있다.
```java
@RequiredArgsConstructor
@Controller
public class IndexController {

    private final PostsService postsService;
    private final HttpSession httpSession;

    @GetMapping("/")

    public String index(Model model, @LoginUser SessionUser user) {
        model.addAttribute("posts", postsService.findAllDesc());

        //SessionUser user = (SessionUser) httpSession.getAttribute("user");
        if(user != null) {
            model.addAttribute("userName", user.getName());
        }
        return "index";
    }

    // ...
}
```
<br>

### 5-4. 네이버 로그인 연동하기
- application-oauth.properties에 키 값들을 등록
    * 네이버에서는 스프링 시큐리티를 공식 지원하지 않기 때문에 수동으로 입력해야 한다.
```properties
#registration

spring.security.oauth2.client.registration.naver.client-id=clientID값
spring.security.oauth2.client.registration.naver.client-secret=clientSecret값
spring.security.oauth2.client.registration.naver.redirect-uri={baseUrl}/{action}/oauth2/code/{registrationId}
spring.security.oauth2.client.registration.naver.authorization-grant-type=authorization_code
spring.security.oauth2.client.registration.naver.scope=name,email,profile_image
spring.security.oauth2.client.registration.naver.client-name=Naver

#provider
spring.security.oauth2.client.provider.naver.authorization-uri=https://nid.naver.com/oauth2.0/authorize
spring.security.oauth2.client.provider.naver.token-uri=https://nid.naver.com/oauth2.0/token
spring.security.oauth2.client.provider.naver.user-info-uri=https://openapi.naver.com/v1/nid/me
spring.security.oauth2.client.provider.naver.user-name-attribute=response
```
- OAuthAttributes에 네이버를 판단하는 코드와 네이버 생성자 코드 추가
```java
@Getter
public class OAuthAttributes {
    // ...

    public static OAuthAttributes of(String registrationId, String userNameAttributeName, Map<String, Object> attributes) {
        if("naver".equals(registrationId)) {
            return ofNaver("id", attributes);
        }

        return ofGoogle(userNameAttributeName, attributes);
    }

    // ...

    private static OAuthAttributes ofNaver(String userNameAttributeName, Map<String, Object> attributes) {
        Map<String, Object> response = (Map<String, Object>) attributes.get("response");

        return OAuthAttributes.builder()
                .name((String) response.get("name"))
                .email((String) response.get("email"))
                .picture((String) response.get("profile_image"))
                .attributes(response)
                .nameAttributeKey(userNameAttributeName)
                .build();
    }
}
```
- index.mustache에 네이버 로그인 버튼 추가
```mustache
{{>layout/header}}
    <h1>스프링 부트로 시작하는 웹 서비스</h1>
    <div class="col-md-12">
        <div class="row">
            <div class="col-md-6">
                <a href="/posts/save" role="button" class="btn btn-primary">글 등록</a>
                {{#userName}}
                    Logged in as: <span id="user">{{userName}}</span>
                    <a href="/logout" class="btn btn-info active" role="button">Logout</a>
                {{/userName}}
                {{^userName}}
                    <a href="/oauth2/authorization/google" class="btn btn-success active" role="button">Google Login</a>
                    <a href="/oauth2/authorization/naver" class="btn btn-secondary active" role="button">Naver Login</a> // 네이버 로그인 버튼 추가
                {{/userName}}
            </div>
        </div>
        <!-- 목록 출력 영역-->

        // ...
```
<br>

### 5-5. 기존 테스트에 시큐리티 적용하기
- 지금 상태에서 전체 테스트를 수행하면 롬복을 이용한 테스트 외에 스프링을 이용한 테스트는 모두 실패한다.

#### 5-5-1. CustomOAuth2UserService를 찾을 수 없음
- CustomOAuth2UserService를 생성하는데 필요한 소셜 로그인 관련 설정값들이 없기 때문에 발생(src/main과 src/test 환경의 차이)
- test 환경을 위한 application.properties 파일을 만들고 가짜 설정값을 등록한다.
```properties
spring.jpa.show_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.h2.console.enabled=true
spring.session.store-type=jdbc

# Test OAuth

spring.security.oauth2.client.registration.google.client-id=test
spring.security.oauth2.client.registration.google.client-secret=test
spring.security.oauth2.client.registration.google.scope=profile,email
```
<br>

#### 5-5-2. 302 Status Code
- 스프링 시큐리티 설정 때문에 인증되지 않은 사용자의 요청은 이동시켜 302 에러 코드가 뜬다.
- 임의로 인증된 사용자를 추가하여 API만 테스트하게 하면 해결 가능하다.
- build.gradle에 spring-security-test 추가
 ```gradle
 compile('org.springframework.security:spring-security-test')
 ```
 - PostsApiControllerTest에 임의의 사용자 인증 추가
 ```java
 @RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class PostsApiControllerTest {
    // ...

    @Autowired
    private WebApplicationContext context;

    private MockMvc mvc;

    @Before
    public void setup() {
        mvc = MockMvcBuilders
                .webAppContextSetup(context)
                .apply(springSecurity())
                .build();
    }

    // ...

    @Test
    @WithMockUser(roles="USER")
    public void Posts_동록된다() throws Exception {
        // ...

        //when
        mvc.perform(post(url)
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .content(new ObjectMapper().writeValueAsString(requestDto)))
                .andExpect(status().isOk());
    }

    // ...

    @Test
    @WithMockUser(roles="USER")
    public void Posts_수정된다() throws Exception {
        // ...

        //when
        mvc.perform(put(url)
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .content(new ObjectMapper().writeValueAsString(requestDto)))
                .andExpect(status().isOk());
    }
    
    // ..
}
 ```
 <br>

#### 5-5-3. @WebMvcTest에서 CustomOAuth2UserService를 찾을 수 없음
- HelloControllerTest에서 `@WebMvcTest`를 사용했기 때문에 5-5-1로 인해 스프링 시큐리티 설정은 작동하지만 CustomOAuth2UserService를 스캔하지 않는다.
    * `@WebMvcTest`는 web과 관련된 것만 스캔한다.
- 따라서 다음과 같이 스캔 대상에서 SecurityConfig를 제거한다.
- 또한 마찬가지로 @WithMockUser를 사용해 가짜로 인증된 사용자를 생성한다.
```java
@RunWith(SpringRunner.class)
@WebMvcTest
@WebMvcTest(controllers = HelloController.class,
        excludeFilters = {
                @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = SecurityConfig.class)
        }
)
public class HelloControllerTest {
    // ...

    @WithMockUser(roles="USER")
    @Test
    public void hello가_리턴된다() throws Exception {
        
        //...
    }

    // ...

    @WithMockUser(roles="USER")
    @Test
    public void helloDto가_리턴된다() throws Exception {
        
        // ...
    }
}
```
<br>

#### 5-5-4. @EnableJpaAuditing 관련 에러
- 에러 코드 : `java.lang.IllegalArgumentException: At least one JPA metamodel must be present!`
- `@EnableJpaAuditing`를 사용하기 위해서는 최소 하나의 @Entity 클래스가 필요하지만 @WebMvcTest이기 떄문에 없다.
- `@EnableJpaAuditing`와 `@SpringBootApplication`이 함께 있기 때문에 `@WebMvcTest`에서도 스캔하므로 이 둘을 분리 시켜야 한다.
- Application.java에서 `@EnableJpaAuditing`을 제거한다.
```java
// @EnableJpaAuditing
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
- JpaConfig를 생성하여 `@EnableJpaAuditing`를 추가한다.
```java
@Configuration
@EnableJpaAuditing
public class JpaConfig {

}
```