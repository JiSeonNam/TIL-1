# 스프링 MVC 활용

## 요청 맵핑하기 1. HTTP Method
HTTP 요청을 핸들러에 맵핑하는 방법
핸들러란 컨트롤러안에 있는 요청을 처리할 수 있는 메소드
### HTTP Method
- GET, POST, PUT, PATCH, DELETE, ...
<br>

### GET 요청
- 클라이언트가 서버의 리소스를 요청할 때 사용한다.
- 캐싱을 할 수 있다. (조건적인 GET으로 바뀔 수 있다.)
- 브라우저 기록에 남는다.
- 북마크 할 수 있다.
- URL에 보이므로 민감한 데이터를 보낼 때 사용하지 않는 것이 좋다. 
- Idempotent하다.
    * 동일한 GET요청일 경우 항상 동일한 응답을 return해야 한다.
<br>

### POST 요청
- 클라이언트가 서버의 리소스를 수정하거나 새로 만들 때 사용한다.
- 서버에 보내는 데이터를 POST 요청 본문에 담는다.
- 캐시할 수 없다.
- 브라우저 기록에 남지 않는다.
- 북마크 할 수 없다.
- 데이터 길이 제한이 없다.
- Idempotent하지 않을 수도 있다.
<br>

### PUT 요청
- URI에 해당하는 데이터를 새로 만들거나 수정할 때 사용한다.
- POST와 다른 점은 “URI”에 대한 의미가 다르다.
    * POST의 URI는 보내는 데이터를 처리할 리소스를 지칭하며
    * PUT의 URI는 보내는 데이터에 해당하는 리소스를 지칭한다.
- Idempotent하다.
<br>

### PATCH 요청
- PUT과 비슷하지만, 기존 엔티티와 새 데이터의 차이점만 보낸다는 차이가 있다.
- Idempotent하다.
<br>

### DELETE 요청
- URI에 해당하는 리소스를 삭제할 때 사용한다.
- Idempotent하다.
<br>

### 스프링 웹 MVC에서 HTTP method 맵핑하기 예시
- `@RequestMapping(method=RequestMethod.GET)`
- `@RequestMapping(method={RequestMethod.GET, RequestMethod.POST})`
- `@GetMapping`, `@PostMapping`, ...
<br>

### HTTP Method 실습
```java
@Controller
// @RequestMapping을 클래스 레벨로 설정할 수 있다.
@RequestMapping(method = RequestMethod.GET) // 모든 핸들러에서 HTTP Method : GET만 처리한다.
public class SampleController {

    @RequestMapping("/hello")
    @ResponseBody
    public String hello() {
        return "hello";
    }
}
// @RequestMapping으로 원하는 Method(GET)를 줘도 되고, 간단하게 @GetMapping을 써도 된다.
```
- 테스트
```java
@RunWith(SpringRunner.class)
@WebMvcTest
public class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void helloTest() throws Exception {
        mockMvc.perform(get("/hello"))
                .andDo(print())
                .andExpect(status().isOk())
        ;

        mockMvc.perform(put("/hello"))
                .andDo(print())
                .andExpect(status().isMethodNotAllowed())
        ;

        mockMvc.perform(post("/hello"))
                .andDo(print())
                .andExpect(status().isMethodNotAllowed())
        ;
    }
} 
```
<br>

## 요청 맵핑하기 2. URI 패턴 맵핑

### 요청 식별자로 맵핑하기
- `@RequestMapping`은 다음과 같은 패턴을 지원한다.
    * ? : 한 글자 ex) /author/??? => /author/123
    * *: 여러 글자 ex) /author/* => /author/hayoung
    * ** : 여러 패스 ex) /author/** => /author/hayoung/book
- 패턴이 중복되는 경우에는 가장 구체적으로 맵핑되는 핸들러를 사용하게 된다.
    * 밑의 코드에서 URI를 `"/hello/hayoung"`이라고 보내면 두 핸들러 모두 해당되지만 helloHayoung()이 맵핑된다.
```java
@Controller
@RequestMapping("/hello")
public class SampleController {
    @RequestMapping("/hayoung")
    @ResponseBody
    public String helloHayoung() {
        return "hello hayoung";
    }

    @RequestMapping("/**")
    @ResponseBody
    public String hello() {
        return "hello";
}
```
- 테스트 
```java
@RunWith(SpringRunner.class)
@WebMvcTest
public class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void helloTest() throws Exception {
        mockMvc.perform(get("/hello"))
        mockMvc.perform(get("/hello/hayoung"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().string("hello hayoung"))
                //핸들러 확인
                .andExpect(handler().handlerType(SampleController.class)) 
                .andExpect(handler().methodName("helloHayoung")) // 핸들러의 메소드 이름이 helloHayoung일 것이다.
        ;
    }
}
```
<br>

### 클래스에 선언한 @RequestMapping과 조합
- 클래스에 선언한 URI 패턴과 이어서 맵핑할 수도 있다.
- 예를 들어 `@RequestMapping("/hello/**")`를 다음 코드와 같이 작성할 수 있다.
```java
@Controller
@RequestMapping("/hello")
public class SampleController {

    @RequestMapping("/**")
    @ResponseBody
    public String hello() {
        return "hello";
    }
}
```

### 정규 표현식으로 맵핑하기
- `/{name:정규식}`
```java
@Controller
@RequestMapping("/hello")
public class SampleController {

    @RequestMapping("/{name:[a-z]+}")   // a~z에 해당하는 문자열 패턴이 올 수 있다. name으로 받겠다.
    @ResponseBody
    // @PathVariable를 사용해 받을 수 있다. ex) hello hayoung 등
    public String hello(@PathVariable String name) {
        return "hello" + name;
    }
}
```
- 테스트 
```java
@RunWith(SpringRunner.class)
@WebMvcTest
public class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void helloTest() throws Exception {
        mockMvc.perform(get("/hello"))
        mockMvc.perform(get("/hello/hayoung"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().string("hello hayoung"))
        ;
    }
}
```
<br>

### URI 확장자 맵핑 지원
- 기본적으로 스프링 MVC가 지원하지만 권장되지 않는다.
    * 보안 이슈 ([RFD Attack](https://www.trustwave.com/en-us/resources/blogs/spiderlabs-blog/reflected-file-download-a-new-web-attack-vector/))
    * URI 변수, Path 매개변수, URI 인코딩을 사용할 때 할 때 불명확하다.
- 예를 들어 `RequestMapping("/hayoung")` 이라고 설정하면 스프링이 알아서 자동으로 `RequestMapping({"/hayoung", "/hayoung.*})` 이러한 맵핑을 해준다.
    * hayoung.html, hayoung.json, hayoung.xml 등의 요청도 처리할 수 있게 등록해준다.
- 스프링 부트는 기본으로 이 기능을 사용하지 않도록 설정 해준다.
- 최근에는 URI에 확장자 패턴을 쓰는 것이 아니라 accept header에 표현한다.
    * parameter를 사용할 수도 있다. ex) `RequestMapping("/hayoung?type=xml")`
<br>
