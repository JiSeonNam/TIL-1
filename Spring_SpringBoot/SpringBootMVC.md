# 스프링 부트 웹 MVC 

## 소개
- 간단한 컨트롤러와 테스트 코드를 작성해보면 아래의 테스트에서 아무런 설정파일을 작성하지 않았지만 스프링 MVC 기능을 사용할 수 있다.
- 이것은 스프링 부트가 제공해주는 기본 설정 덕분이다.
    * spring-boot-autoconfigure -> spring.factories -> WebMvcAutoConfiguration 클래스
```java
@RunWith(SpringRunner.class)
@WebMvcTest(UserControllerTest.class)   //@WebMvcTest 애노테이션을 사용하면 MockMvc를 주입받아서 사용할 수 있다.
public class UserControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void hello() throws Exception {
        mockMvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string("hello"));

    }
}
```
```java
@RestController
public class UserController {

    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
}
```
<br>

### 스프링 부트가 제공해주는 Web MVC 기능의 확장, 재정의
- 기본적으로 제공해주는 기능에서 추가적인 설정이 필요한 경우 아래 코드와 같은 설정 파일을 만들면 된다.
- 인터페이스 WebMvcConfigurer 제공하는 여러 콜백 메소드를 사용해 커스터마이징을 하면 된다. 
- 이 때, 확장을 하고싶은 경우 `@EnableWebMvc`를 붙이면 안된다. 
- 붙이면 스프링 부트가 제공하는 모든 웹 MVC 기능들이 사라지고 직접 웹 MVC 관련 설정을 해야한다.
- 반대로 `@EnableWebMvc`를 붙여 재정의 할 수 있다.   
```java
@Configuration
// @EnableWebMvc 사용 하면 안된다.
public class WebConfig implements WebMvcConfigurer {
    ...
}
```
<br>

## [HttpMessageConverters](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-config-message-converters)
- 스프링 프레임 워크에서 제공하는 인테페이스로 스프링 MVC의 일부분.
- HTTP 요청 본문을 객체로 변경하거나, 객체를 HTTP 응답 본문으로 변경할 때 사용.
- 어떤 요청을 받았는지, 응답을 보내는지에 따라서 메세지컨버터가 달라진다. 
    * 요청이 JSON 이고, 본문도 JSON이면 JSON MessageConverter가 사용된다.
- 주로 @RequestBody, @ResponseBody와 같이 사용한다. 
    * @RestController가 붙어있으면 @ResponseBody를 생략 가능하다.
```java
@RunWith(SpringRunner.class)
@WebMvcTest(UserController.class)
public class UserControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void createUser_JSON() throws Exception {
        String userJson = "{\"username\":\"hayoung\", \"password\":\"123\"}";
        //요청을 만드는 단계
        mockMvc.perform(post("/users/create")
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .accept(MediaType.APPLICATION_JSON_UTF8)
                .content(userJson))
                //응답 확인단계
                    .andExpect(status().isOk())
                    .andExpect(jsonPath("$.username", is(equalTo("hayoung"))))
                    .andExpect(jsonPath("$.password", is(equalTo("123"))));

    }
}
```
```java
@RestController
public class UserController {

    @GetMapping("/hello")
    public String hello(){
        return "hello";
    }

    @PostMapping("/users/create")
    public User create(@RequestBody User user){
        return user;
    }
}
```
```java
public class User {
    private Long id;
    private String username;
    private String password;

    ... getter and setter ...
}
```
<br>

## ViewResolver
- 스프링부트에 등록 되어있는 스프링 웹 MVC의 ContentNegotiatingViewResolver가 어떤 contentType일 때 어떤 응답을 보내고, accept header 요청에 의해서 해당 요청에 맞는 응답을 보내는 작업을 알아서 해준다.
- ContentNegotiatingViewResolver는 ViewResolver 중의 하나로 들어오는 요청에 accept header에 따라 응답이 달라진다.
    * accept header : 브라우저 또는 클라이언트가 어떠한 타입의 응답을 원한다고 서버에게 알려주는 것.
    * 어떠한 요청이 들어오면 응답을 만들어 낼 수 있는 모든 View를 찾아내고, View의 타입을 Accept Header랑 비교를 해서 최종적으로 선택을 하고 리턴한다.
    * 판단하기 가장 좋은 정보는 Accept Header이지만 경우에 따라 Accept Header를 제공하지 않는 경우도 있다.
    * Accept Header가 없는 경우 format이라는 parameter를 사용하여 /path?format=pdf와 같은 형식으로 알 수 있다.
- 요청은 JSON으로 보내고 응답은 XML로 받으려면 다음과 같이 코드를 작성하면 된다.
```java
@Test
public void createUser_XML() throws Exception {
    String userJson = "{\"username\":\"hayoung\", \"password\":\"123\"}";
    mockMvc.perform(post("/users/create")
            .contentType(MediaType.APPLICATION_JSON_UTF8)
            .accept(MediaType.APPLICATION_XML)  // accept header를 XML로 변경
            .content(userJson))
                .andExpect(status().isOk())
                .andExpect(xpath("/User/username").string("hayoung")) // jsonPath가 아니라 xpath로 확인
                .andExpect(xpath("/User/password").string("123"));

}
```
- 테스트를 돌리면 406 HttpMediaTypeNotAcceptableException 에러가 발생한다.
<br>

### 해결 방법
- HttpMessageConverter는 HttpMessageConvertersAutoConfiguration으로 인해서 적용된다.
- JacksonHttpMessageConvertersConfiguration 클래스에는 MappingJackson2XmlHttpMessageConverterConfiguration이라는 XML을 converting 해주는 converter가 있고
- MappingJackson2XmlHttpMessageConverterConfiguration 클래스에는 `@ConditionalOnClass(XmlMapper.class)`으로 XmlMapper클래스가 classpath에 존재해야만 등록되도록 정의되어 있다. 
- 406 에러가 발생한 이유는 XML로 변환할 수 있는 컨버터가 등록되어있지 않기 때문이다.
- XML 메세지 converter 추가하기
    * pom.xml에 의존성을 추가하면 된다.
```html
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
    <version>2.9.8</version>
</dependency>
```
<br>
