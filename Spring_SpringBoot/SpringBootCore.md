# 스프링 부트 핵심 기능

## [SpringApplication](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-spring-application.html#boot-features-spring-application)

### 기본 로그 레벨 INFO
- 지금까지 `SpringApplication.run(SpringinitApplication.class, args);` 이렇게 실행했지만 SpringApplication이 제공하는 커스터마이징한 기능을 사용하기 어렵다.
- 그래서 인스턴스를 만들어서 run을 하는 방법으로 하면 다양한 기능을 사용할 수 있다.
```java
@SpringBootApplication
public class SpringinitApplication {

    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(SpringinitApplication.class);
        app.run(args);
    }
}
```
- 아무런 옵션도 변경하지 않고 실행하면 기본적으로 Application의 로그 레벨은 INFO 레벨이다.
    * IntelliJ의 Edit Configuration에서 VM options값을 -Ddebug로 설정해서 실행하면, 디버그 모드로 애플리케이션이 동작하고 DEBUG 레벨의 로그도 볼 수 있다.
    * 디버그 레벨로 로그를 찍을 때 한 가지 특이한 점은 어떤 자동 설정이 적용 됐는지, 적용 안된 자동 설정은 왜 안됐는지에 관한 로그를 볼 수 있다.
<br>

### FailureAnalyzer
- 어떠한 애플리케이션 에러가 났을 때 에러 메세지를 조금 더 이쁘게 보여주는 기능
- 기본적으로 스프링 부트 애플리케이션은 여러가지 FailureAnalyzer들이 등록되어 있다.
- 직접 만들어서 등록할 수도 있지만 거의 쓰지 않는다.
<br>

### 배너
- 배너란 애플리케이션 실행 할 때 기본적으로 보이는 Spring 이라고 적혀있는 것
- banner.txt | gif | jpg | png
- 배너를 바꾸고 싶을 때는 src/main/resources에 banner.txt 파일에 배너를 만들면 된다.
- `${spring-boot.version}` 등의 변수를 사용할 수 있다.
    * 일부는 manifest 파일이 생성이 되어야 사용 가능한 변수가 있다.
    * 그런 변수들은 mvn package를 해서 jar 파일을 만들면 manifest파일도 만들기 때문에 사용 가능하다. 
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/SpringBootCore_1.jpg" width="600px"></p>

- 배너를 코딩으로 구현하기
    * 배너를 코딩으로 구현하면 gif, jpg, png 등 이미지 파일도 사용 가능하다.
    * application.properties에 spring.banner의 img관련된 속성을 사용하면 된다.
    * 커스텀도 가능(banner.txt파일 추가 방법과 같이 쓰면 txt파일이 나온다)
```java
@SpringBootApplication
public class SpringinitApplication {

    public static void main(String[] args) {
        SrpingApplication app = new SpringApplication(SpringintitApplication.class);
        app.setBanner((enviroment, sourceClass, out) -> {
            out.println("=========================");
            out.println("Hayoung");
            out.println("=========================");
        })
        app.run(args);
    }
}
```
- 배너를 끄는 방법
    * `app.setBannerMode(Banner.Mode.OFF);`
- SpringApplicationBuilder로 빌더 패턴 사용도 가능하다.
<br>

### ApplicationEvent 등록
- 이벤트 리스너가 bean으로 등록하면 등록되어 있는 bean중에 해당하는 이벤트에 대한 리스너를 알아서 실행해준다.
- 중요한 점은 이벤트가 언제 발생하는 시점이다.
    1. `ApplicationStartedEvent` : ApplicationContext가 만들어진 다음에 발생한 이벤트들은 bean을 실행할 수 있다.
    2. `ApplicationStartingEvent` : ApplicationContext가 만들어지기 전에 발생한 이벤트는 bean으로 등록한다 하더라도 리스너가 동작하지 않는다.
    * 2번의 경우 직접 등록을 해줘야 한다.
```java
// @Component 선언을 해줄 필요가 없다.
public class SampleListener implements ApplicationListener<ApplicationStartingEvent> {

    @Override
    public void onApplicationEvent(ApplicationStartingEvent applicationStartingEvent) {
        System.out.println("=======================");
        System.out.println("Application is Starting");
        System.out.println("=======================");
    }
}
```
```java
@SpringBootApplication
public class SpringinitApplication {

	public static void main(String[] args) {
		SpringApplication app = new SpringApplication(SpringinitApplication.class);
		app.addListeners(new SampleListener());
		app.run(args);
	}
}
``` 
<br>

### WebApplicationType 설정
- `app.setWebApplicationType(WebApllicationType.타입);`
- SERVELET : 스프링 MVC가 들어있으면 SERVELET으로 돈다. 
- REACTIVE : 스프링 WebFlux가 들어있으면 REACTIVE 돈다. (SERVELET이 있으면 무조건 SERVLET)
- None : 둘 다 없으면 NONE으로 동작
- 둘 다 있으면 무조건 SERVELET으로 동작하지만 위의 코드로 타입을 REACIVE로 설정하면 REACTIVE로 가능하다.
<br>

### 애플리케이션 아규먼트 사용하기
- ApplicationArguments를 빈으로 등록해 주니까 가져다 쓰면 된다.
- 만약 Configuration Vm optionsd에 -Dfoo, Program arguments에는 --bar로 설정하고 아래의 코드를 실행하면 다음과 같다.
```java
@Component
public class ArgumentsTest {
  // 어떤 bean의 생성자가 1개이고, 생성자의 파라미터가 bean일 경우 그 bean을 spring이 알아서 주입해준다.
  public ArgumentsTest(ApplicationArguments arguments) {
    System.out.println("foo: " + arguments.containsOption("foo"));
    System.out.println("bar: " + arguments.containsOption("bar"));
  }
}
/* 실행결과
foo: false
bar: true 
*/
```
- -D로 들어오는 VM options는 application argument가 아니고 --로 들어오는 것이 application argument이다.
<br>

### 애플리케이션 실행한 뒤 뭔가 더 실행하고 싶을 때 
- ApplicationRunner 또는 CommandLineRunner 방법이 있지만 ApplicationRunner가 추천된다.
    * 더 고급 유용한 기능 사용 가능
1. ApplicationRunner
- ApplicationArguments 타입으로 메소드를 만들어주기 때문에 제공하는 유용한 메소드를 사용할 수 있다.
```java
@Component
public class SampleApplicationRunner implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("foo: "+args.containsOption("foo"));
        System.out.println("bar: "+args.containsOption("bar"));
    }
}
```
2. CommandLineRunner
```java
@Component
public class SampleCommandLineRunner implements CommandLineRunner {

    @Override
    public void run(String... args) throws Exception {
        Arrays.stream(args).forEach(System.out::println);
    }
}
```
- ApplicationRunner들이 여러 개일 경우 `@Order`를 사용해 순서를 정할 수 있다.
    * ex) `@Order(1)`
    * 숫자가 낮을수록 먼저 실행된다.
<br>

## [외부 설정](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-external-config)
- 외부 설정 파일은 애플리케이션에서 사용하는 여러가지 설정 값들을 애플리케이션 안 또는 밖에 설정할 수 있는 기능이다. 
- 가장 많이 쓰고 중요한 애플리케이션 설정 파일은 application.properties 파일이다.
    * 스프링 부트가 애플리케이션을 구동할 때 자동으로 로딩하는 파일 이름 컨벤션이다.
    * 파일 안에 key-value형태로 값을 정의하면 애플리케이션에서 참조해서 사용할 수 있다.
    * 참조하는 가장 기본적인 방법(우선순위 15)
        * @Value는 SpEL을 사용할 수 있다.
    ```java
    @Value("${hayoung.name}")
    private String name;
    ```
<br>

### 사용할 수 있는 외부 설정
- properties
- YAML
- 환경 변수
- 커맨드 라인 argument
<br>

### 프로퍼티 우선 순위
1. 유저 홈 디렉토리에 있는 spring-boot-dev-tools.properties
2. 테스트에 있는 @TestPropertySource (
    * ex) `@TestPropertySource(properties = "hayoung.name=hayoung2")`
3. @SpringBootTest 애노테이션의 properties 애트리뷰트 
    * ex) `@SpringBootTest(properties = "hayoung.name=hayoung2")`
4. 커맨드라인 아규먼트
    * ex) `java -jar target/springinit-0.0.1-SNAPSHOT.jar --hayoung.name=hayoung`
5. SPRING_APPLICATION_JSON (환경 변수 또는 시스템 프로퍼티)에 들어있는 프로퍼티
6. ServletConfig 파라미터
7. ServletContext 파라미터
8. Java:comp/env JNDI 애트리뷰트
9. System.getProperties() 자바 시스템 프로퍼티
10. OS 환경 변수
11. RandomValuePropertySource
12. JAR 밖에 있는 특정 프로파일용 application.properties
13. JAR 안에 있는 특정 프로파일용 application.properties
14. JAR 밖에 있는 application.properties
15. JAR 안에 있는 application.properties
16. @PropertySource
17. 기본 프로퍼티(SpringApplication.setDefaultProperties)
<br>

### 랜덤값 설정하기 
- `${random.*}`
<br>

### 플레이스 홀더
- 이미 정의한 값을 사용할 수 있다.
```
name=hayoung
fullName=${name} Kim
```
<br>

### application.properties 중복
- src/main/resources와 src/test/resources에 중복으로 application.properties 파일이 있을 경우 문제가 자주 발생한다. 
    * main쪽 application.properties에 값이 더 있는데 test에서 참조하려고 할 경우 오류 발생
    * 동일한 key값이 있을 경우 우선순위가 test쪽이 더 높기 때문에 덮어씌워 진다.
- 이를 해결하기 위해 test쪽의 application.properties를 삭제하고 test.properties를 만든 뒤 `@TestPropertySource(locations = "classpath:/test.properties")`라고 하면
- 빌드할 때 application.properties도 들어가고 test.properties도 들어가서 우선순위가 적용된다.
<br>

### application.propertie 우선순위
1. file:./config : 프로젝트 root 밑에 config라는 디렉토리를 만들고 놓기
2. file:./ : jar파일을 실행하는 위치에 놓기
3. classpath:./config/
4. classpath:/
<br>

###  type-safe한 프로퍼티 @ConfigurationProperties
- 외부 설정이 많은 경우 같은 key로 시작하는 외부 설정을 묶어서 하나의 bean으로 등록하는 방법
- properties에 있는 값들을 class에 바인딩을 해준다.
```java
@Component // bean으로 등록
@ConfigurationProperties("hayoung") // 마크를 해준다. 만약 Classpath에 Annotation Processor가 없다는 경고와 링크가 뜨면 meta정보를 생성 해주는 의존성을 추가해주면 된다. 

public class HayoungProperties {

    private String name;

    private int age;

    private String fullName;

    ...getter and setter...
}
```
```java
@SpringBootApplication
// @EnableConfigurationProperties(HayoungProperties.class) 해줘야 하지만 자동으로 등록해준다.
public class SpringinitApplication {

	public static void main(String[] args) {
		SpringApplication app = new SpringApplication(SpringinitApplication.class);
		app.run(args);
	}
}
```
- 이렇게 하면 Runner에서 @Autowired로 주입받아 사용할 수 있다.
```java
@Component
public class SampleRunner implements ApplicationRunner {

    @Autowired
    HayoungProperties hayoungProperties;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(hayoungProperties.getName());
        System.out.println(hayoungProperties.getAge());
        System.out.println(hayoungProperties.getFullName());
    }
}
```
- type-safe하다는 의미는 `@Value("${hayoung.name}")` 처럼 프로퍼티 값을 직접 입력하면서 발생할 수 있는 에러를 내지 않을 수 있다는 의미이다.
    * @ConfigurationProperties으로 정의하고 빈으로 만든 뒤 getter를 통해서 값을 가져오기 때문에 @Value로 직접 쓰는 것 보다 안전하게 사용할 수있다.
<br>

### Relaxed Binding(융통성 있는 바인딩)
- 앞에서 설정한 프로퍼티를 camelcase로 적지 않아도 mapping을 해준다
    * `hayoung.fullName= ${hayoung.name} Kim`
    * `hayoung.full-name= ${hayoung.name} Kim`
    * `hayoung.full_name= ${hayoung.name} Kim`
<br>

### 프로퍼티 type conversion 지원
- properties 문서안에서는 type이라는 것이 없다.
    * `hayoung.age= ${random.int(0,100)}`나 `hayoung.age= 100`도 모두 문자열인데 사용할 때는 int로 변환이 되서 type conversion이 일어난다.
- @DurationUnit
    * 스프링 부트가 제공하는 독특한 conversion type으로 시간 정보를 받고 싶을 때 사용 가능하다.
    ```java
    @DutationUnit(ChronoUnit.SECONDS)   // 초로 받겠다
    private Duration sessionTimeout = Duration.ofSeconds(30);   // 값이 안들어오면 기본값은 30초
    /* application.properties에
    hayoung.sestionTimeout = 25로 주고 실행하면 초로 conversion이 일어난다.
    */
    ```
    * @DurationUnit 애노테이션을 쓰지않고 `hayoung.sestionTimeout = 25s`로 s를 붙이면 알아서 conversion을 해준다.
<br>

### 프로퍼티 값 검증
- @Validated 애노테이션을 붙이면 값에 @NotEmpty, @NotNull, @Size등을 사용할 수 있다. 
```java
@Component
@ConfigurationProperties("hayoung")
@Validated
public class HayoungProperties {

    @NotEmpty
    private String name;
    
    ...
}
```
- @NotEmpty로 설정했는데 프로퍼티 값이 비워져 있으면 에러가 난다.
<br>

## Profile(프로파일)
- 스프링 프레임워크에서 제공해주는 기능으로 각각의 환경에 따라 bean을 다르게 사용해야 할 경우, 또는 특정 환경에서만 어떠한 bean을 등록해야하는 경우에 사용한다.
```java
@Profile("prod")
@Configuration
public class BaseConfiguration {

    @Bean
    public String hello() {
        return "hello";
    }
}
```
```java
@Profile("test")
@Configuration
public class TestConfiguration {

    @Bean
    public String hello() {
        return "hello test";
    }
}
```
- `@Profile("prod")`라고 붙이면 bean 설정 파일 자체가 "prod"라는 Profile일때 사용이 된다. "prod"가 아니면 사용이 안된다.
<br>

### profile 활성화
- application.properties에 `spring.profiles.active` 설정을 추가하면 profile을 활성화 할 수 있다. 
- `spring.profiles.active=prod`라고 설정하면 prod가 활성화된다.(test라고 하면 test profile이 활성화)
    * 이것도 프로퍼티이므로 우선순위가 적용된다.
<br>

### profile용 프로퍼티
- `application-profile이름.properties`
- profile과 관련된 properties들의 우선순위가 application.properties보다 높다.
- 따라서 profile용 properties를 만들고 같은 key값을 설정하면 profile properties가 오버라이딩되어 덮어씌워 진다.
```java
//application-prod.properties
hayoung.name=hayoung prod   // application.properties의 hayoung.name을 덮어쓴다.
```
<br>

### profile 추가
- `spring.profiles.include`
- 추가할 profile을 설정
```java
//application-prod.properties
hayoung.name=hayoung prod
spring.profiles.include=proddb  // prod 설정이 읽혀졌을 때(활성화 되었을 때) proddb라는 profile도 활성화된다.
```
<br>

## 로깅 - 스프링 부트 기본 로거 설정
- 스프링 부트는 기본적으로 Commons Logging을 사용한다.  결국 SLF4j를 사용하게 된다. 소스코드에서도 SLF4j를 사용하면 된다.
- Logging Facade : Commons Logging, SLF4j
- 로거 : JUL(Java Utility Logging), Log4J2, Logback
<br>

### Logging Facade(로깅 퍼사드)와 로거
- Commons Logging과 SLF4j는 실제 로깅을 하는 것이 아니라 로거 API들을 추상화 해놓은 인터페이스들이다. 
- 주로 프레임워크들은 로깅 퍼사드를 사용해서 코딩한다. (애플리케이션을 만드는 개발자도 로깅 퍼사드를 통해서 로거를 써도 문제없다.)
- 로깅 퍼사드의 장점
    * 로깅 퍼사드 밑에 있는 로거를 바꿀 수 있다.
<br>

### [스프링 5에서의 로거 변경사항](https://docs.spring.io/spring/docs/5.0.0.RC3/spring-framework-reference/overview.html#overview-logging)
- 자체 내에서 Spring-JCL 모듈을 만들어서 Commons Logging을 컴파일 시점에 SLF4j나 Log4j2로 변경할 수 있다.
- pom.xml에 exclusion 안해도 된다.
- 결론적으로 Spring-JCL이 개입하면서 Commons Logging -> SLF4j or Log4j2 -> Logback으로 보낸다.
<br>

### 스프링 부트 로깅
- --debug
    * 일부 핵심 라이브러리들(core loggers)만 디버깅 모드로 찍어준다.
- --trace
    * 전부 디버깅 모드로 찍힌다.
- spring.output.ansi.enabled
    * 컬러 출력
    * ex) `spring.output.ansi.enabled=always`
- logging.file 또는 logging.path
    * 파일 출력
    * 로그파일은 기본적으로 10M까지 저장되고, 넘치면 아카이빙하는 등 여러가지 설정도 할 수 있다.
- logging.level.패키지 = 로그 레벨
    * 로그 레벨 조정
    * ex) `logging.level.me.hayoung.springintit=DEBUG`
<br>

## [로깅 - 커스터마이징](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto-logging)

### 커스텀 로그 설정 파일 사용하기
- Logback : logback-spring.xml (logback.xml도 가능하지만 logback.xml은 너무 일찍 로딩이 되기 때문에 Logback extension을 사용할 수 없다.)
- Log4J2 : log4j2-spring.xml
- JUL : loggging.properties
- logback-spring.xml 예제
```java
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/base.xml"/>
    <logger name="me.hayoung" level="DEBUG"/>
</configuration>
```
- Logback extension
    * `<springProfile name="프로파일">`
        - 특정 profile일 때만 설정이 되도록 할 수 있다.
    * `<springProperty>`
        - Environment에 들어있는 property들도 사용할 수 있다.
<br>

### [로거를 Log4j2로 변경하기](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto-configure-log4j-for-logging)
- Log4j2로 변경하면 최종적으로 로그 메세지를 찍는 것은 Log4j2가 된다.
```html
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```
<br>

## 테스트
- 일단 pom.xml에 spring-boot-starter-test 의존성이 있어야한다.

### @SpringBootTest
- @RunWith(SrpingRunner.class)와 같이 써야한다.
- bean 설정 파일을 알아서 찾아준다.
- webEnvironment
    * MOCK 
        - 내장 톰캣 구동 하지 않는다.
        - SpringBootTest의 webEnvironment의 기본값은 MOCK이다. 
        - Servlet을 Mocking 한 것이 구동되고 Mockup이 된 Servlet에 어떤 Interaction 할려면 MockMVC라는 클라이언트를 구성해야 된다. 
        - MockMVC를 만드는 방법이 여러가지가 있지만 아래 코드와 같이 구성하는 것이 가장 쉽게 만드는 방법이다.
        ```java
        @RunWith(SpringRunner.class)
        @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
        @AutoConfigureMockMvc
        public class SampleControllerTest {

            @Autowired
            MockMvc mockMvc;

            @Test
            public void hello() throws Exception {
                mockMvc.perform(get("/hello"))  //hello로 get 요청
                        .andExpect(status().isOk())   // status 코드가 200
                        .andExpect(content().string("hello hayoung"))     // content의 내용이 hello hayoung인지
                        .andDo(print());    // 프린트
            }
        }
        ```
    * RANDOM_PORT, DEFINED_PORT
        * 내장 톰캣 구동되고 Servelet이 올라간다.
        * 테스트용 REST 템플릿(TestRestTemplate)이나 TEST용 웹 클라이언트를 써야한다.
        ```java
        @RunWith(SpringRunner.class)
        @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
        public class SampleControllerRestTemplate {

            @Autowired
            TestRestTemplate testRestTemplate;

            @Test
            public void hello() throws Exception {
             String result = testRestTemplate.getForObject("/hello", String.class);
                assertThat(result).isEqualTo("hello hayoung");
            }
        }
        ```
<br>

### @MockBean
- ApplicationContext에 들어있는 bean을 Mock으로 만든 객체로 교체 한다.
- 모든 @Test마다 자동으로 리셋
```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class SampleControllerRestTemplate {

    @Autowired
    TestRestTemplate testRestTemplate;

    @MockBean
    SampleService mockSampleService;    //ApplicationContext에 있는 SampleService가 교체됨

    @Test
    public void hello() throws Exception {
        when(mockSampleService.getName()).thenReturn("Kimhayoung");
   
        String result = testRestTemplate.getForObject("/hello", String.class);
        assertThat(result).isEqualTo("hello Kimhayoung");
    }
}
```
<br>

### WebTestClient
- Spring5의 WebFlux에 추가된 RestClient 중 하나로 Asynchronous(비동기식) 방식
- 기존의 RestClient는 Synchronous(동기식)
- 사용하려면 webflux 쪽 dependency를 추가 해야한다.
```html
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```
- 테스트 코드
```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class SampleControllerRestTemplate {
    @Autowired
    WebTestClient webTestClient;

    @MockBean
    SampleService mockSampleService;

    @Test
    public void hello() throws Exception {
        when(mockSampleService.getName()).thenReturn("Kimhayoung");
        webTestClient.get().uri("/hello").exchange()
                .expectStatus().isOk()
                .expectBody(String.class).isEqualTo("hello Kimhayoung");
    }
}
```
<br>

### Slice Test
- 지금까지 위의 테스트는 큰 규모의 테스트이다. 테스트 하고 싶은 것만 등록하고싶을 때 사용
- 레이어 별로 잘라서 적용이 된다. 
- [@JsonTest](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-testing-spring-boot-applications-testing-autoconfigured-json-tests)
    * 예상되는 JSON 형식을 테스트 해볼 수 있다.
- [@WebMvcTest](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-testing-spring-boot-applications-testing-autoconfigured-mvc-tests)
    * 슬라이싱해서 하나만 테스트 할 수 있다.
    * 빈 하나만 테스트하므로 굉장히 가볍다.
    * 테스트 할 Controller와 필수 웹과 관련된 모듈들만 bean으로 등록되고 일반적인 컴포넌트들은 bean으로 등록되지 않음
    * 따라서 의존성들이 다 끊기기 때문에 사용하는 의존성이 있다면 MockBean으로 만들어서 채워줘야 한다.
    * @WebMvcTest는 항상 MockMvc로 테스트 해야한다.
    ```java
    @RunWith(SpringRunner.class)
    @WebMvcTest(SampleController.class)
    public class SampleControllerWebMvcTest {

        @MockBean
        SampleService mockSampleService;

        @Autowired
        MockMvc mockMvc;

        @Test
        public void hello() throws Exception {
            when(mockSampleService.getName()).thenReturn("Kimhayoung"); // MockBean으로 주입한 Mock Service

            mockMvc.perform(get("/hello"))
                    .andExpect(status().isOk()) // status 는 200 이고
                    .andExpect(content().string("hello Kimhayoung"))
                    .andDo(print());
        }
    }
    ```
- @WebFluxTest
- @DataJpaTest
- ...
<br>

### 테스트 유틸리티
- **OutputCapture**
    * 제일 유용하고 나머지는 잘 쓰이지 않는다.
    * JUnit에 있는 Rule을 확장해서 만든 것이다. 
    ```java
    @RestController
    public class SampleController {

        Logger logger = LoggerFactory.getLogger(SampleController.class);

        @Autowired
        private SampleService sampleService;

        @GetMapping("/hello")
        public String hello() {
            logger.info("holoman");
            System.out.println("skip");
            return "hello" + sampleService.getName();
        }
    }
    ```
    ```java
    @RunWith(SpringRunner.class)
    @WebMvcTest(SampleController.class)
    public class SampleControllerWebMvcTest {

        @Rule
        // OutputCapture는 로그를 비롯해 console에 찍히는 모든 것을 capture한다.
        public OutputCapture outputCapture = new OutputCapture();

        @MockBean
        SampleService mockSampleService;

        @Autowired
        MockMvc mockMvc;

        @Test
        public void hello() throws Exception {
            when(mockSampleService.getName()).thenReturn("Kimhayoung");

            mockMvc.perform(get("/hello")) 
                    .andExpect(status().isOk())
                    .andExpect(content().string("hello Kimhayoung"))
                    .andDo(print());

            assertThat(outputCapture.toString())
                    .contains("Holoman")
                    .contains("skip");
        }   
    }
    ```
- TestPropertyValues
- TestRestTemplate
- ConfigFileApplicationContextInitializer
<br>

## Spring-Boot-Devtools

### 캐시 설정을 개발 환경에 맞게 변경
- 스프링 부트가 제공하는 optional한 tool로 반드시 써야하는 것도, 기본적으로 적용되어 있는 것도 아니다.
- 사용하려면 의존성 추가
```html
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>
```
- 의존성들을 추가하는 순간 기본적으로 적용되는 properties들이 바뀐다.
    * [Reference](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-devtools-property-defaults)
    * [Reference에 있는 링크를 누르면 나오는 바뀌는 것들](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-devtools/src/main/java/org/springframework/boot/devtools/env/DevToolsPropertyDefaultsPostProcessor.java)
    * 주로 cache를 끄는 것과 관련되어 있다.
<br>

### Auto Restart
- classpath에 있는 파일이 변경 될 때마다 자동으로 restart해준다
- 톰캣을 직접 껐다 키는 (run을 하는) 속도보다 빠르다.
- 이유 : SpringBoot는 Class Loader를 두 개를 사용한다.
    * Base Class Loader : 기본적으로 바뀌지 않는 기본 Class Loader
    * Restart Class Loader: Application 에서 사용하는 일반적인 Class Loader  
-  릴로딩 보다는 느리다. (JRebel 같은건 아니다.)
- 리스타트 하고 싶지 않은 리소스 
    * `spring.devtools.restart.exclude`
- 리스타트 기능 끄기 
    * `spring.devtools.restart.enabled = false`
<br>

### Live Reload
- Restart 했을 때 브라우저까지 자동 리프레시 되는 것
- 브라우저 플러그인을 설치해야 한다.
<br>

### 글로벌 설정
- 프로퍼티 우선순위의 1순위
- `~/.spring-boot-devtools.properties`
* spring-boot-devtools 의존성이 있어야 1순위다.
<br>

#### Remote Applications
- 원격에다 애플리케이션을 띄우고 로컬에서 실행하는 것.
- production 용이 아니다. 
- 쓸 일이 거의 없다.