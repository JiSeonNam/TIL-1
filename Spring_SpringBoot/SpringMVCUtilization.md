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

## 요청 맵핑하기 3. 미디어 타입

### 특정한 타입의 데이터를 담고 있는 요청만 처리하는 핸들러
- consumes를 사용하여 원하는 타입의 데이터 요청만 처리할 수 있다.
- ex) `@RequestMapping(value = "/hello", consumes = MediaType.APPLICATION_JSON_UTF8_VALUE)`
    * 문자열(application/json)을 입력하는 대신 MediaType을 사용하면 상수를 (IDE에서) 자동 완성(MediaType.APPLICATION_JSON_UTF8_VALUE)으로 사용할 수 있다.
- Content-Type 헤더로 필터링한다.
- 매치 되는 않는 경우에 415(Unsupported Media Type) 에러가 난다.
<br>

### 특정한 타입의 응답을 만드는 핸들러
- produces를 사용하여 원하는 응답을 설정할 수 있다.
- ex) `@RequestMapping(produces=”application/json”)`
    * 문자열 대신 자동완성으로 사용 가능하다.
- Accept 헤더로 필터링한다.
    * accept type이 없는 경우에는 content type이 반드시 맞아야 하는게 아니라 다 맵핑이 된다.
- 매치 되지 않는 경우에 406(Not Acceptable) 에러가 난다.
<br>

### 클래스에 선언한 @RequestMapping과 조합
- 클래스에 선언한 @RequestMapping에 사용한 것과 조합이 되지 않고 메소드에 사용한 @RequestMapping의 설정으로 덮어쓴다(Overriding).
<br>

### 미디어 타입 실습
- Not (!)을 사용해서 특정 미디어 타입이 아닌 경우로 맵핑 할 수도 있다.
```java
@Controller
@RequestMapping(consumes = MediaType.APPLICATION_XML_VALUE) // 클래스에 선언해도 밑의 메소드에 선언한 설정으로 덮어쓴다.
public class SampleController {

    @RequestMapping("/hayoung")
    @GetMapping(
            value = "/hello",
            consumes = MediaType.APPLICATION_JSON_UTF8_VALUE, // 요청 설정
            produces = MediaType.TEXT_PLAIN_VALUE   // 응답 설정
    )
    @ResponseBody
    public String hello() {
        return "hello";
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
                    .contentType(MediaType.APPLICATION_JSON)
                    .accept(MediaType.APPLICATION_JSON))
                .andDo(print())
                .andExpect(status().isOk())
        ;
    }
}
```
<br>

## 요청 맵핑하기 4. 헤더와 매개변수
- 특정한 헤더가 있는 요청을 처리하고 싶은 경우
    * `@RequestMapping(headers = “key”)`
    * ex) `@GetMapping(value = "/hello", headers = HttpHeaders.AUTHORIZATION)`
- 특정한 헤더가 없는 요청을 처리하고 싶은 경우
    * `@RequestMapping(headers = “!key”)`
    * ex) `@GetMapping(value = "/hello", headers = "!" + HttpHeaders.FROM)`
- 특정한 헤더 키/값이 있는 요청을 처리하고 싶은 경우
    * `@RequestMapping(headers = “key=value”)`
    * ex) `@GetMapping(value = "/hello", headers = HttpHeaders.AUTHORIZATION + "=" + "111")`
- 특정한 요청 매개변수 키를 가지고 있는 요청을 처리하고 싶은 경우
    * `@RequestMapping(params = “a”)`
    * ex) `@GetMapping(value = "/hello", params = "name")`
- 특정한 요청 매개변수가 없는 요청을 처리하고 싶은 경우
    * `@RequestMapping(params = “!a”)`
- 특정한 요청 매개변수 키/값을 가지고 있는 요청을 처리하고 싶은 경우
    * `@RequestMapping(params = “a=b”)`
    * ex) `@GetMapping(value = "/hello", params = "name=hayoung")`
<br>

## 요청 맵핑하기 5. HEAD와 OPTIONS 요청 처리
- 직접 구현하지 않아도 스프링 웹 MVC에서 제공해주는 기능의 HTTP Method
    * HEAD
    * OPTIONS
- 참고 - [HTTP Method Definitions](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html)
<br>

### HEAD
- GET 요청과 동일하지만 응답 본문을 받아오지 않고 응답 헤더만 받아온다.
<br>

### OPTIONS
- 서버 또는 URI에 해당하는 특정 리소스가 제공하는 기능을 확인할 수 있다.
- 사용할 수 있는 HTTP Method 제공해 준다.
- 서버는 Allow 응답 헤더에 사용할 수 있는 HTTP Method 목록을 제공한다.
<br>

## 요청 맵핑하기 6. 커스텀 애노테이션 
- 예를 들어 `@RequestMapping` 애노테이션을 메타 애노테이션으로 사용해 `@GetMapping` 같은 커스텀한 애노테이션을 만들 수 있다.
<br>

### 메타(Meta) 애노테이션
- 애노테이션에 사용할 수 있는 애노테이션
- 스프링이 제공하는 대부분의 애노테이션은 메타 애노테이션으로 사용할 수 있다.
<br>

### 조합(Composed) 애노테이션
- 한개 혹은 여러 메타 애노테이션을 조합해서 만든 애노테이션
- 코드를 간결하게 줄일 수 있다.
- 보다 구체적인 의미를 부여할 수 있다.
<br>

### @Retention
- 해당 애노테이션 정보를 언제까지 유지할 것인가.
    * 기본값은 Class이다.
- Source
    * 소스 코드까지만 유지.
    * 컴파일 하면 해당 애노테이션 정보는 사라진다.
- Class
    * 컴파인 한 .class 파일에도 유지.
    * 런타임 시, 클래스를 메모리로 읽어오면 해당 정보는 사라진다.
- Runtime
    * 클래스를 메모리에 읽어왔을 때까지 유지
    * 코드에서 이 정보를 바탕으로 특정 로직을 실행할 수 있다.
<br>

### @Target
- 해당 애노테이션을 어디에 사용할 수 있는지 결정한다.
<br>

### @Documented
- 해당 애노테이션을 사용한 코드의 문서에 그 애노테이션에 대한 정보를 표기할지 결정한다.
<br>

### 커스텀 애노테이션 만들기 실습
- hello라는 요청 맵핑을 많이 해야 할 경우 커스텀 애노테이션 `@GetHelloMapping` 만들기
```java
@Documented
@Target(ElementType.METHOD) // 애노테이션을 Method에 사용할 수 있다고 설정
@Retention(RetentionPolicy.RUNTIME) // 애노테이션 정보 유지 설정
// @GetMapping은 메타 애노테이션이 아니므로 @RequestMapping을 써야한다.
@RequestMapping(method = RequestMethod.GET, value = "/hello")
public @interface GetHelloMapping {
}
```
```java
@Controller
public class SampleController {

    @GetHelloMapping
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
                .andDo(print())
                .andExpect(status().isOk())
        ;
    }
}
```
<br>

## 핸들러 메소드 1. 지원하는 메소드 아규먼트와 리턴 타입

### 핸들러 메소드 아규먼트
- 주로 요청 그 자체 또는 요청에 들어있는 정보를 받아오는데 사용한다.

| 핸들러 메소드 아규먼트 | 설명 |
|:--------|:--------|
| WebRequest<br>NativeWebRequest<br>ServletRequest(Response)<br>HttpServletRequest(Response) | 요청 또는 응답 자체에 접근 가능한 API |
| InputStream<br>Reader<br>OutputStream<br>Writer | 요청 본문을 읽어오거나, 응답 본문을 쓸 때 사용할 수 있는 API |
| PushBuilder | 스프링 5, HTTP/2 리소스 푸쉬에 사용 |
| HttpMethod | GET, POST, ... 등에 대한 정보 |
| Locale<br>TimeZone<br>ZoneId | LocaleResolver가 분석한 요청의 Locale 정보 |
| @PathVariable | URI 템플릿 변수 읽을 때 사용 |
| @MatrixVariable | URI 경로 중에 키/값 쌍을 읽어 올 때 사용 |
| @RequestParam | 서블릿 요청 매개변수 값을 선언한 메소드 아규먼트 타입으로 변환해준다. <br> 단순 타입인 경우에 이 애노테이션을 생략할 수 있다. |
| @RequestHeader | 요청 헤더 값을 선언한 메소드 아규먼트 타입으로 변환해준다. |
|...|...|
<br>

- [Handler Methods - Method Arguments](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-ann-arguments)
<br>

### 핸들러 메소드 리턴
- 주로 응답 또는 모델을 랜더링할 뷰에 대한 정보를 제공하는데 사용한다.

| 핸들러 메소드 리턴 | 설명 |
|:--------|:--------|
| @ResponseBody | 리턴 값을 HttpMessageConverter를 사용해 응답 본문으로 사용한다. |
| HttpEntity<br> ReponseEntity | 응답 본문 뿐 아니라 헤더 정보까지, 전체 응답을 만들 때 사용한다. | 
| String | ViewResolver를 사용해서 뷰를 찾을 때 사용할 뷰 이름. |
| View | 암묵적인 모델 정보를 랜더링할 뷰 인스턴스 |
| Map<br>Model | (RequestToViewNameTranslator를 통해서) 암묵적으로 판단한 뷰 랜더링할 때 사용할 모델 정보 |
| @ModelAttribute | (RequestToViewNameTranslator를 통해서) 암묵적으로 판단한 뷰 랜더링할 때 사용할 모델 정보에 추가한다.<br> 이 애노테이션은 생략할 수 있다. |
<br>
