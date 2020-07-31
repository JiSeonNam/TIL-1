# 스프링 MVC 설정

## 스프링 MVC 빈 설정

### 스프링 MVC 구성 요소 직접 빈으로 등록하기
- 스프링 MVC 구성 요소를 설정하지 않아도 DispatcherServlet.properties에 있는 기본 전략을 사용하게 된다. 
- 이 기본 전략을 사용하게 되면 클래스들이 가지고 있는 기본값이 적용된다.
    * 예를 들어 InternalResourceViewResolver는 prefix와 suffix를 설정할 수 있지만 둘 다 없는 상태로 사용이 된다.
- 이 때, 원한다면 `@Configuration`을 사용한 자바 설정 파일에 직접 `@Bean`을 사용해서 등록할 수 있다. 
- 예시 코드
```java
@Configuration
@ComponentScan
public class WebConfig {

    @Bean
    public HandlerMapping handlerMapping() {
        RequestMappingHandlerMapping handlerMapping = new RequestMappingHandlerMapping();
        //가장 흔히 사용되는 설정 중의 하나, Servlet Filter와 유사한 기능
        // RequestMappingHandlerMapping은 애노테이션과 관련있다.
        handlerMapping.setInterceptors();
        handlerMapping.setOrder(Ordered.HIGHEST_PRECEDENCE);
        return handlerMapping;
    }

    @Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setPrefix("/WEB-INF/");
        viewResolver.setSuffix(".jsp");
        return viewResolver;
    }
}
```
- 이런 식으로 bean설정을 직접하는 것은 low-level이다. 좀 더 편한 방법이 있으므로 일일히 설정할 수 있다는 정도만 알면 된다.(스프링 부트가 나오기 이전에도 이렇게 하진 않았다)
<br>

## @EnableWebMvc
- 애노테이션 기반의 스프링 MVC를 사용할 때 일일히 bean으로 등록하지 않도록 편리한 기능을 제공한다.
- `@Configuration`을 사용한 자바 설정 파일에 `@EnableWebMvc`를 추가하면 된다.
- 또한 WebApplicationContext에 ServletContext 설정을 해줘야 한다. 
    * `context.setServletContext(servletContext);`
    * `@EnableWebMvc`가 불러오는 DelegatingWebMvcConfiguration가 servletContext를 참조하기 때문에 꼭 해줘야 한다.
```java
@Configuration
@EnableWebMvc
public class WebConfig{

}
```
<br>

## WebMvcConfigurer
- `@EnableWebMvc`가 제공하는 빈을 커스터마이징할 수 있는 기능을 제공하는 인터페이스
- `@EnableWebMvc`를 사용할 때 import하는 `DelegatingWebMvcConfiguration`은 Delegation구조(어딘가에 위임해서 읽어오는 방식)으로 구성되어 있다.
    * 따라서 처음부터 bean을 등록하는게 아니라 손쉽게 `DelegatingWebMvcConfiguration`가 상속하는 `WebMvcConfigurationSupport`가 설정해주는 bean에 추가하고 싶은 부분을 추가할 수 있다.
    * 원하는 데로 커스텀할 수 있도록 확장성이 좋다  
- 이런 확장을 인터페이스를 통해서 지원하고 있는데 그 인터페이스가 WebMvcConfigurer이다. 
- 예를 들어 ViewResolver를 직접 bean으로 등록하지 않아도 @EnableWebMvc가 등록해주는 viewResolver를 커스터마이징하면서 같은 결과를 얻을 수 있다.
```java
@Configuration
@ComponentScan
@EnableWebMvc
public class WebConfig {
public class WebConfig implements WebMvcConfigurer {
    /* 직접 구현하지 않아도 된다.
    @Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setPrefix("/WEB-INF/");
        viewResolver.setSuffix(".jsp");
        return viewResolver;
    }
    */
    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        // prefix: /WEB-INF/, suffix: .jsp
        registry.jsp("/WEB-INF/", ".jsp");
    }
    // Formatter 추가
    @Override
    public void addFormatters(FormatterRegistry registry) {

    }
    // Interceptor 추가
    @Override
    public void addInterceptors(InterceptorRegistry registry) {

    }
}
```
<br>

## 스프링 부트의 스프링 MVC 설정

### 스프링 부트의 “주관”이 적용된 자동 설정이 동작한다.
- JSP 보다 Thymeleaf를 선호한다.
- JSON 지원 (XML은 지원하지 않는다)
- 정적 리소스 지원 (+ 웰컴 페이지, 파비콘 등 지원)
<br>

### 스프링 MVC 커스터마이징
- application.properties
    * properties들을 고치는 방법. 
    * 가장 간단하다.
- @Configuration + Implements WebMvcConfigurer
    * 스프링 부트의 스프링 MVC 자동설정 + 추가 설정
    * 대부분의 경우에 합리적인 선택이다.
    * 일일히 설정하기 보다는 스프링 부트가 제공하는 설정을 사용하면서 추가로 설정하는 방법이다.
- @Configuration + @EnableWebMvc + Imlements WebMvcConfigurer
    * 스프링 부트의 스프링 MVC 자동설정을 사용하지 않음.
<br>

## 스프링 부트와 JSP

### 스프링 부트에서 JSP 사용하기
- 제약사항
    * 프로젝트를 JAR 프로젝트로 만들 수 없고, WAR 프로젝트로 만들어야 한다.
    * Java -JAR로 실행할 수는 있지만 “실행가능한 JAR 파일”은 지원하지 않는다.
    * Undertow(JBoss에서 만든 서블릿 컨테이너)는 JSP를 지원하지 않는다.
    * Whitelabel 에러 페이지를 error.jsp로 오버라이딩 할 수 없다.
- 스프링 부트 Document에도 가급적이면 JSP 사용을 피하라고 나와있다. 
> "If possible, JSPs should be avoided. There are several known limitations when using them with embedded servlet containers."
- 의존성 추가
```xml
<dependency>
<groupId>javax.servlet</groupId>
<artifactId>jstl</artifactId>
</dependency>

<dependency>
<groupId>org.apache.tomcat.embed</groupId>
<artifactId>tomcat-embed-jasper</artifactId>
<scope>provided</scope>
</dependency>
```
- Event 클래스와 Controller 생성
```java
public class Event {

    private String name;

    private LocalDateTime starts;

    //...getter and setter...
}
```
```java
@Controller
public class EventController {

    @GetMapping("/events")
    public String getEvents(Model model) {
        Event event1 = new Event();
        event1.setName("스프링 웹 MVC 스터디 1");
        event1.setStarts(LocalDateTime.of(2019, 1, 15 ,10, 0));
        Event event2 = new Event();
        event2.setName("스프링 웹 MVC 스터디 2");
        event2.setStarts(LocalDateTime.of(2019, 1, 22 ,10, 0));

        List<Event> events = List.of(event1, event2);

        model.addAttribute("events", events);
        model.addAttribute("message", "Happy New Year!");

        return "events/list";
    }
}
```
- 뷰 만들기(JSP), jstl 태그 라이브러리도 선언
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <h1>이벤트 목록</h1>
    <h2>${message}</h2>
    <table>
        <tr>
            <th>이름</th>
            <th>시작</th>
        </tr>
        <c:forEach items="${events}" var="event">
            <tr>
                <td>${event.name}</td>
                <td>${event.starts}</td>
            </tr>
        </c:forEach>
    </table>
</body>
</html>
```
- application.properties에 prefix와 suffix 설정
```
spring.mvc.view.prefix=/WEB-INF/jsp/
spring.mvc.view.suffix=.jsp
```
<br>

## WebMvcConfigurer 설정 - Fomatter 설정

### Formatter
- 어떠한 객체를 문자열로 변환하거나 문자열을 객체로 변환할 때 사용할 수 있는 인터페이스
- Formatter를 사용하면 우리가 원하는 데이터를 객체로 받을 수 있다.
- Formatter는 2가지 인터페이스를 합친 것이다. 
    * Printer: 객체를 (Locale  정보를 참고하여) 문자열로 어떻게 출력할 것인가
    * Parser: 문자열을 (Locale 정보를 참고하여) 객체로 어떻게 변환할 것인가

### Formatter 실습
- Controller와 Test코드 추가
```java
@RestController
public class SampleController {

    @GetMapping("/hello/{name}")
    public String hello(@PathVariable("name") Person person) {
        return "hello" + person.getName();
    }
}
```
```java
@RunWith(SpringRunner.class)
@WebMvcTest
public class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void hello() throws Exception {
        this.mockMvc.perform(get("/hello"))
                .andDo(print())
                .andExpect(content().string("hello"));
    }
}
```
- name에 들어오는 문자열을 Person으로 변환해야하는지 Spring MVC는 모르기 때문에 Formatter 추가
```java
public class PersonFormatter implements Formatter<Person> {
    @Override
    public Person parse(String text, Locale locale) throws ParseException {
        Person person = new Person();
        person.setName(text);
        return person;
    }

    @Override
    public String print(Person person, Locale locale) {
        return person.toString();
    }
}
```
- 스프링 설정 파일에 Fomatter를 등록
    * WebMvcConfigurer 인터페이스를 구현할 때 FormatterRegistry를 제공하는 addFormatters 메소드를 구현하면 된다.
    * 이제는 스프링 MVC가 문자열을 Person이라는 객체로 어떻게 변환해야하는지 알고 있게 된다.
    * 애플리케이션을 실행하면 잘 동작한다. (localhost:8080/hello/hayoung)
```java
@Configuration
public class WebConfig implements WebMvcConfigurer{
    
    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addFormatter(new PersonFormatter());
    }
}
```
- url path뿐만 아니라 request parameter로도 잘 동작한다. (localhost:8080/hello?name=hayoung)
```java
@RestController
public class SampleController {

    @GetMapping("/hello")
    public String hello(@RequestParam("name") Person person) {
        return "hello " + person.getName();
    }
}
```
```java
@RunWith(SpringRunner.class)
@WebMvcTest
public class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void hello() throws Exception {
        this.mockMvc.perform(get("/hello")
                    .param("name", "hayoung"))
                .andDo(print())
                .andExpect(content().string("hello hayoung"));
    }
}
```

### 스프링 부트에서의 Formatter 
- 스프링 부트를 쓰면 굳이 스프링 설정 파일에 등록할 필요 없다.
- Formatter가 bean으로 등록되어 있으면 자동으로 등록해 준다.
- 하지만 Component로 bean으로 등록하면 애플리케이션을 잘 동작하지만 Test는 실패한다.
    * `@WebMvcTest`가 slicing-test용으로, 웹과 관련된 bean들만 등록해주기 때문에 Formatter를 인식하지 못한다.
    * `@WebMvcTest`를 `@SpringBootTest`로 통합 Test를 하면 된다.
    * mockMvc가 자동으로 bean으로 등록되지 않기 때문에 `@AutoConfigureMockMvc` 애노테이션도 Test에 붙여줘야 한다.
<br>

## 도메인 클래스 컨버터 자동 등록
- 스프링 데이터 JPA는 스프링 MVC용 도메인 클래스 컨버터를 제공한다. 
- 도메인 클래스 컨버터
    * 스프링 데이터 JPA가 제공하는 Repository를 사용해서 ID에 해당하는 엔티티를 읽어온다.

### 실습
- 예시 코드에서 만약 id에 해당하는 person의 이름을 출력하고 싶을 때는 Formatter나 Converter를 직접 만들어도 되지만 스프링 데이터 JPA의 도움을 받으면 쉽게 해결할 수 있다.
- 스프링 데이터 JPA, H2 의존성 추가
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
```
- 엔티티 맵핑
```java
@Entity
public class Person {
    @Id @GeneratedValue
    private Long id;

    private String name;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```
- Repository 추가
    * Id값에서 엔티티로 Converting을 할 때 Repository를 사용한다.
```java
// JpaRepository를 상속받으면서 첫번째 타입은 엔티티 타입(Person), 두번째 타입은 key값의 타입(Long)
// Reposiroty를 알아서 bean으로 등록해준다.
public interface PersonRepository extends JpaRepository<Person, Long> {

}
```
- 테스트
```java
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
public class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Autowired
    PersonRepository personRepository;

    @Test
    public void hello() throws Exception {
        Person person = new Person();
        person.setName("hayoung");
        Person savedPerson = personRepository.save(person);

        this.mockMvc.perform(get("/hello")
                    .param("id", savedPerson.getId().toString()))
                .andDo(print())
                .andExpect(content().string("hello hayoung"));
    }
}
```
<br>

## [HandlerInterceptor](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/HandlerInterceptor.html    )
- 핸들러 맵핑에 설정할 수 있는 인터셉터
    * 핸들러 맵핑이 찾아주는 핸들러에 인터셉터들을 적용해준다.
- 핸들러를 실행하기 전(preHandle), 요청 처리 후 뷰 랜더링 전(postHandler), 뷰 렌더링까지 끝난 완료시점(afterCompletion)까지 3가지 시점에 부가 작업을 하고 싶은 경우에 사용할 수 있다.
- 여러 핸들러에서 반복적으로 사용하는 코드를 줄이고 싶을 때 사용할 수 있다.
    * 로깅, 인증 체크, Locale 변경 등...
<br>

### boolean preHandle(request, response, handler)
- 핸들러 실행하기 전에 호출 된다.
- "핸들러"에 대한 정보를 사용할 수 있기 때문에 서블릿 필터에 비해 보다 세밀한 로직을 구현할 수 있다.
- 리턴값으로 계속 다음 인터셉터 또는 핸들러로 요청,응답을 전달할지(true) 응답 처리가 이곳에서 끝났는지(false) 알린다.
    * false를 리턴하면 요청처리가 끝나기 때문에 afterCompletion까지 가지 않는다.
<br>

### void postHandle(request, response, modelAndView)
- 핸들러 실행이 끝나고 아직 뷰를 랜더링 하기 이전에 호출 된다.
- “뷰"에 전달할 추가적이거나 여러 핸들러에 공통적인 모델 정보를 담는데 사용할 수도 있다.
    * ModelAndView를 커스터마이징하거나 뷰를 변경하는 것이 가능하다.
- 인터셉터 역순으로 호출된다.
- 비동기적인 요청 처리 시에는 호출되지 않는다
<br>

### void afterCompletion(request, response, handler, ex)
- 요청 처리가 완전히 끝난 뒤(뷰 랜더링 끝난 뒤)에 호출 됨
- preHandler에서 true를 리턴한 경우에만 호출 됨
- 인터셉터 역순으로 호출된다.
- 비동기적인 요청 처리 시에는 호출되지 않는다.
<br>

### HandlerInterceptor vs 서블릿 필터
- 서블릿 보다 구체적인 처리가 가능하다.
- 서블릿은 보다 일반적인 용도의 기능을 구현하는데 사용하는게 좋다.
<br>

### HandlerInterceptor 실습
- HandlerInterceptor 생성
```java
public class GreetingInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("preHandle 1");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle 1");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("afterCompletion 1");
    }
}
```
```java
public class AnotherInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("preHandle 2");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle 2");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("afterCompletion 2");
    }
}
```
- 설정 파일에 HandlerInterceptor 등록
    * 따로 설정하지 않으면 add한 순서대로 Interceptor가 적용된다.
    * 특정 패턴에 해당하는 요청에만 적용할 수도 있다.
    * order를 사용해 순서를 지정할 수 있다.
```java
@Configuration
public class WebConfig implements WebMvcConfigurer{
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 따로 설정하지 않으면 add한 순서대로 Interceptor가 적용된다.
        registry.addInterceptor(new GreetingInterceptor()).order(0);
        registry.addInterceptor(new AnotherInterceptor())
                .addPathPatterns("/hi")
                .order(-1);
    }
}
```
<br>

## [ResourceHandler](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/config/annotation/WebMvcConfigurer.html#addResourceHandlers-org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry-)
- 이미지, 자바스크립트, CSS 그리고 HTML 파일과 같은 정적인 리소스를 처리하는 핸들러이다.

### Default Servlet
- ResourceHandler는 Default Servlet을 이해해야 한다.
- Servlet 컨테이너가 기본으로 제공하는 Servlet으로 정적인 리소스를 처리할 때 사용한다.
- [참고 자료](https://tomcat.apache.org/tomcat-9.0-doc/default-servlet.html)
<br>

### 스프링 MVC ResourceHandler 맵핑 등록
- 정적인 ResourceHandler가 요청을 가로채면 직접 만든 Handler보다 먼저 요청을 찾아지기 때문에 가장 낮은 우선 순위로 등록된다.
    * 다른 핸들러 맵핑이 “/” 이하 요청을 처리하도록 허용하고
    * 최종적으로 ResourceHandler가 처리하도록 한다.
- DefaultServletHandlerConfigurer
<br>

### ResourceHandler 설정
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/mobile/**")   // 어떠한 패턴의 요청을 처리할 지
                .addResourceLocations("classpath:/mobile/") // 리소스를 어디서 찾을 지
                .setCacheControl(CacheControl.maxAge(10, TimeUnit.MINUTES));    // cache 설정, 리소스가 변경되면 10분이 안지났더라도 다시 받아온다.
    }
}
```
- 테스트
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SampleControllerTest {
    @Test
    public void helloStatic() throws Exception {
        this.mockMvc.perform(get("/mobile/index.html"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().string(Matchers.containsString("Hello Mobile")))
                .andExpect(header().exists(HttpHeaders.CACHE_CONTROL));
    }
}
```
- ResourceResolver: 요청에 해당하는 리소스를 찾는 전략
    * 캐싱, 인코딩(gzip, brotli), WebJar, ...
- ResourceTransformer: 응답으로 보낼 리소스를 수정하는 전략
    * 캐싱, CSS 링크, HTML5 AppCache, ...
- [참고 자료](https://www.slideshare.net/rstoya05/resource-handling-spring-framework-41)
<br>

\* 스프링 부트에서는 기본 정적 ResourceHandler와 캐싱 제공해준다.
<br>

## [HTTP Message Converter](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/config/annotation/WebMvcConfigurer.html#configureMessageConverters-java.util.List-)
- 요청 본문에서 메시지를 읽어들이거나(@RequestBody), 응답 본문에 메시지를 작성할 때(@ResponseBody) 사용한다.
- 예를 들어
    * 요청 본문에 들어있는 문자열을 변환하거나
    * 문자열이 JSON인 경우 객체로 변환하거나
    * 문자열이 XML인 경우 객체로 변환하거나
    * 문자열로 받거나 할 수 있다.

### 기본으로 등록해주는 HTTP 메시지 컨버터
- 바이트 배열 컨버터
- 문자열 컨버터
- Resource 컨버터
- Form 컨버터 (폼 데이터 to/from MultiValueMap<String, String>)
- (JAXB2 컨버터)
- (Jackson2 컨버터)
- (Jackson 컨버터)
- (Gson 컨버터)
- (Atom 컨버터)
- (RSS 컨버터) 등등
\* 괄호가 있는 컨버터는 classpath의 pom.xml파일에 해당 dependency가 있는 경우에만 등록된다.

### 문자열 변환 실습
```java
@RestController
public class SampleController {

    @GetMapping("/message")
    public String message(@RequestBody String body) {
        return body;
    }

}
```
```java
@RunWith(SpringRunner.class)
@WebMvcTest
public class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void stringMessage() throws Exception {
        this.mockMvc.perform(get("/message")
                .content("hello"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().string("hello"))
        ;
    }
}
```
### 설정 방법
- extendMessageConverters
    * 기본으로 등록해주는 컨버터에 새로운 컨버터 추가하기
- configureMessageConverters
    * 기본으로 등록해주는 컨버터는 다 무시하고 새로 컨버터 설정하기
- 의존성 추가로 컨버터 등록하기 (주로 사용되고 추천된다)
    * 메이븐 또는 그래들 설정에 의존성을 추가하면 그에 따른 컨버터가 자동으로 등록 된다.
    * WebMvcConfigurationSupport 클래스에서 판단 후 등록
<br>
