> [강의 링크](https://www.inflearn.com/course/spring-framework_core)
 
 # 스프링 프레임워크 핵심 기술

 ## IoC 컨테이너와 빈
- Inversion of Control : 의존 관계 주입(Dependency Injection)이라고도 하며, 어떤 객체가 사용하는 의존 객체를 직접 만들어 사용하는게 아니라, 주입 받아 사용하는 방법을 말한다.

### 스프링 IoC 컨테이너
- [BeanFactory](https://docs.spring.io/spring-framework/docs/5.0.8.RELEASE/javadoc-api/org/springframework/beans/factory/BeanFactory.html) : 스프링 IoC 컨테이너의 최상위 인터페이스 
- ApplicationContext
    * BeanFactory를 상속받고 있기 때문에 BeanFactory의 모든 기능을 포함하며, 추가 기능도 제공되기 때문에 일반적으로 BeanFactory보다 추천된다. 
    * 메세지 소스 기능(i18n)
    * 이벤트 발행 기능
    * 리소스 로딩 기능 등
- 애플리케이션 컴포넌트의 중앙 저장소.
- 빈 설정 소스로 부터 빈 정의를 읽어들이고, 빈을 구성하고 제공한다.

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


