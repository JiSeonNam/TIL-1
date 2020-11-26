# DB와 이메일 설정
- 지금까지 DB는 인메모리 데이터베이스를 썼고 기본적인 profile을 local로 사용했다.
- 이메일 관련 기능 또한 Profile이 local일때 ConsoleMailSender 사용해서 처리했었다.
- 이제 DB를 PostgreSQL을 사용하도록 바꾸고 
- JavaMailSender 또한 실제 이메일을 사용하도록 한다.
<br>

## PostgreSQL 설치 및 설정
- [PostgreSQL 설치](https://www.postgresql.org/download/)
- [psql 접속](https://www.postgresqltutorial.com/connect-to-postgresql-database/)
- DB와 유저(role) 만들고 유저에게 권한 할당하기
    * `create database testdb;`
    * `create user testuser with encrypted password 'testpass';`
    * `grant all privileges on database testdb to testuser;`
- application-dev.properties에 DB 정보 설정
    * dev Profile용 설정 파일
```properties
# dev 모드이므로 create-drop이 아닌 update를 사용한다.
spring.jpa.hibernate.ddl-auto=update

spring.datasource.url=jdbc:postgresql://localhost:5432/testdb
spring.datasource.username=testuser
spring.datasource.password=testpass
```
<br>

## SMTP 설정
- SMTP 서버와 연결해서 실제 이메일 전송
    * Gmail
- Gmail 서버에 App Passwords 발급받기
    * 2차 인증을 설정하고
    * 앱 비밀번호를 설정하면 된다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/db_email_setting_1.jpg"></p>

- application-dev.properties 파일에 설정
```properties
...

# SMTP
spring.mail.host=smtp.gmail.com
spring.mail.port=587
spring.mail.username=khy07181@gmail.com
spring.mail.password=hgiauoajiemdumcu
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.timeout=5000
spring.mail.properties.mail.smtp.starttls.enable=true
```
- 실행해서 가입하면 다음과 같이 실제 이메일로 인증 메일이 온다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/db_email_setting_2.jpg"></p>

- 그러나 Gmail SMTP는 하루에 보낼 수 있는 이메일이 한정되어 있어 실제 운영환경에 쓰기에는 적합하지 않다.
- 대체 서비스 
    * [SendGrid](https://sendgrid.com/)
    * [mailgun](https://www.mailgun.com/)
    * [Amazon](https://aws.amazon.com/ko/ses/)
    * [Google G Suite](https://workspace.google.com/)
<br>

## EmailService
- 현재는 이메일을 보낼 때 SimpleMailMessage를 사용하고 있지만 실제 웹서비스에서는 대부분 html로 만들어진 이메일을 보낸다.
    * 그렇게 하려면 mimemessage로 보내야 한다.

### 구현
- EmailService 인터페이스 생성
```java
public interface EmailService {

    void sendEmail(EmailMessage emailMessage);
}
```
- EmailMessage 클래스 생성
```java
@Data
@Builder
public class EmailMessage {

    private String to;

    private String subject;

    private String message;

}
```
- Profile이 local일 때 사용할 ConsoleEmailService 클래스 생성
    * Bean을 주입받을 필요없고 logging만 하면 된다.
```java
@Slf4j
@Profile("local")
@Component
public class ConsoleEmailService implements EmailService{

    @Override
    public void sendEmail(EmailMessage emailMessage) {
        log.info("sent email: {}", emailMessage.getMessage());
    }
}
```
- Profile이 dev일 때 사용할 HtmlEmailService 클래스 생성
    * MimeMessageHelper : MimeMessage 만들 때 사용할 수 있는 유틸리티
```java
@Slf4j
@Profile("dev")
@Component
@RequiredArgsConstructor
public class HtmlEmailService implements EmailService {

    private final JavaMailSender javaMailSender;

    @Override
    public void sendEmail(EmailMessage emailMessage) {
        MimeMessage mimeMessage = javaMailSender.createMimeMessage();
        try {
            MimeMessageHelper mimeMessageHelper = new MimeMessageHelper(mimeMessage, false, "UTF-8");
            mimeMessageHelper.setTo(emailMessage.getTo());
            mimeMessageHelper.setSubject(emailMessage.getSubject());
            mimeMessageHelper.setText(emailMessage.getMessage(), false);
            javaMailSender.send(mimeMessage);
            log.info("sent email: {}", emailMessage.getMessage());
        } catch (MessagingException e) {
            log.error("failed to send email", e);
        }
    }
}
```
- ConsoleMailSender는 더이상 필요없으므로 삭제
- AccountService 수정
    * JavaMailSender 대신 EmailService 사용
        - EmailMessage을 만든 다음 EmailService를 통해 `senEmail()`한다.
    * `sendSignUpConfirmEmail()`, `sendLoginLink()`
        - JavaMailSender -> EmailMessage
```java
@Service
@Transactional
@RequiredArgsConstructor
public class AccountService implements UserDetailsService {

    ...
    private final EmailService emailService;
    ...

    ...

    public void sendSignUpConfirmEmail(Account newAccount) {
        EmailMessage emailMessage = EmailMessage.builder()
                .to(newAccount.getEmail())
                .subject("스터디올래, 회원 가입 인증")
                .message("/check-email-token?token=" + newAccount.getEmailCheckToken() +
                        "&email=" + newAccount.getEmail())
                .build();

        emailService.sendEmail(emailMessage);
    }

    ...

    public void sendLoginLink(Account account) {
        EmailMessage emailMessage = EmailMessage.builder()
                .to(account.getEmail())
                .subject("스터디올래, 로그인 링크")
                .message("/login-by-email?token=" + account.getEmailCheckToken() +
                        "&email=" + account.getEmail())
                .build();
        emailService.sendEmail(emailMessage);
    }

    ...
}
```
- 테스트 `signUpSubmit_with_correct_input()` : "회원 가입 처리 - 입력값 정상" 수정
    * JavaMailSender -> EmailService
```java
@Transactional
@SpringBootTest
@AutoConfigureMockMvc
class AccountControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private AccountRepository accountRepository;

    @MockBean
    EmailService emailService;

    ...

    @DisplayName("회원 가입 처리 - 입력값 정상")
    @Test
    void signUpSubmit_with_correct_input() throws Exception {
        mockMvc.perform(post("/sign-up")
                .param("nickname", "hayoung")
                .param("email", "hayoung@email.com")
                .param("password", "12345678")
                .with(csrf()))
                .andExpect(status().is3xxRedirection())
                .andExpect(view().name("redirect:/"))
                .andExpect(authenticated().withUsername("hayoung"));

        // 패스워드 인코딩 테스트
        Account account = accountRepository.findByEmail("hayoung@email.com");
        assertNotNull(account);
        assertNotEquals(account.getPassword(), "12345678");
        assertNotNull(account.getEmailCheckToken());
        then(emailService).should().sendEmail(any(EmailMessage.class));
    }
}
```
- 이제 Profile을 local로 설정하면 콘솔에 이메일 인증 메세지가 출력되고, dev로 설정하면 실제 이메일 서버로 이메일이 온다.
<br>
