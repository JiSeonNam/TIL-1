# 스프링 MVC 설정

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

