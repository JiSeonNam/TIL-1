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
    * 1. `ApplicationStartedEvent` : ApplicationContext가 만들어진 다음에 발생한 이벤트들은 bean을 실행할 수 있다.
    * 2. `ApplicationStartingEvent` : ApplicationContext가 만들어지기 전에 발생한 이벤트는 bean으로 등록한다 하더라도 리스너가 동작하지 않는다.
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