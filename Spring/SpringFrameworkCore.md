> [강의 링크](https://www.inflearn.com/course/spring-framework_core)
 
 # 스프링 프레임워크 핵심 기술

 ## IoC 컨테이너와 빈
- Inversion of Control : 의존 관계 주입(Dependency Injection)이라고도 하며, 어떤 객체가 사용하는 의존 객체를 직접 만들어 사용하는게 아니라, 주입 받아 사용하는 방법을 말한다.
<br>

### 스프링 IoC 컨테이너
- [BeanFactory](https://docs.spring.io/spring-framework/docs/5.0.8.RELEASE/javadoc-api/org/springframework/beans/factory/BeanFactory.html) : 스프링 IoC 컨테이너의 최상위 인터페이스 
- ApplicationContext
    * BeanFactory를 상속받고 있기 때문에 BeanFactory의 모든 기능을 포함하며, 추가 기능도 제공되기 때문에 일반적으로 BeanFactory보다 추천된다. 
    * 메세지 소스 기능(i18n)
    * 이벤트 발행 기능
    * 리소스 로딩 기능 등
- 애플리케이션 컴포넌트의 중앙 저장소.
- 빈 설정 소스로 부터 빈 정의를 읽어들이고, 빈을 구성하고 제공한다.
<br>

### 빈
- Bean : 스프링 IoC 컨테이너가 가지고 있고 관리하는 객체.
- 빈으로 등록되면 좋은 장점
    * 의존성 관리(빈으로 등록되어 있어야만 의존성 주입이 가능하다.)
    * Test하기 편리해진다.
    * 스코프
        - 싱글톤 : 하나의 객체를 사용, 기본적으로 빈으로 등록할 때 싱글톤으로 등록된다.
        - 프로토타입 : 매번 다른 객체를 사용
    * 라이프사이클 인터페이스 지원
<br>

## ApplicationContext와 다양한 빈 설정 방법

### 1. Spring bean 설정 파일을 만들고 bean을 주입하는 방법
- 고전적인 방법으로 요즘에는 거의 쓰이지 않는다.
- application.xml 파일을 만들고 bean을 등록한다. 
```java
// application.xml
<? xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans">
       
  <bean id="bookService"
        class="me.whiteship.springapplicationcontext.BookService">  // 빈 만들기
        <property name="bookRepository" ref="bookRepository" /> // 빈 주입
        // name은 setter에서 가져온 것이고, ref는 다른 bean을 참조한다는 뜻.
        // ref에는 setter(name)에 들어갈 다른 bean의 id 값이 들어와야한다. 
  <bean>
  
  <bean id="bookRepository"
         class="me.whiteship.springapplicationcontext.BookService"/>
```
- bean 설정과 등록이 끝나면 ApplicationContext를 만들어 사용가능하다.
```java
public class DemoApplication {

  public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
    
    String[] beanDefinitionNames = context.getBeanDefinitionNames();
    System.out.println(Arrays.toString(beanDefinitionNames);
    
    BookService bookService = (BookService) context.getBean("bookService");
    System.out.println(bookService.bookRepository != null);
    
    // application.xml 파일에서 빈으로 등록되고 의존성 주입을 받았기 때문에 true가 출력된다.
```
- 이러한 방법은 모든 빈을 일일히 등록(`<bean id = " ">`)을 해줘야 하기 때문에 번거롭다.
<br>

### 2. context-component-scan
- bean을 스캐닝해서 등록하는 것이다. (Spring 2.5부터 등장)
- @Component와 Component 애노테이션을 확장한 @Service, @Repository 등을 사용해 bean으로 등록 할 수 있다.
- 빈으로 등록만 되기 때문에 의존성 주입은 @Autowired 나 @Inject를 통해 받아야 한다.
```java
<? xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans">
  
  <context:component-scan base-package="me.whiteshp.springapplicationcontext"/>
  //package이하의 애너테이션들을 스캐닝해서 등록해준다

```
- BookRepository
```java
@Repository
public class BookRepository {
}
```
- BookService
```java
@Service
public class BookService {

  @Autowired
  BookRepository bookRepository;
  
  public void setBookRepository(BookRepository bookRepository) {
    this.bookRepository = bookRepository;
  }
}
```
<br>

### 3. java 설정 파일
- @Comfiguration이라 애노테이션을 설정하고 @Bean을 등록한다
- 굉장히 유연한 bean 설정이 가능하다.
- Application.config
```java
@Configuration
public class ApplicationConfig {
  
  @Bean
  public BookRepository bookRepository() {
    return new BookRepository();
  }
  
  // 메소드를 호출해서 의존성을 주입받는 방법
  @Bean
  public BookService bookService() {
    BookService bookService = new BookService();
    bookService.setBookRepository(bookRepository());
    return bookService;
  }
  
  // 메소드 파라미터로 의존성을 주입받는 방법
  @Bean
  public BookService bookService(BookRepository bookRepository) {
    BookService bookService = new BookService();
    bookService.setBookRepository(bookRepository);
    return bookService;
  }
  
  // setter를 사용할 경우 의존성 주입을 직접하지 않고, @Autowired를 쓰는 방법
  @Bean
  public BookService bookService(BookRepository bookRepository) {
    return new BookService();
  }
  /* BookService에 @Autowired 애노테이션을 붙여야 한다.    
    public class BookService {
        @Autowired
        BookRepository bookRepository;
            ...
    } 
   */
```
- DemoApplication.java 
```java
public class DemoApplication {

  public static void main(String[] args) {
    ApplicationContext context = new AnnotationConfigApplicationContext(ApplicationConfig.class);   
    // ApplicationConfig 클래스를 bean 설정으로 사용
    // ApplicationConfig에서 @Bean이 달린 bean 정의들을 읽어서 bean들로 등록하고, 의존성 주입은 코드를 작성한 대로 발생한다.
```
<br>

### java 설정 파일의 componet-scanning
- java 설정 파일을 통한 방법도 일일히 bean을 등록해야 한다. 따라서 xml에서 했던 것과 비슷하게 component-scanning을 할 수 있다.
```java
@Configuration
@ComponentScan(basePackagesClasses = DemoApplication.class) // class가 위치한 곳부터 Component-scanning을 하라는 의미
public class ApplicationConfig {
}
```
- 현재 사용되고 있는 스프링 부트 기반으로 스프링을 쓰고 있는 가장 가까운 방법이다.
- 스프링에서는 `@ComponetScan`과 `@Configuration`을 사용하지만 스프링 부트에서는 `@SpringBootApplication`을 붙이면 필요없다. 
- DemoApplication의 ApplicationContext와 ApplicationConfig.java도 필요없다.
- 아래 코드가 사실상 bean 설정 파일이다.
```java
@SpringBootApplication
public class DemoApplication {
 
 public static void main(String[] args) {
  
  }
}
```
<br>

## Autowired

### 생성자를 사용한 의존성 주입
```java
@Service    // bean으로 등록
public class BookService {
       
    BookRepository bookRepository;

    @Autowired
    public BookService(BookRepository bookRespository) {
           this.bookRepository = bookRepository;
    }
}
```
```java
@Repository // bean으로 등록
// 애노테이션을 붙이지 않으면 BookRepository의 bean을 찾지 못해서 오류가 난다.
public class BookRepository {
}
```

### Setter를 사용한 의존성 주입
```java
@Service
public class BookService {
       
    BookRepository bookRepository;

    @Autowired
    public void setBookRepository(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
}
```
- BookService 자체의 인스턴스는 만들 수 있지만 `@Autowired`라는 애노테이션이 있기 때문에 의존성 주입을 하려고 시도한다가 실패한다.
- 이런 경우 `@Autowired(required = false)`라고 설정(기본값은 true)하면 BookService의 인스턴스는 만들어져 bean으로 등록되고, BookRepository는 의존성 주입이 안된 상태로 빈으로 등록된다.
<br>

### Field에 의존성 주입
```java
@Service
public class BookService {

     @Autowired(required = false)
     BookRepository bookRepository;
}
```
- 이렇게 Setter나 field를 통한 의존성 주입은 `@Autowired(required = false)`를 사용하여 BookService가 BookRepository의 의존성 없이도 bean으로 등록되도록 할 수 있다.
- 그러나 생성자를 사용한 의존성 주입은 bean을 만들 떄도 개입이 되기 때문에 전달받아야하는 타입의 bean이 없으면 인스턴스를 만들지 못하고 bean으로도 등록되지 못한다.
<br>

### 해당 타입의 빈이 여러 개인 경우
```java
public interface BookRepository {
}
```
```java
@Repository
public class MyBookRepository implements BookRepository {
}
```
```java
@Repository
public class KeesunBookRepository implements BookRepository {
}
```
```java
@Service
public class BookService { 
     
     @Autowired
     BookRepository bookRepository;
}
```
- 위와 같은 경우에는 둘 중 어떤 것을 원하는지 알 수 없기 때문에  BookService에 의존성 주입을 못해준다. 
- 이 경우 다음과 같은 방법으로 해결할 수 있다. 
    * `@Primary` : 여러가지 동일한 bean 중 이 bean을 사용할 것이다라는 의미
    ```java
    @Repository @Primary
    public class MyBookRepository implements BookRepository {
    }
    ```
    * `@Qualifier` : 사용하고 싶은 bean의 id를 선언해준다.
    ```java
    @Service
    public class BookService { 
     
        @Autowired @Qualifier("keesunBookRepository")   //첫글자는 소문자!
        BookRepository bookRepository;
    }
    ```
    * 여러 개의 bean 모두 받을 수도 있다.
    ```java
    @Service
    public class BookService { 
     
        @Autowired
        List<BookRepository> bookRepositories; 
    }
    ```
<br>

### Autowired 동작원리
- AutowiredAnnotationBeanPostProcessor 가 기본적으로 Bean으로 등록되어있고
- BeanFactory 가 자신에게 등록된 BeanPostProcessor 들을 찾아서 일반적인 Bean들에게 로직을 적용함.
- 따라서 bean으로 등록되어 있는 모든 bean들은 `@Autowired`로 주입 가능하다.
<br>

## @Component와 컴포넌트 스캔

### component-scan이 되는 대상
- @Component 
    * @Repository
    * @Service
    * Controller
    * Configuration
<br>

### 컴포넌트 스캔의 주요 기능
- 스캔 위치 설정
    * ComponentScan을 붙이고 있는 Configuration부터 component-scan을 시작한다.
    * 시작 지점을 담고 있는 패키지까지 스캔 범위이다. (시작 지점을 담고있는 패키지 외의 패키지는 scan하지 않는다.)
- 필터
    * component-scan을 한다고 해서 모든 애노테이션들을 처리해서 bean으로 등록해주진 않는다.
    * 그것을 걸러주는 것이 필터이다.
<br>

## 빈의 스코프

### Singleton scope
- 애플리케이션 전반에 걸쳐서 해당 빈의 인스턴스를 오직 한개 사용.
- 빈의 기본 스코프는 싱클톤 스코프이다.
- 따라서 아래 코드의 애플리케이션 실행 결과 proto의 인스턴스가 동일하다.
- 프로퍼티가 공유가 되기 때문에 Thread safe한 방법으로 코딩해야 한다.
- 모든 sigleton scope의 bean들은 기본값으로 ApplicationContext를 만들 때 만들게 되어 있다. (application 구동 시간이 좀 더 걸릴 수 있다.) 
```java
@Component
public class Single {
       
       @Autowired
       private Proto proto;
       
       public Proto getProto() {
              return proto;
       }
}
```
```java
@Component
public class Proto {
}
```
```java
@Component
public class AppRunner implements ApplicationRunner {

       @Autowired
       Single single;

       @Autowired
       Proto proto;

       @Override
       public void run(ApplicationArguments args) throws Exception {
              System.out.println(proto);
              System.out.println(single.getProto());
              // 둘다 동일한 래퍼런스가 출력된다.
       }
}
```
<br>

### Prototype scope
- 매번 새로운 인스턴스를 만들어서 사용
- Request, Session, WebSocket, Application, Thread scope 등은 프로토타입 scope와 유사하다.
- 따라서 아래 코드의 애플리케이션 실행 결과 prototype은 항상 다른 래퍼런스가 sigleton은 항상 같은 래퍼런스가 출력된다. 
```java
@Component @Scope("prototype")
public class Proto {
}
```
```java
@Component
public class AppRunner implements ApplicationRunner {
       
       @Autowired
       ApplicationContext ctx;
       
       @Override
       public void run(ApplicationArguments args) throws Exception {
              System.out.println(ctx.getBean(Proto.class));
              System.out.println(ctx.getBean(Proto.class));
              System.out.println(ctx.getBean(Proto.class));
              
              System.out.println(ctx.getBean(Single.class));
              System.out.println(ctx.getBean(Single.class));
              System.out.println(ctx.getBean(Single.class));
       }
}
```
<br>

### sigleton scope와 prototype socpe의 혼용
1. prototype 빈이 sigleton 빈을 참조하는 경우
```java
@Component @Scope("prototype")
public class Proto {

       @Autowired
       Single Single;
```
- sigle 인스턴스는 매번 같은 인스턴스가 들어오고 prototype 인스턴스는 매번 꺼낼 때마다 새로운 인스턴스가 생성된다. 
- prototype의 bean은 새롭지만 참조하는 sigleton scope의 bean은 항상 동일하다.
- 아무 문제가 없다. 

2. sigleton bean이 prototype bean을 참조하는 경우
- sigleton scope의 bean은 인스턴스가 한번만 만들어지고 prototype socpe의 bean도 이미 세팅이 되버린다. 
- 따라서 sigleton scope의 bean을 계속 쓸 때 prototype scope의 bean이 변경되지 않는다. (문제 발생)
```java
@Component
public class AppRunner implements ApplicationRunner {
       
       @Autowired
       ApplicationContext ctx;
       
       @Override
       public void run(ApplicationArguments args) throws Exception {
              System.out.println(ctx.getBean(Single.class).getProto());
              System.out.println(ctx.getBean(Single.class).getProto());
              System.out.println(ctx.getBean(Single.class).getProto());
              // 변경되지 않고 같은 인스턴스 값이 출력된다.

       }
}
```

3. 2번 문제를 해결하는 방법
- proxyMode를 설정
```java
@Component @Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class Proto {
}
```
  * proxyMode를 설정할 경우, 프로토타입 빈은 업데이트 된다. (proxyMode의 기본값은 default로 proxy를 사용하지 않는다는 옵션)
  * proxyMode를 쓴다는 것은 클래스 기반의 프록시로 감싸는 것이다.
  * 프록시로 감싸는 이유 : sigleton 인스턴스가 prototype 빈 인스턴스를 직접 참조하면 안되고 프록시를 거쳐서 참조해야 하기 때문
  * 프록시를 거쳐서 참조해야 하는 이유 : 직접 참조하면 이 prototype의 인스턴스를 매번 새로운 인스턴스로 바꿔줄 수 없기 때문에. 매 번 새로운 인스턴스로 바꿔줄 수 있는 프록시로 감싸도록 한다.
  * prototype 빈을 상속받은 클래스를 만들어서 프록시를 만들어주는 CG라이브러리 라는 third-party 라이브러리가 있다.(클래스도 프록시를 만들어줄 수 있게 해준다.)
  * 원래 Java JDK 안에 있는 다이나믹 프록시는 인터페이스의 프록시 밖에 못만든다.  
  * 위의 코드에서 처럼 `proxyMode = ScopedProxyMode.TARGET_CLASS`는 CG라이브러리를 이용해서 클래스를 상속받은 프록시를 만들라고 지시하는 것이다.
  * 만약 인터페이스가 있었다면 `proxyMode = ScopedProxyMode.INTERFACE`를 지시하여 JDK의 인터페이스 기반의 프록시를 만들어 사용

- ObjectProvider
  * 코드 수정을 통해 해결 가능하지만 코드에 스프링 코드가 들어가기 때문에 추천되지 않는다.
```java
@Component
public class Single {

       @Autowired
       private ObjectProvider<Proto> proto;
       
       public Proto getProto() {
              return proto.getIfAvailable();
       }
}
```
\* 일반적으로 sigleton 이외의 scope를 쓸 일이 거의 없다. 생긴다면 참고
<br><br>

## ApplicationContext - Environment

### Frofile
- ApplicationContext가 가지고 있는 기능 environmentCapable 인터페이스를 통해 사용가능
- bean들의 묶음이며, 환경이다.(test환경에서는 어떤 bean들을 쓰겠다, 실제 production에서는 이러이러한 bean들을 쓰겠다라는 환경)
- 각각의 환경에 따라 bean을 다르게 사용해야 할 경우, 또는 특정 환경에서만 어떠한 bean을 등록해야하는 경우에 사용한다.
- environmentCapable 인터페이스의 getEnvironment()를 사용하여 Environment를 가져올 수 있다.
- environment.getActiveProfile()을 통해 현재 active되어있는 프로파일을 가져올 수 있다.
<br>

#### profile 정의하는 방법
- 클래스에 정의
  * @Configuration @profile("test")
  * @Component @Profile("test")
- 메소드에 정의
  * @Bean @ Profile("test")
```java
@Component
public class AppRunner implements ApllicationRunner {

    @Autowired
    ApplicationContext ctx;

    @Override
    public void run(ApplicationArguments args) throws Exception {
       Environment environment = ctx.getEnvironment();
       System.out.println(Arrays.toString(environment.getActiveProfiles());  // 설정한게 없으면 []
       System.out.println(Arrays.toString(environment.getDefaultProfiles()); // Default
    }
}
```
- 아래 코드에서 test 라는 profile로 어플리케이션 실행하기 전까지는 아래의 설정이 적용이 안된다.
```java
@Configuration
@Profile("test")
public class TestConfiguration {

    @Bean
    public BookRepository bookRepository() {
        return new TestBookRepository();
    }
}
```
<br>

#### profile 설정 방법
1. Active profiles
- IDE에서 제공하는 Active profiles에 profile 추가
2. VM options
- 실습 기준 IntelliJ에서 Community 버젼의 경우 Active profiles가 없을 경우, VM option에 `-Dspring.profiles.active="profile이름"`으로 profile 추가
<br>

#### profile 표현 예시
- !(not), &(and), |(or) 모두 쓸 수 있다.
```java
@Repository
@Profile("test")
public class TestBookRepository implements BookRepository {
}
@Repository
@Profile("!Prod") //Prod가 아닌 profile
public class TestBookRepository implements BookRepository {
}
@Repository
@Profile("!Prod & test") //Prod이 아니면서 test인 profile
public class TestBookRepository implements BookRepository {
}
```
<br>

### Property
- 다양한 방법으로 정의할 수 있는 설정값이다.
- 애플리케이션에 등록된 여러가지 key-value 쌍으로 제공되는 property에 접근할 수 있는 기능
- 계층형으로 접근한다. (우선 순위가 있다)
- `@PropertySource`로 property를 추가할 수 있다. 
```java
@SpringBootApplication
@PropertySource("classpath:/app.properties") //Environment를 통해 property를 추가
public class Demospring51Application {
       
       public static void main(String[] args) {
            SpringApplication.run(Demospring51Application.class, args);
       }
}
```
- `environment.getProperty();` 를 사용해 propery를 얻을 수 있다. 
```java
@Component
public class AppRunner implements ApplicationRunner {

  @Autowired
  ApplicationContext ctx;
  
  @Autowired
  BookRepository bookRepository;
  
  @Override
  public void run(ApplicationArguments args) throws Exception {
       Environment environment = ctx.getEnvironment();
       System.out.println(environment.getProperty("app.name"));
       System.out.println(environment.getProperty("app.about"));
  }
}
```
<br>

### Property 우선 순위
- StandardServletEnvironment의 우선순위
  * ServletConfig 매개변수
  * ServletContext 매개변수
  * JNDI (java:comp/env/)
  * JVM 시스템 프로퍼티 (-Dkey="value")
  * JVM 시스템 환경 변수 (운영 체제 환경 변수)
  <br>

## MessageSource
- ApplicationContext가 상속받고 있는 인터페이스
- 국제화(i18n) 기능을 제공하는 인터페이스
- 스프링 부트에서는 message로 시작하는 properties들을 다 읽어주기 때문에 별다른 설정 필요없이 사용 가능하다.
```java
// messages.properties 파일
greeting=Hello {0}
```
```java
// messages_ko_KR.properties 파일
greeting=안녕, {0}
```java
@Component
public class AppRunner implements ApplicationRunner {
  @Autowired
  MessageSource messageSource;
  @Override
  public void run(ApplicationArguments args) throws Exception {
      System.out.println(messageSource.getMessage("greeting", new String[]{"keesun"}, Locale.KOREA));
      System.out.println(messageSource.getMessage("greeting", new String[]{"keesun"}, Locale.getDefault()));
    }
}
```
<br>

### Reloading 기능이 있는 메세지 소스 사용하기
- 변경 사항이 있으면 build 해주면 된다.
```java
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    MessageSource messageSource;
    @Override
    public void run(ApplicationArguments args) throws Exception {
        while (true) {
            System.out.println(messageSource.getMessage("greeting", new String[]{"keesun"}, Locale.KOREA));
            System.out.println(messageSource.getMessage("greeting", new String[]{"keesun"}, Locale.getDefault()));
            Thread.sleep(1000l);  // 1초에 한번씩 출력
        }
    }
}

```
```java
@SpringBootApplication
public class Demospring51application {

    public static void main(String[] args) {
        SpringApplication.run(Demospring51application.class, args);

    }

    @Bean
    public MessageSource messageSource() {
        var messageSource = new ReloadableResourceBundleMessageSource();
        messageSource.setBasename("classpath:/messages");
        messageSource.setDefaultEncoding("UTF-8");
        messageSource.setCacheSeconds(3); //캐싱하는 시간을 최대 3초까지만 캐싱하고 다시 읽음
        return messageSource;
    }
}
```
<br>

## ApplicationEventPublisher
- ApplicationContext가 상속받고 있는 인터페이스
- 옵저버 패턴의 구현체로, 이벤트 기반의 프로그래밍을 할 때 유용하다.

### 이벤트 만들기
```java
// 이벤트는 bean이 아니다.
// spring 4.2 이후부터는 이벤트가 ApplicationEvent를 상속받지 않아도 된다.
public class MyEvent {

  private int data;
  private Object source;

  public MyEvent(Object source, int data) {
      this.source = source;
      this.data = data;
  }

  public Object getSource() {
      return source;
  }

  public int getData() {
      return data;
  }
}
```
<br>

### 이벤트 발생 시키기
```java
// 이벤트를 발생시키는 로직
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    ApplicationEventPublisher publisherEvent;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        publisherEvent.publishEvent(new MyEvent(this, 100));
    }
}
```
<br>

### 이벤트 처리
```java
// 이벤트를 받아서 처리하는 eventHandler(bean으로 등록되어야 한다)
@Component
public class MyEventHandler {

    @EventListener
    public void handle(MyEvent event) {
        System.out.println("이벤트 받았다. 데이터는 " + event.getData());
    }
}
```
- Handler가 추가된 경우
  * 순차적으로 실행된다. (동시에 다른 thread에서 실행되는게 아니라 같은 thread에서 순차적으로)
```java
@Component
public class AnotherHandler {

    @EventListener
    public void handler(MyEvent event) {
        System.out.println("Another " + event.getData());
    }
}
```
- 순서를 정해줄 수도 있다. 
  * `@Order`
  * ex) `@Order(Ordered.HIGHEST_PRECEDENCE + 2)`
- 비동기적으로 실행
  * `@Async` 
  * 비동기적으로 실행할 때는 각각의 쓰레드 풀에서 따로 놀고, 어느 것이 먼저 실행 될지는 쓰레드 스케쥴링에 따라 달라지기 때문에 Order가 더이상 의미 없다.
  * Application 파일에도 `@EnableAsync`를 추가하면 Async하게 동작한다.
<br>

### spring이 제공해주는 ApplicationContext 관련된 기본 이벤트
- ContextRefreshedEvent
  * ApplicationContext를 초기화 했더나 리프래시 했을 때 발생.
- ContextStartedEvent
  * ApplicationContext를 start()하여 라이프사이클 빈들이 시작 신호를 받은 시점에 발생.
- ContextStoppedEvent
  * ApplicationContext를 stop()하여 라이프사이클 빈들이 정지 신호를 받은 시점에 발생.
- ContextClosedEvent
  * ApplicationContext를 close()하여 싱글톤 빈 소멸되는 시점에 발생.
- RequestHandledEvent
  * HTTP 요청을 처리했을 때 발생.
- 예시 코드
```java
@Component
public class MyEventHandler {
    @EventListener
    public void handle(ContextRefreshedEvent event) {
        System.out.println("ContextRefreshedEvent");
    }
    @EventListener
    public void handle(ContextClosedEvent event) {
        System.out.println("ContextClosedEvent");
    }
}
```
<br>

## ResourceLoader
- ApplicationContext가 상속받고 있는 인터페이스
- 리소스를 읽어오는 기능을 제공한다.
```java
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    ResourceLoader resourceLoader;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Resource resource = resourceLoader.getResource("classpath:test.txt"); // 파일 읽어오기
        System.out.println(resource.exists()); // 파일 유무 
        System.out.println(resource.getDescription()); // 설명
        // URI를 받아와서 Path를 만들고 Path에 해당하는 파일을 읽어서 컨텐츠를 출력 
        System.out.println(Files.readString(Path.of(resource.getURI()))); // URL로 읽어오기
    }
}
/* 실행결과
true
class path resource [test.txt]
hello spring
*/
```
<br>

## Resource 추상화
- java.net.URL을 추상화 한 것.
- 스프링 내부에서 많이 사용하는 인터페이스
<br>

### 추상화 한 이유
- 클래스패스 기준으로 리소스 읽어오는 기능 부재
- ServletContext를 기준으로 상대 경로로 읽어오는 기능 부재
-  새로운 핸들러를 등록하여 특별한 URL 접미사를 만들어 사용할 수는 있지만 구현이 복잡하고 편의성 메소드가 부족하다.
<br>

### 주요 구현체
- UrlResource : 기본으로 지원하는 프로토콜 http, https, ftp, file, jar.
- ClassPathResource: 지원하는 접두어 classpath:
- FileSystemResource
- ServletContextResource: 웹 애플리케이션 루트에서 상대 경로로 리소스 찾는다.
  * 읽어들이는 resource타입이 ApplicationContext 타입에 따라 결정되기 때문에 가장 많이 쓴다.
<br>

### 리소스 읽어오기
- Resource의 타입은 location 문자열과 **ApplicationContext의 타입에 따라** 결정된다.
  * ClassPathXmlApplicationContext -> ClassPathResource
  * FileSystemXmlApplicationContext -> FileSystemResource
  * WebApplicationContext -> ServletContextResource
- **ApplicationContext의 타입에 상관없이 리소스 타입을 강제하려면 java.net.URL 접두어(+classpath:) 중 하나를 사용할 수 있다.**
  * **classpath:**me/whiteship/config.xml -> ClassPathResource
  * **file:///**some/resource/path/config.xml -> FileSystemResource
  * 이렇게 접두어가 있으면 명시적이다. 
- 접두어를 사용하지 않으면 ServletContextResource로 Resolve 된다.
- 루트는 ///를 사용하고 와일드 카드나 classpath*도 사용할 수도 있다.