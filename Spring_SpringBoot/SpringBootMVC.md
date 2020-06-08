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