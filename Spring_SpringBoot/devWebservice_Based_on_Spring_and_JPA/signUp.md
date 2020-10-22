# 회원가입과 로그인

## 계정 관리 기능 미리보기
- 회원 가입
- 이메일 인증
- 로그인
- 로그아웃
- 프로필 추가 정보 입력
- 프로필 이미지 등록
- 알림 설정
- 패스워드 수정
- 패스워드를 잊어버린 경우
- 관심 주제(태그) 등록
- 주요 활동 지역 등록
<br>

## 계정 도메인
- domain 패키지에 Account 도메인 생성
    * `@EqualsAndHashCode(of = "id")`
        - 연관관계가 복잡해지면 `equals()`, `hashcode()`에서 서로 다른 연관관계를 계속해서 순환 참조하느라 무한 루프가 발생하고 결국에는 StackOverFlow가 발생할 수 있다.
        - 따라서 id만 사용 
    * email과 nickname은 중복 되면 안된다.(unique)
    * `@Lob @Basic(fetch = FetchType.EAGER)`
        - 프로필 이미지는 VARCHAR(255)보다 커질 가능성이 있고
        - 유저를 로딩할 때 종종 같이 쓰이므로 바로바로 가져올 수 있도록 즉시 로딩 사용
```java
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
@Builder @AllArgsConstructor @NoArgsConstructor
public class Account {

    @Id @GeneratedValue
    private Long id;

    @Column(unique = true)
    private String email;

    @Column(unique = true)
    private String nickname;

    private String password;

    private boolean emailVerified; // 이메일 인증 절차 유무를 판단할 flag

    private String emailCheckToken; // 이메일을 검증할 때 사용할 토큰 값

    private LocalDateTime joinedAt; // 인증을 거친 사용자들의 가입 날짜

    private String bio; // 간단한 자기소개

    private String url; // 웹사이트 url

    private String occupation; // 직업

    private String location; // 거주 지역

    @Lob @Basic(fetch = FetchType.EAGER)
    private String profileImage; // 프로필 이미지

    private boolean studyCreatedByEmail; // 스터디가 생성되면 이메일로 받을 것인지

    private boolean studyCreatedByWeb; // 스터디가 생성되면 웹으로 받을 것인지

    private boolean studyEnrollmentResultByEmail; // 스터디 가입 신청 결과를 이메일로 받을 것인지

    private boolean studyEnrollmentResultByWeb; // 스터디 가입 신청 결과를 웹으로 받을 것인지

    private boolean studyUpdatedByEmail; // 스터디 변경 정보를 이메일로 받을 것인지

    private boolean studyUpdatedByWeb; // 스터디 변경 정보를 웹으로 받을 것인지

}
```
<br>

## 회원가입 컨트롤러

### 목표
- GET “/sign-up” 요청을 받아서 account/sign-up.html 페이지 보여준다.
- 회원 가입 폼에서 입력 받을 수 있는 정보를 “닉네임", “이메일", “패스워드" 폼 객체로 제공한다
<br>

### 구현
- account 패키지 생성 후 AccountController 생성
```java
@Controller
public class AccountController {

    @GetMapping("/sign-up")
    public String signUpForm(Model model) {
        return "account/sign-up";
    }
}
```
- 실행하면 다음과 같이 login 화면이 나온다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/signUp_1.jpg"></p>

- 매번 localhost:8080에 접속할 때마다 login으로 redirect되기 때문에 Spring Security 설정 추가
    * config 패키지를 만들고 SecurityConfig 추가
    * `@EnableWebSecurity`
        - Spring Security 설정을 직접 한다.
    * 설정한 특정 요청은 Security 인증하지 않도록 설정하고 나머지는 로그인 해야만 하도록 설정
    * profile은 GET만 허용
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .mvcMatchers("/", "/login", "/sign-up", "/check-email", "check-email-token",
                        "/email-login", "/check-email-login", "/login-link").permitAll()
                .mvcMatchers(HttpMethod.GET, "profile/*").permitAll()
                .anyRequest().authenticated();
    }
}
```
- 실행하면 로그인 화면(/login)으로 가지 않는 것을 확인할 수 있다.
- 테스트 코드 작성
```java
@SpringBootTest
@AutoConfigureMockMvc
class AccountControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @DisplayName("회원 가입 화면이 보이는지 테스트")
    @Test
    void signUpForm() throws Exception {
        mockMvc.perform(get("/sign-up"))
                .andExpect(status().isOk())
                .andExpect(view().name("account/sign-up"));
    }

}
```
<br>

## 회원가입 뷰

### 목표
- Bootstrap
    * 네이게이션 바 만들기
    * 폼 만들기
- Thymeleaf
    * SignUpForm 타입 객체를 폼 객체로 설정하기
- 웹(HTML, CSS, JavaScript)
    * 제약 검증 기능 사용하기
        - 닉네임 (3~20자, 필수 입력)
        - 이메일 (이메일 형식, 필수 입력)
        - 패스워드 (8~50자, 필수 입력)
<br>

### 구현
- 회원가입 뷰 작성
    * 참고로 뷰에서 사용하고 있는 icon 이미지 파일을 넣어줘야 한다.(templates/static/images)
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>StudyOlle</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.5.3/dist/css/bootstrap.min.css" integrity="sha384-TX8t27EcRE3e/ihU7zmQxVncDAy5uIKz4rEkgIXeMed4M0jlfIDPvg6uqKI2xXr2" crossorigin="anonymous">
    <style>
        .container {
            max-width: 100%;
        }
    </style>
</head>

<body class="bg-light">
    <nav class="navbar navbar-expand-sm navbar-dark bg-dark">
        <a class="navbar-brand" href="/" th:href="@{/}">
            <img src="/images/logo_sm.png" width="30" height="30">
        </a>
        <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
            <span class="navbar-toggler-icon"></span>
        </button>

        <div class="collapse navbar-collapse" id="navbarSupportedContent">
            <ul class="navbar-nav mr-auto">
                <li class="nav-item">
                    <form th:action="@{/search/study}" class="form-inline" method="get">
                        <input class="form-control mr-sm-2" name="keyword" type="search" placeholder="스터디 찾기" aria-label="Search" />
                    </form>
                </li>
            </ul>

            <ul class="navbar-nav justify-content-end">
                <li class="nav-item">
                    <a class="nav-link" href="#" th:href="@{/login}">로그인</a>
                </li>
                <li class="nav-item">
                    <a class="nav-link" href="#" th:href="@{/signup}">가입</a>
                </li>
            </ul>
        </div>
    </nav>

    <div class="container">
        <div class="py-5 text-center">
            <h2>계정 만들기</h2>
        </div>
        <div class="row justify-content-center">
            <form class="needs-validation col-sm-6" action="#"
                  th:action="@{/signup}" th:object="${signUpForm}" method="post" novalidate>
                <div class="form-group">
                    <label for="nickname">닉네임</label>
                    <input id="nickname" type="text" th:field="*{nickname}" class="form-control"
                           placeholder="닉네임" aria-describedby="nicknameHelp" required minlength="3" maxlength="20">
                    <small id="nicknameHelp" class="form-text text-muted">
                        공백없이 문자와 숫자로만 3자 이상 20자 이내로 입력하세요. 가입후에 변경할 수 있습니다.
                    </small>
                    <small class="invalid-feedback">닉네임 형식이 올바르지 않습니다.</small>
                    <small class="form-text text-danger" th:if="${#fields.hasErrors('nickname')}" th:errors="*{nickname}">Nickname Error</small>
                </div>

                <div class="form-group">
                    <label for="email">이메일</label>
                    <input id="email" type="email" th:field="*{email}" class="form-control"
                           placeholder="example@email.com" aria-describedby="emailHelp" required>
                    <small id="emailHelp" class="form-text text-muted">
                        스터디올래는 사용자의 이메일을 공개하지 않습니다.
                    </small>
                    <small class="invalid-feedback">이메일 형식이 올바르지 않습니다.</small>
                    <small class="form-text text-danger" th:if="${#fields.hasErrors('email')}" th:errors="*{email}">Email Error</small>
                </div>

                <div class="form-group">
                    <label for="password">패스워드</label>
                    <input id="password" type="password" th:field="*{password}" class="form-control"
                           aria-describedby="passwordHelp" required minlength="8" maxlength="50">
                    <small id="passwordHelp" class="form-text text-muted">
                        8자 이상 50자 이내로 입력하세요. 영문자, 숫자, 특수기호를 사용할 수 있으며 공백은 사용할 수 없습니다.
                    </small>
                    <small class="invalid-feedback">패스워드 형식이 올바르지 않습니다.</small>
                    <small class="form-text text-danger" th:if="${#fields.hasErrors('password')}" th:errors="*{password}">Password Error</small>
                </div>

                <div class="form-group">
                    <button class="btn btn-primary btn-block" type="submit"
                            aria-describedby="submitHelp">가입하기</button>
                    <small id="submitHelp" class="form-text text-muted">
                        <a href="#">약관</a>에 동의하시면 가입하기 버튼을 클릭하세요.
                    </small>
                </div>
            </form>
        </div>

        <footer th:fragment="footer">
            <div class="row justify-content-center">
                <img class="mb-2" src="/images/logo_long_kr.png" alt="" width="100">
                <small class="d-block mb-3 text-muted">&copy; 2020</small>
            </div>
        </footer>
    </div>
    <script src="https://code.jquery.com/jquery-3.5.1.slim.min.js" integrity="sha384-DfXdz2htPH0lsSSs5nCTpuj/zy4C+OGpamoFVy38MVBnE+IbbVYUew+OrCXaRkfj" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.1/dist/umd/popper.min.js" integrity="sha384-9/reFTGAW83EW2RDu2S0VKaIzap3H66lZH81PoYlFhbGU+6BZp6G7niu735Sk7lN" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@4.5.3/dist/js/bootstrap.min.js" integrity="sha384-w1Q4orYjBQndcko6MimVbzY0tgp4pWB4lZ7lr30WKz0vr/aWKhXdBNmNb5D92v7s" crossorigin="anonymous"></script>
    <script type="application/javascript">
        (function () {
            'use strict';

            window.addEventListener('load', function () {
                // Fetch all the forms we want to apply custom Bootstrap validation styles to
                var forms = document.getElementsByClassName('needs-validation');

                // Loop over them and prevent submission
                Array.prototype.filter.call(forms, function (form) {
                    form.addEventListener('submit', function (event) {
                        if (form.checkValidity() === false) {
                            event.preventDefault();
                            event.stopPropagation();
                        }
                        form.classList.add('was-validated')
                    }, false)
                })
            }, false)
        }())
    </script>
</body>
</html>
```
- 회원가입 폼 추가
```java
@Data
public class SignUpForm {

    private String nickname;
    private String email;
    private String password;

}
```
- 컨트롤러에 model 추가
```java
@Controller
public class AccountController {

    @GetMapping("/sign-up")
    public String signUpForm(Model model) {
        // model.addAttribute("signUpForm", new SignUpForm());
        // 생략하면 클래스 이름의 CamelCase를 사용한다.
        model.addAttribute(new SignUpForm());
        return "account/sign-up";
    }
}
```
- 실행하면 이미지 파일들이 Spring Security에 걸려서 허용되지 않아 403 에러가 나온다.
    * 따라서 static resource들은 인증하지 않고 제공하기 위해 설정한다.
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    ...

    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring()
                .requestMatchers(PathRequest.toStaticResources().atCommonLocations());
    }
}
```
- 테스트 코드 작성
```java
@SpringBootTest
@AutoConfigureMockMvc
class AccountControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @DisplayName("회원 가입 화면이 보이는지 테스트")
    @Test
    void signUpForm() throws Exception {
        mockMvc.perform(get("/sign-up"))
                .andExpect(status().isOk())
                .andExpect(view().name("account/sign-up"))
                .andExpect(model().attributeExists("signUpForm"));

    }
}
```
- 실행하면 다음과 같이 뷰가 잘 만들어진 것을 볼 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/signUp_2.jpg"></p>

<br>

## 회원가입 폼 서브밋 검증

### 목표
- JSR 303 애노테이션 검증
    * 값의 길이, 필수값
- 커스텀 검증
    * 중복 이메일, 닉네임 여부 확인
- 폼 에러 있을 시, 폼 다시 보여주기

### 구현
- **JSR 303 애노테이션 검증하기**
- 폼 서브밋 요청에 대한 Post맵핑 추가
```java
@Controller
public class AccountController {

    ...

    @PostMapping("/sign-up")
    public String signUpSubmit(@Valid SignUpForm signUpForm, Errors errors) {
        if(errors.hasErrors()) {
            return "account/sign-up";
        }
        
        // TODO 회원 가입 처리
        return "redirect:/";

    }
}
```
- 폼 객체에 Validation 애노테이션 추가
```java
@Data
public class SignUpForm {

    @NotBlank
    @Length(min = 3, max = 20)
    @Pattern(regexp = "^[ㄱ-ㅎ가-힣a-z0-9_-]{3,20}$")
    private String nickname;

    @Email
    @NotBlank
    private String email;

    @NotBlank
    @Length(min = 8, max = 50)
    private String password;

}
```
- 실행하면 잘 되지만 다음과 같이 패턴에 어긋나는 값을 넣을 경우 경고 메세지가 나온다.
    * 프론트에서 검증 코드를 넣었지만 뚤린다.
        - 의미가 없진 않다.
        - 서버에 오기 전에 걸러주면 리소스를 절약할 수 있고
        - 사용자에게 빠르게 fail-fast해줄 수 있다.
    * **프론트에서도 검증하고 백엔드에서도 검증을 해야하는 이유다.**
- **커스텀 검증하기**
- SignUpFormValidator 생성
    * `@Component`
        - AccountRepository을 주입받아 사용하려면 bean으로 등록되야 한다.
```java
@Component // AccountRepository을 주입받아 사용하려면 bean으로 등록되야 한다.
@RequiredArgsConstructor
public class SignUpFormValidator implements Validator {

    private final AccountRepository accountRepository;

    @Override
    public boolean supports(Class<?> aClass) {
        return aClass.isAssignableFrom(SignUpForm.class); // 검증할 인스턴스의 타입
    }

    @Override
    public void validate(Object object, Errors errors) {
        SignUpForm signUpForm = (SignUpForm)object;

        if(accountRepository.existsByEmail(signUpForm.getEmail())) {
            errors.rejectValue("email", "invalid.email", new Object[]{signUpForm.getEmail()}, "이미 사용중인 이메일입니다.");
        }

        if(accountRepository.existsByNickname(signUpForm.getNickname())) {
            errors.rejectValue("nickname", "invalid.nickname", new Object[]{signUpForm.getNickname()}, "이미 사용중인 닉네임입니다.");
        }
    }
}
```
- AccountRepository 생성
```java
@Transactional(readOnly = true) // 읽기전용으로 만들어 성능에 조금이라도 이점을 가져온다. 
public interface AccountRepository extends JpaRepository<Account, Long> {
    
    boolean existsByEmail(String email);
    boolean existsByNickname(String nickname);
}
```
- 컨트롤러에 커스텀 검증 사용
    * `@InitBinder("signUpForm)`
        - signUpForm이라는 데이터를 받을 때 바인더를 설정한다.
        - WebDataBinder라는 것을 파라미터로 받고 Validator를 추가할 수 있다.
        - 추가하면 signUpForm을 받을 때 받은 파라미터의 camelCase에 맞춰 validator도 사용이 된다.
```java
@Controller
@RequiredArgsConstructor
public class AccountController {

    private final SignUpFormValidator signUpFormValidator;

    @InitBinder("signUpForm")
    public void initBinder(WebDataBinder webDataBinder) {
        webDataBinder.addValidators(signUpFormValidator);
    }

    ...
}
```
<br>

## 회원 가입 폼 서브밋 처리

### 목표
- 회원 가입 처리
    * 회원 정보 저장
    * 인증 이메일 발송
    * 처리 후 첫 페이지로 리다이렉트([Post-Redirect-Get 패턴](https://en.wikipedia.org/wiki/Post/Redirect/Get))

### 구현
- 이메일은 MailSenderAutoConfiguration에 이미 정의 되어있고 MailSender를 주입받아 사용할 수 있지만  아직은 가짜 객체를 만들어서 콘솔에 출력하도록 한다.
    * 실제 객체로는 나중에 구현
```java
@Profile("local")
@Component
@Slf4j
public class ConsoleMailSender implements JavaMailSender {
    @Override
    public MimeMessage createMimeMessage() {
        return null;
    }

    @Override
    public MimeMessage createMimeMessage(InputStream inputStream) throws MailException {
        return null;
    }

    @Override
    public void send(MimeMessage mimeMessage) throws MailException {

    }

    @Override
    public void send(MimeMessage... mimeMessages) throws MailException {

    }

    @Override
    public void send(MimeMessagePreparator mimeMessagePreparator) throws MailException {

    }

    @Override
    public void send(MimeMessagePreparator... mimeMessagePreparators) throws MailException {

    }

    @Override
    public void send(SimpleMailMessage simpleMailMessage) throws MailException {
        log.info(simpleMailMessage.getText());
    }

    @Override
    public void send(SimpleMailMessage... simpleMailMessages) throws MailException {

    }
}
```
- Profile을 local로 설정
    * application.properties
```properties
spring.profiles.active=local
```
- 회원 가입 처리
```java
@Controller
@RequiredArgsConstructor
public class AccountController {

    private final SignUpFormValidator signUpFormValidator;
    private final AccountRepository accountRepository;
    private final JavaMailSender javaMailSender;

    ...

    @PostMapping("/sign-up")
    public String signUpSubmit(@Valid SignUpForm signUpForm, Errors errors) {
        
        ...

        // 회원 가입 처리
        Account account = Account.builder()
                .email(signUpForm.getEmail())
                .nickname(signUpForm.getNickname())
                .password(signUpForm.getPassword()) // TODO encoding 해야한다.
                .studyCreatedByWeb(true)
                .studyEnrollmentResultByWeb(true)
                .studyUpdatedByWeb(true)
                .build();

        Account newAccount = accountRepository.save(account);

        newAccount.generateEmailCheckToken();
        SimpleMailMessage mailMessage = new SimpleMailMessage();
        mailMessage.setTo(newAccount.getEmail());
        mailMessage.setSubject("스터디올래, 회원 가입 인증"); // 제목
        mailMessage.setText("/check-email-token?token=" + newAccount.getEmailCheckToken() +
                "&email=" + newAccount.getEmail()); // 본문
        javaMailSender.send(mailMessage);

        return "redirect:/";
    }
}
```
- 실행하면 다음과 같이 토큰이 log에 찍힌 것을 볼 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/signUp_3.jpg"></p>

<br>

## 회원가입 리펙토링과 테스트
- 리팩토링 하기전에 테스트 코드를 먼저 작성하자.
    * 그래야 코드를 변경한 이후에 불안하지 않다.
    * 변경한 코드가 무언가를 깨트리지 않았다는 것을 확인할 수 있다.
<br>

### 목표
- 테스트
    * 폼에 이상한 값이 들어간 경우에 다시 폼이 보여지는지
    * 폼에 값이 정상적인 경우 
        - 가입한 회원 데이터가 존재하는지
        - 이메일이 보내지는지
- 리팩토링
    * 메소드의 길이
    * 코드의 가독성
        - 내가 작성한 코드를 내가 읽기 어렵다면 남들에겐 훨씬 더 어렵다.
    * 코드가 적절한 위치
        - 객체들 사이의 의존 관계
        - 책임이 너무 많진 않은지
<br>

### 구현
- **테스트**
```java
@SpringBootTest
@AutoConfigureMockMvc
class AccountControllerTest {

    @Autowired
    private MockMvc mockMvc;

    ...

    @DisplayName("회원 가입 처리 - 입력값 오류")
    @Test
    void signUpSubmit_with_wrong_input() throws Exception {
        mockMvc.perform(post("/sign-up")
        .param("nickname", "hayoung")
        .param("email", "email..")
        .param("password", "12345"))
                .andExpect(status().isOk())
                .andExpect(view().name("account/sign-up"));
    }
}
```
- 실행해보면 403에러가 난다.
- 이유 : csrf 토큰이 없기 때문
    * Spring Security를 적용하면 기본적으로 csrf 설정이 활성화 되어있다.
    * csrf 토큰이라는 기술을 자동으로 사용한다.
    * 폼 데이터와 함께 토큰이 서버가 같이 전송되어 토큰을 보고 안전한 데이터인지를 판단한다.
    * SecurityConfig에서 인증이 필요없도록 설정했지만 그렇다고 안전하지 않은 요청을 받아들이는 것은 아니다.
    * 참고) [crsf](https://ko.wikipedia.org/wiki/%EC%82%AC%EC%9D%B4%ED%8A%B8_%EA%B0%84_%EC%9A%94%EC%B2%AD_%EC%9C%84%EC%A1%B0)
- 따라서 다음과 같이 crsf 토큰을 추가해줘야 한다.
```java
@SpringBootTest
@AutoConfigureMockMvc
class AccountControllerTest {

    @Autowired
    private MockMvc mockMvc;

    ...

    @DisplayName("회원 가입 처리 - 입력값 오류")
    @Test
    void signUpSubmit_with_wrong_input() throws Exception {
        mockMvc.perform(post("/sign-up")
        .param("nickname", "hayoung")
        .param("email", "email..")
        .param("password", "12345")
        .with(csrf()))
                .andExpect(status().isOk())
                .andExpect(view().name("account/sign-up"));
    }
}
```
- 옳은 값을 넣었을 경우 테스트(이메일까지)
```java
@SpringBootTest
@AutoConfigureMockMvc
class AccountControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private AccountRepository accountRepository;

    @MockBean
    JavaMailSender javaMailSender;

    ...

    @DisplayName("회원 가입 처리 - 입력값 정상")
    @Test
    void signUpSubmit_with_correct_input() throws Exception {
        mockMvc.perform(post("/sign-up")
        .param("nickname", "hayoung")
        .param("email", "hayoung@email.com")
        .param("password", "12345789")
        .with(csrf()))
                .andExpect(status().is3xxRedirection())
                .andExpect(view().name("redirect:/"));

        assertTrue(accountRepository.existsByEmail("hayoung@email.com"));
        // 이메일을 보내는지 테스트
        then(javaMailSender).should().send(any(SimpleMailMessage.class));
    }
}
```
- **리펙토링**
- AccountController
    * Account 생성과 이메일 보내는 것을 빼서 Service로 만들기
    * 전의 코드와 비교해서 보면 쉽다.
```java
@Service
@RequiredArgsConstructor
public class AccountService {

    private final AccountRepository accountRepository;
    private final JavaMailSender javaMailSender;

    // Account 생성
    private Account saveNewAccount(@Valid SignUpForm signUpForm) {
        Account account = Account.builder()
                .email(signUpForm.getEmail())
                .nickname(signUpForm.getNickname())
                .password(signUpForm.getPassword()) // TODO encoding 해야한다.
                .studyCreatedByWeb(true)
                .studyEnrollmentResultByWeb(true)
                .studyUpdatedByWeb(true)
                .build();

        return accountRepository.save(account);
    }

    // 이메일 보내기
    private void sendSignUpConfirmEmail(Account newAccount) {
        SimpleMailMessage mailMessage = new SimpleMailMessage();
        mailMessage.setTo(newAccount.getEmail());
        mailMessage.setSubject("스터디 올래, 회원 가입 인증"); // 제목
        mailMessage.setText("/check-email-token?token=" + newAccount.getEmailCheckToken() +
                "&email=" + newAccount.getEmail()); // 본문
        javaMailSender.send(mailMessage);
    }

    // 회원가입
    public void processNewAccount(SignUpForm signUpForm) {
        Account newAccount = saveNewAccount(signUpForm);
        newAccount.generateEmailCheckToken();
        sendSignUpConfirmEmail(newAccount);
    }
}
```
```java
@Controller
@RequiredArgsConstructor
public class AccountController {

    private final SignUpFormValidator signUpFormValidator;
    private final AccountService accountService;


    // 커스텀 검증
    @InitBinder("signUpForm")
    public void initBinder(WebDataBinder webDataBinder) {
        webDataBinder.addValidators(signUpFormValidator);
    }

    @GetMapping("/sign-up")
    public String signUpForm(Model model) {
        model.addAttribute(new SignUpForm());
        return "account/sign-up";
    }

    @PostMapping("/sign-up")
    public String signUpSubmit(@Valid SignUpForm signUpForm, Errors errors) {
        // JSR 303 검증
        if(errors.hasErrors()) {
            return "account/sign-up";
        }
        // 회원 가입 처리
        accountService.processNewAccount(signUpForm);

        return "redirect:/";
    }
}
```
- 코드를 많이 고쳤음에도 이전에 작성한 테스트를 돌려보면 통과한다.
    * 안전한 변경이었다.
<br>
