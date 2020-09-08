## 2. 스프링부트에서 테스트 코드 작성하기
- Application 클래스 생성
```java
@SpringBootApplication  // 스프링 부트의 자동 설정, 스프링 Bean읽기 및 생성이 자동으로 설정된다.
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args); // 내장 WAS 실행(톰캣을 설치할 필요 없고 Jar파일로 실행하면 된다.)
    }
} 
```
<br>

### 2-1. HelloController 테스트 코드 작성하기
- HelloController 클래스 생성
    * `@RestController`
        - 컨트롤러를 JSON을 반환하는 컨트롤러로 만들어준다.
        - 각 메소드마다 `@ResponseBody`를 선언할 필요 없다.
    * `@GetMapping`
        - HTTP Method인 Get 요청을 받을 수 있는 API를 만들어준다.
        - `@RequestMapping(method = RequestMethod.GET)`을 사용할 필요 없다.
```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
} 
```
- HelloControllerTest 테스트 코드 생성
    * `@RunWith(SpringRunner.class)`
        - JUnit에 내장된 실행자 외에 다른 실행자(SpringRunner.class)를 실행시킨다.
        - 스프링 부트와 JUnit 사이의 연결자 역할
    * `@WebMvcTest`
        * Web 테스트 특화 어노테이션
        * `@Controller`, `@ControllerAdvice` 등은 사용할 수 있고. `@Service`, `@Component`, `@Repository` 등은 사용할 수 없다.
    * `@Autowired`
        - 스프링이 관리하는 Bean을 주입받는다.
```java
@RunWith(SpringRunner.class)
@WebMvcTest
public class HelloControllerTest {

    @Autowired
    // 웹 API를 테스트 할 때 사용하며 HTTP GET, POST 등에 대한 테스트 가능
    private MockMvc mvc;

    @Test
    public void hello가_리턴된다() throws Exception {
        String hello = "hello";

        mvc.perform(get("/hello")) // MockMvc를 통해 /hello 주소로 HTTP GET 요청
                .andExpect(status().isOk()) // HTTP Header의 Status 검증(에러 상태 코드)
                .andExpect(content().string(hello)); //응답 본문 내용 검증
    }
} 
```
<br>

### 2-2. HelloController 롬복으로 전환하기
- HelloResponseDto 클래스 생성
```java
@Getter
@RequiredArgsConstructor //final로 선언된 필드에 대해 생성자를 생성해준다.
public class HelloResponseDto {

    private final String name;
    private final int amount;

}
```
- HelloResponseDtoTest 클래스로 롬복이 잘 동작하는지 테스트
    * `assertThat`
        - assertj라는 테스트 검증 라이브러리의 검증 메소드로, 검증하고 싶은 대상을 메소드 인자로 받는다.
        - 메소드 체이닝 지원
    * isEqualTo
        - assertj의 동등 비교 메소드로, assertThat 값과 isEqualTo 값을 비교해 같으면 true를 반환한다.
```java
public class HelloResponseDtoTest {

    @Test
    public void 롬복_기능_테스트() {
        //given
        String name = "test";
        int amount = 1000;

        //when
        HelloResponseDto dto = new HelloResponseDto(name, amount);

        //then
        assertThat(dto.getName()).isEqualTo(name);
        assertThat(dto.getAmount()).isEqualTo(amount);
    }
} 
```
- HelloController에 ResponseDto를 사용하도록 코드 추가
    * `@RequestParam`
        - 외부에서 API로 넘긴 파라미터를 가져오는 어노테이션
```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }

    @GetMapping("/hello/dto")
    public HelloResponseDto helloDto(@RequestParam("name") String name, @RequestParam("amount") int amount) {
        return new HelloResponseDto(name, amount);
    }
} 
```
- HelloControllerTest에서 API 테스트
    * param
        - API 테스트 시 사용될 요청 파라미터를 설정한다.
        - 값은 String만 허용되며, 숫자/날짜 등의 데이터도 등록할 때는 문자열로 변경해야 한다.
    * jsonPath
        - JSON 응답값을 필드별로 검증할 수 있는 메소드
        - $를 기준으로 필드명을 명시한다.
```java
@RunWith(SpringRunner.class)
@WebMvcTest
public class HelloControllerTest {

    @Autowired
    private MockMvc mvc;

    @Test
    public void hello가_리턴된다() throws Exception {
        String hello = "hello";

        mvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string(hello));
    }

    @Test
    public void helloDto가_리턴된다() throws Exception {
        String name = "hello";
        int amount = 1000;

        mvc.perform(
                get("/hello/dto")
                        .param("name", name)
                        .param("amount", String.valueOf(amount)))
                .andExpect(status().isOk()) 
                .andExpect(jsonPath("$.name", is(name)))
                .andExpect(jsonPath("$.amount", is(amount)));
    }
} 
```