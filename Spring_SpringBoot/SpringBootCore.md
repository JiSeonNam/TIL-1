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
    * ex) `@Order(1)` : 낮을수록 먼저 실행된다.
<br>
