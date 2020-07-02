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
