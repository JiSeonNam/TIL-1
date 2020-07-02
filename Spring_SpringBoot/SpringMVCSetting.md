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
