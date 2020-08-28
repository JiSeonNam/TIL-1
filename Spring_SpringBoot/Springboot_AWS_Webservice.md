# 스프링 부트와 AWS로 혼자 구현하는 웹 서비스

## 1. 인텔리제이로 스프링 부트 시작하기

### 1-1. 그레이들 프로젝트 생성 후 스프링 부트 프로젝트로 변경하기
- build.gradle 파일에 다음 코드와 같이 작성한다.
    * ext 키워드는 build.gradle에서 사용하는 전역변수를 설정하겠다는 의미
    * repositoris는 각종 의존성들을 어떠한 원격 저장소에서 받을 지를 지정하는 것
        - 버젼을 명시하지 않아야 `dependencies { classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")`에서 작성한 대로 버젼을 따라간다.
    }
```gradle
// 플러그인 의존성 관리 설정
buildscript {
    ext {
        springBootVersion = '2.1.7.RELEASE'
    }
    repositories {
        mavenCentral()
        jcenter()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}

// 앞서 선언한 플러그인 의존성들을 적용할 것인지를 결정하는 코드
// 자바와 스프링 부트를 사용하기 위한 필수 플러그인
apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management' // 스프링 부트의 의존성을 관리해 주는 플러그인으로 반드시 추가해야 한다.

group 'org.example'
version '1.0-SNAPSHOT'

repositories {
    mavenCentral()
}

dependencies {
    compile('org.springframework.boot:spring-boot-starter-web')
    testCompile ('org.springframework.boot:spring-boot-starter-test')
}
```
<br>

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
<br>

## 3. 스프링 부트에서 JPA로 데이터베이스 다루기

### 3-1. 프로젝트에 Spring Data Jpa 적용하기
- build.gradle에 다음과 같이 jpa와 h2 의존성들을 추가한다. 
```gradle
dependencies {
    compile('org.springframework.boot:spring-boot-starter-web')
    compile('org.projectlombok:lombok')
    //  Spring Boot용 Spring Data JPA 추상화 라이브러리로, Spring Boot의 버전에 맞춰 JPA 관련 라이브러리들의 버전을 관리해준다.
    compile('org.springframework.boot:spring-boot-starter-data-jpa')
    // in-memory 관계형 DB로, 별도의 설치없이 프로젝트 의존성만으로 관리할 수 있다.
    compile('com.h2database:h2')
    testCompile ('org.springframework.boot:spring-boot-starter-test')
}
```
<br>

- 실제 DB의 테이블과 매칭될 클래스 Posts 생성
    * 다음 코드와 같이 주요 어노테이션을 클래스에 가깝게 두면 이후에 코틀린 등의 새 언어 전환으로 롬복이 더이상 필요없을 경우 쉽게 삭제할 수 있다.
    * `@NoArgsConstructor`
        - 기본 생성자를 자동으로 추가해주는 어노테이션으로 `public Post() { }`과 같으 효과이다.
    * `@Entity`
        - 테이블과 링크될 클래스임을 나타낸다.
        - 기본값으로 클래스의 카멜케이스 이름을 언더스코어 네이밍(_) 으로 테이블 이름을 매칭한다.
    * `@Id`
        - 해당 테이블 PK 필드를 나타낸다.
    * `@GeneratedValue`
        - PK의 생성 규칙을 나타낸다.
    * `@Column`
        - 테이블의 컬럼을 나타내며 굳이 선언하지 않더라도 해당 클래스의 필드는 모두 컬럼이 된다.
    * `@Builder`
        - 해당 클래스의 빌더 패턴 클래스를 생성한다.
        - 생성자 상단에 선언 시 생성자에 포함된 필드만 빌더에 포함된다.
```java
@Getter
@NoArgsConstructor
@Entity
public class Posts extends BaseTimeEntity {
    //Entity 클래스에서는 절대 Setter 메소드를 만들지 않는다.

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(length = 500, nullable = false) // 사이즈를 500으로
    private String title;

    @Column(columnDefinition = "TEXT", nullable = false) // 타입을 TEXT로 변경
    private String content;

    private String author;

    @Builder
    public Posts(String title, String content, String author) {
        this.title = title;
        this.content = content;
        this.author = author;
    }

    public void update(String title, String content) {
        this.title = title;
        this.content = content;
    }
}
```
- Posts 클래스로 Database에 접근하게 해줄 JpaRepository 생성
```java
public interface PostsRepository  extends JpaRepository<Posts, Long> {

}
```
<br>

### 3-2. Spring Data Jpa 테스트 코드 작성하기
- 테스트 클래스 PostsRepositoryTest를 생성하고 save, findAll 기능 테스트
    * `@After`
        - Junit에서 단위 테스트가 끝날 때마다 수행되는 메소드를 지정한다.
        - 보통 배포 전 전체 테스트를 수행할 때 테스트간 데이터 침범을 막기 위해서 사용한다.
        - 여러 테스트가 동시에 수행되면 테스트용 데이터베이스인 H2에 데이터가 그대로 남아 있어 다음 테스트 실행 시 테스트가 실패할 수 있다.
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class PostRepositoryTest {
 
    @Autowired
    PostsRepository postsRepository;

    @After
    public void cleanup() {
        postsRepository.deleteAll();
    }

    @Test
    public void 게시글저장_불러오기() {
        //given
        String title = "테스트 게시글";
        String content = "테스트 본문";

        //테이블 posts에 insert/update 쿼리를 실행한다. id 값이 있다면 update, id 값이 없다면 insert 쿼리가 실행된다.
        postsRepository.save(Posts.builder()
                .title(title)
                .content(content)
                .author("khy07181@gmail.com")
                .build());
        //when
        // 테이블 posts에 있는 모든 데이터를 조회해오는 메소드
        List<Posts> postsList = postsRepository.findAll();

        //then
        Posts posts = postsList.get(0);
        assertThat(posts.getTitle()).isEqualTo(title);
        assertThat(posts.getContent()).isEqualTo(content);
    }
```
<br>

### 3-3. 등록/수정/조회 API 만들기

- PostsApiController 클래스 생성
```java
@RequiredArgsConstructor
@RestController
public class PostsApiController {

    private final PostsService postsService;

    @PostMapping("/api/v1/posts")
    public Long save(@RequestBody PostsSaveRequestDto requestDto) {
        return postsService.save(requestDto);
    }
    @PutMapping("/api/v1/posts/{id}")
    public Long update(@PathVariable Long id, @RequestBody PostsUpdateRequestDto requestDto) {
        return postsService.update(id, requestDto);
    }

    @GetMapping("/api/v1/posts/{id}")
    public PostsResponseDto findById(@PathVariable Long id) {
        return postsService.findById(id);
    }
}
```
- PostsResponseDto 클래스 생성
```java
@Getter
public class PostsResponseDto {
    private Long id;
    private String title;
    private String content;
    private String author;

    public PostsResponseDto(Posts entity) {
        this.id = entity.getId();
        this.title = entity.getTitle();
        this.content = entity.getContent();
        this.author = entity.getAuthor();
    }
}
```
- PostsupdaterequestDto 클래스 생성
```java
@Getter
@NoArgsConstructor
public class PostsUpdateRequestDto {
    private String title;
    private String content;

    @Builder
    public PostsUpdateRequestDto(String title, String content) {
        this.title = title;
        this.content = content;
    }
} 
```
- PostsService 클래스 생성
    * 절대 Entity 클래스를 Request/Response 클래스로 사용하면 안된다.
        - Entity 클래스는 데이터베이스와 맞닿은 핵심 클래스로, Entity 클래스를 기준으로 테이블이 생성되고 스키마가 변경되기 때문이다.
```java
@RequiredArgsConstructor
@Service
public class PostsService {
    private final PostsRepository postsRepository;

    @Transactional
    public Long save(PostsSaveRequestDto requestDto) {
        return postsRepository.save(requestDto.toEntity()).getId();
    }

    @Transactional
    public Long update(Long id, PostsUpdateRequestDto requestDto) {
        Posts posts = postsRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("해당 사용자가 없습니다. id=" + id));

        posts.update(requestDto.getTitle(), requestDto.getContent());

        return id;
    }

    public PostsResponseDto findById(Long id) {
        Posts entity = postsRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("해당 사용자가 없습니다. id=" + id));

        return new PostsResponseDto(entity);
    }
} 
```
- Controller와 Service에서 사용할 PostsSaveRequestDto 클래스 생성
```java
@Getter
@NoArgsConstructor
public class PostsSaveRequestDto {
    private String title;
    private String content;
    private String author;

    @Builder
    public PostsSaveRequestDto(String title, String content, String author) {
        this.title = title;
        this.content = content;
        this.author = author;
    }

    public Posts toEntity() {
        return Posts.builder()
                .title(title)
                .content(content)
                .author(author)
                .build();
    }
}
```
- 테스트 코드로 검증
```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class PostsApiControllerTest {

    @LocalServerPort
    private int port;

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private PostsRepository postsRepository;

    @After
    public void tearDown() throws Exception {
        postsRepository.deleteAll();
    }
    
    @Test
    public void Posts_동록된다() throws Exception {
        //given
        String title = "title";
        String content = "content";
        PostsSaveRequestDto requestDto = PostsSaveRequestDto.builder()
                .title(title)
                .content(content)
                .author("author")
                .build();
        String url = "http://localhost:" + port + "/api/v1/posts";

        //when
        ResponseEntity<Long> responseEntity = restTemplate.postForEntity(url, requestDto, Long.class);

        //then
        assertThat(responseEntity.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(responseEntity.getBody()).isGreaterThan(0L);

        List<Posts> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(title);
        assertThat(all.get(0).getContent()).isEqualTo(content);
    }
    
    @Test
    public void Posts_수정된다() throws Exception {
        //given
        Posts savedPosts = postsRepository.save(Posts.builder()
                .title("title")
                .content("content")
                .author("author")
                .build());

        Long updateId = savedPosts.getId();
        String expectedTitle = "title2";
        String expectedContent = "content2";

        PostsUpdateRequestDto requestDto = PostsUpdateRequestDto.builder()
                .title(expectedTitle)
                .content(expectedContent)
                .build();

        String url = "http://localhost:" + port + "/api/v1/posts/" + updateId;

        HttpEntity<PostsUpdateRequestDto> requestEntity = new HttpEntity<>(requestDto);

        //when
        ResponseEntity<Long> responseEntity = restTemplate.exchange(url, HttpMethod.PUT, requestEntity, Long.class);

        //then
        assertThat(responseEntity.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(responseEntity.getBody()).isGreaterThan(0L);

        List<Posts> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(expectedTitle);
        assertThat(all.get(0).getContent()).isEqualTo(expectedContent);
    }
}
```
<br>

### 3-4. JPA Auditing으로 생성시간/수정시간 자동화하기
- BaseTimeEntity 클래스 생성
```java
@Getter
@MappedSuperclass // JPA Entity 클래스들이 BaseTimeEntity를 사용할 경우 필드들(createDate, modifiedDate)도 column으로 인식하도록 한다.
@EntityListeners(AuditingEntityListener.class) // BaseTimeEntity 클래스에 Auditing 기능을 포함시킨다.
public abstract class BaseTimeEntity {

    @CreatedDate // Entity가 생성되어 저장될 때 자동 저장된다.
    private LocalDateTime createdDate;

    @LastModifiedDate // 조회한 Entity의 값을 변경할 때 시간이 자동 저장
    private LocalDateTime modifiedDate;

} 
```
- JPA Auditing 어노테이션들을 모두 활성화 할 수 있도록 Application 클래스에 활성화 어노테이션 추가
```java
@EnableJpaAuditing // JPA Auditing 활성화  
@SpringBootApplication  
public class Application {

    public static void main(String[] args) {  
        SpringApplication.run(Application.class, args);  
    }  
}
```
- PostsRepositoryTest 클래스 수정(JPA Auditing 테스트 코드 추가)
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class PostRepositoryTest {

    // ...

    @Test
    public void BaseTimeEntity_등록() {
        //given
        LocalDateTime now = LocalDateTime.of(2019, 6, 4, 0, 0, 0);
        postsRepository.save(Posts.builder()
                .title("title")
                .content("content")
                .author("author")
                .build());
        //when
        List<Posts> postsList = postsRepository.findAll();

        //then
        Posts posts = postsList.get(0);

        System.out.println(">>>>>>>>> createDate=" + posts.getCreatedDate() + ", modifiedDate=" + posts.getModifiedDate());

        assertThat(posts.getCreatedDate()).isAfter(now);
        assertThat(posts.getModifiedDate()).isAfter(now);
    }
}
```
<br>

## 4. 머스테치로 화면 구성하기
- 머스테치 파일 위치는 기본적으로 src/main/resources/templates이다.
- 인텔리제이에서는 머스테치 플러그인을 사용하여 편리하게 사용할 수 있다.
- 스프링 부트에서 머스테치를 편하게 사용할 수 있도록 build.gradle 파일에 머스테치 스타터 의존성을 등록
```gradle
compile('org.springframework.boot:spring-boot-starter-mustache')
```
<br>

### 4-1. 기본 index 페이지 만들기
- index.html 생성
```html
<!DOCTYPE HTML>
<html>
<head>
    <title>스프링부트 웹서비스</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />

</head>
<body>
<h1>스프링 부트로 시작하는 웹 서비스</h1>
</body>
</html> 
```
- IndexController를 생성하여 URL 맵핑
```java
@Controller
public class IndexController {

    @GetMapping("/")
    public String index() {
        return "index"; // 앞의 경로와 뒤의 확장자는 자동적으로 지정된다.
        // src/main/resources/templates/index.mustache
    }
}
```
- IndexControllerTest 클래스를 생성하여 테스트
```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = RANDOM_PORT)
public class IndexControllerTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    public void 메인페이지_로딩() {
        //when
        String body = this.restTemplate.getForObject("/", String.class);

        //then
        assertThat(body).contains("스프링 부트로 시작하는 웹 서비스");
    }
} 
```
<br>

### 4-2. 게시글 등록 화면 만들기
- 부트스트랩과 제이쿼리를 레이아웃 방식으로 추가
    * 레이아웃 방식 : 공통 영역을 별도의 파일로 분리하여 필요한 곳에서 가져다 쓰는 방식
    * 페이지 로딩 속도를 높이기 위해 css는 header에 js는 footer에 두는 것이 좋다.
- header.mustache 파일 생성
```java
<!DOCTYPE HTML>
<html>
<head>
    <title>스프링 부트 웹서비스</title>
    <meta http-equiv="Content-Type" content="text/html";
          charset=UTF-8"/>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css">
</head>
<body> 
```
- footerl.mustache 파일 생성
```java
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"></script>

</body>
</html> 
```
- index.mustache에 글 등록 버튼 추가
    * 공통영역은 header와 footer로 가져다 쓰기
```mustache
{{>layout/header}}

    <h1>스프링 부트로 시작하는 웹 서비스</h1>
    <div class="col-md-12">
        <div class="row">
            <div class="col-md-6">
                <a href="/posts/save" role="button" class="btn btn-primary">글 등록</a>
          </div>
      </div>
    </div>

{{>layout/footer}} 
```
- IndexController에 컨트롤러 생성
```java
@RequiredArgsConstrutor
@Controller
public class IndexController {

    @GetMapping("/")
    public String index() {
        return "index"; 
    }
    // 컨트롤러 추가
    @GetMapping("/posts/save")
    public String postsSave() {
        return "posts-save";
    }
}
```
- posts-save.mustache 파일 생성
```mustache
{{>layout/header}}

<h1>게시글 등록</h1>

<div class="col-md-12">
    <div class="col-md-4">
        <form>
            <div class="form-group">
                <label for="title">제목</label>
                <input type="text" class="form-control" id="title" placeholder="제목을 입력하세요">
            </div>
            <div class="form-group">
                <label for="author">작성자</label>
                <input type="text" class="form-control" id="author" placeholder="작성자를 입력하세요">
            </div>
            <div class="form-group">
                <label for="content">내용</label>
                <textarea class="form-control" id="content" placeholder="내용을 입력하세요"></textarea>
            </div>
        </form>
        <a href="/" role="button" class="btn btn-secondary">취소</a>
        <button type="button" class="btn btn-primary" id="btn-save">등록</button>
    </div>
</div>
{{>layout/footer}} 
```
- index.js 파일 생성하여 등록 버튼 기능 만들기
```js
var index = {
    init : function () {
        var _this = this;
        $('#btn-save').on('click', function () {
            _this.save();
        });
    },
    save : function () {
        var data = {
            title: $('#title').val(),
            author: $('#author').val(),
            content: $('#content').val()
        };

        $.ajax({
            type: 'POST',
            url: '/api/v1/posts',
            dataType: 'json',
            contentType:'application/json; charset=utf-8',
            data: JSON.stringify(data)
        }).done(function() {
            alert('글이 등록되었습니다.');
            location.reload();
        }).fail(function (error) {
            alert(error);
        });
    }

};

index.init(); 
```
<br>

### 4-3. 전체 조회 화면 만들기
- index.mustache 파일에 조회 UI 만들기
    * `{{#posts}}`
        - posts라는 List를 순회한다. Java에서의 for문과 동일
    * `{{변수명}}` ex) `{{id}}`
        - List에서 뽑아낸 객체의 필드를 사용한다.
```mustache
{{>layout/header}}
    <h1>스프링 부트로 시작하는 웹 서비스</h1>
    <div class="col-md-12">
        <div class="row">
            <div class="col-md-6">
                <a href="/posts/save" role="button" class="btn btn-primary">글 등록</a>
            </div>
        </div>
        <!-- 목록 출력 영역-->
        <table class="table table-horizontal table-bordered">
            <thead class="thead-strong">
            <tr>
                <th>게시글번호</th>
                <th>제목</th>
                <th>작성자</th>
                <th>최종수정일</th>
            </tr>
            </thead>
            <tbody id="tbody">
            {{#posts}} //
                <tr>
                    <td>{{id}}</td>
                    <td>{{title}}</td>
                    <td>{{author}}</td>
                    <td>{{modifiedDate}}</td>
                </tr>
            {{/posts}}
            </tbody>
        </table>
    </div>

{{>layout/footer}}
```
- PostsRepository에 쿼리 추가
```java
public interface PostsRepository  extends JpaRepository<Posts, Long> {

    @Query("SELECT p FROM Posts p ORDER BY p.id DESC")
    List<Posts> findAllDesc();
}
```
- PostsService에 코드 추가
```java
@RequiedArgsConstructor
@Service
public class PostsService {
    
    // ...

    @Transactional(readOnly = true)
    public List<PostsListResponseDto> findAllDesc() {
        return postsRepository.findAllDesc().stream()
                .map(PostsListResponseDto::new)
                .collect(Collectors.toList());
        }

    // ...
}
```
- PostsListResponseDto 생성
```java
@Getter
public class PostsListResponseDto {
    private Long id;
    private  String title;
    private  String author;
    private LocalDateTime modifiedDate;

    public PostsListResponseDto(Posts entity) {
        this.id = entity.getId();
        this.title = entity.getTitle();
        this.author = entity.getAuthor();
        this.modifiedDate = entity.getModifiedDate();
    }
}
```
- IndexController 변경
```java
@RequiredArgsConstructor
@Controller
public class IndexController {

    private final PostsService postsService;

    @GetMapping("/")
    public String index(Model model) {
        model.addAttribute("posts", postsService.findAllDesc());
        return "index";
    }

    @GetMapping("/posts/save")
    public String postsSave() {
        return "posts-save";
    }
}
```
<br>

### 4-4. 게시글 수정 화면 만들기
- 게시글 수정 화면 머스테치 파일 posts-update.mustache 파일 생성
    * `{{post.id}}`
        - 머스테치는 객체의 필드 접근 시 Dot(.)로 구분한다.
        - Post 클래스의 id에 대한 접근을 의미한다
    * readonly
        - Input 태그에 읽기 기능만 허용하는 속성이다.
        - id와 author는 수정할 수 없도록 읽기만 허용하도록 추가한다.
```mustache
{{>layout/header}}

<h1>게시글 수정</h1>

<div class="col-md-12">
    <div class="col-me-4">
        <form>
            <div class="form-group">
                <label for="title">글 번호</label>
                <input type="text" class="form-control" id="id" value="{{post.id}}" readonly>
            </div>
            <div class="form-group">
                <label for="title">제목</label>
                <input type="text" class="form-control" id="title" value="{{post.title}}">
            </div>
            <div class="form-group">
                <label for="author">작성자</label>
                <input type="text" class="form-control" id="author" value="{{post.author}}" readonly>
            </div>
            <div class="form-group">
                <label for="content">내용</label>
                <textarea class="form-control" id="content">{{post.content}}</textarea>
            </div>
        </form>
        <a href="/" role="button" class="btn btn-secondary">취소</a>
        <button type="button" class="btn btn-primary" id="btn-update">수정 완료</button>
    </div>
</div>

{{>layout/footer}} 
```
- 수정 기능을 위해 index.js 파일에 update function 추가
    * `$(`btn-update`).on('click')`
        - btn-update란 id를 가진 HTML 엘리먼트에 click 이벤트가 발생할 때 update function을 실행하도록 이벤트 등록한다.
    * `type: "PUT"`
        - 여러 HTTP Method 중 PUT 메소드를 선택
        - REST에서 CRUD는 다음과 같이 HTTP Method에 매핑된다.
            * 생성(Create) - POST
            * 읽기(Read) - GET
            * 수정(Update) - PUT
            * 삭제(Delete) - DELETE
    * `url: '/api/v1/posts/'+id`
        - 어느 게시글을 수정할지 URL path로 구분하기 위해 Path에 id를 추가한다.
```js
var index = {
    
    // ...

    update : function () {
        var data = {
            title: $('#title').val(),
            content: $('#content').val()
        };

        var id = $('#id').val();

        $.ajax({
            type: 'PUT',
            url: '/api/v1/posts/'+id,
            dataType: 'json',
            contentType:'application/json; charset=utf-8',
            data: JSON.stringify(data)
        }).done(function() {
            alert('글이 수정되었습니다.');
            window.location.href = '/';
        }).fail(function (error) {
            alert(JSON.stringify(error));
        });
    }
};
index.init();
```
- 수정 화면을 연결한 Controller 작업을 위해 IndexController에 메소드 추가
```java
@RequiredArgsConstructor
@Controller
public class IndexController{ 
    
    // ...

    @GetMapping("/posts/update/{id}")
    public String postsUpdate(@PathVariable Long id, Model model) {
        PostsResponseDto dto = postsService.findById(id);
        model.addAttribute("post", dto);

        return "posts-update";
    }
}
```
<br>

### 4-5. 게시글 삭제 화면 만들기
- 삭제는 본문을 확인하고 진행해야 하므로 수정화면(posts-update.mustache)에 삭제 버튼 추가
```mustache
{{>layout/header}}

<h1>게시글 수정</h1>

<div class="col-md-12">
    <div class="col-me-4">
        <form>
            <div class="form-group">
                <label for="title">글 번호</label>
                <input type="text" class="form-control" id="id" value="{{post.id}}" readonly>
            </div>
            <div class="form-group">
                <label for="title">제목</label>
                <input type="text" class="form-control" id="title" value="{{post.title}}">
            </div>
            <div class="form-group">
                <label for="author">작성자</label>
                <input type="text" class="form-control" id="author" value="{{post.author}}" readonly>
            </div>
            <div class="form-group">
                <label for="content">내용</label>
                <textarea class="form-control" id="content">{{post.content}}</textarea>
            </div>
        </form>
        <a href="/" role="button" class="btn btn-secondary">취소</a>
        <button type="button" class="btn btn-primary" id="btn-update">수정 완료</button>
        <button type="button" class="btn btn-danger" id="btn-delete">삭제</button> // 삭제버튼
    </div>
</div>

{{>layout/footer}} 
```
- 삭제 기능을 위해 index.js 파일에 delete function 추가 
```js
var index = {

    // ...

    delete : function () {
        var id = $('#id').val();

        $.ajax({
            type: 'DELETE',
            url: '/api/v1/posts/'+id,
            dataType: 'json',
            contentType:'application/json; charset=utf-8'
        }).done(function() {
            alert('글이 삭제되었습니다.');
            window.location.href = '/';
        }).fail(function (error) {
            alert(JSON.stringify(error));
        });
    }
};

index.init();
```
- PostsService 클래스에 delete 메소드 추가
    * `postsRepository.delete(posts)`
        - JpaRepository에서 이미 delete 메소드를 지원해주기 때문에 활용 가능하다.
        - 엔티티를 파라미터로 삭제할 수도 있고, deleteById 메소드를 이용하면 id로 삭제할 수도 있다.
        - 존재하는 Posts인지 확인하기 위해 Entity 조회 후 그대로 삭제한다.
```java
@RequiredArgsConstructor
@Service
public class PostsService {
    private final PostsRepository postsRepository;

    // ...

    @Transactional
    public void delete(Long id) {
        Posts posts = postsRepository.findById(id).orElseThrow(() -> new IllegalArgumentException("해당 사용자가 없습니다. id=" + id));
        postsRepository.delete(posts);
    }

    // ...
}
```
- PostsService에서 만든 delete 메소드를 컨트롤러가 사용하도록 PostsApiController 수정
```java
@RequiredArgsConstructor
@RestController
public class PostsApiController {

    // ...

    @DeleteMapping("/api/v1/posts/{id}")
    public Long delete(@PathVariable Long id) {
        postsService.delete(id);
        return id;
    }

    // ...
}
```
<br>

## 5. 스프링 시큐리티와 OAuth2.0으로 로그인 기능 구현하기

### 5-1. 구글 서비스와 OAuth2.0 사용
- build.gradle에 의존성 추가
```gradle
dependencies {
    // ...
    compile('org.springframework.boot:spring-boot-starter-security')
    compile('org.springframework.boot:spring-boot-starter-oauth2-client')
    // ...
}
```
- src/main/resources에 application-oauth.properties 파일 생성
```properties
spring.security.oauth2.client.registration.google.client-id={clientId}
spring.security.oauth2.client.registration.google.client-secret={clientSecret}
spring.security.oauth2.client.registration.google.scope=profile,email
```
- application.properties에 코드 추가
```properties
spring.profiles.include=oauth
```
<br>

### 5-2. 구글 로그인 연동하기
- User 클래스 생성
    * 사용자 정보를 담당할 도메인
    * `@Enumerated(EnumType.STRING)`
        - JPA로 이용해 데이터베이스로 저장할 때 Enum 값을 어떤 형태로 저장할지를 결정한다.
        - 기본적으로 int로 된 숫자가 저장된다.
        - 숫자로 저장되면 데이터베이스로 확인할 때 그 값이 무슨 코드를 의미하는 지 알 수가 없다.
        - 따라서 문자열(EnumType.STRING)로 저장될 수 있도록 선언한다.
```java
@Getter
@NoArgsConstructor
@Entity
public class User extends BaseTimeEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private String email;

    @Column(nullable = false)
    private String picture;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private Role role;

    @Builder
    public User(String name, String email, String picture, Role role) {
        this.name = name;
        this.email = email;
        this.picture = picture;
        this.role = role;
    }

    public User update(String name, String picture) {
        this.name = name;
        this.picture = picture;

        return this;
    }

    public String getRoleKey() {
        return this.role.getKey();
    }
}
```
- Enum 클래스 Role 생성
    * 각 사용자의 권한을 관리한 Enum 클래스
```java
@Getter
@RequiredArgsConstructor
public enum Role {

    GUEST("ROLE_GUEST", "손님"),
    USER("ROLE_USER", "일반 사용자");

    private final String key;
    private final String title;
}
```
- UserRepository 생성
    * User의 CRUD를 담당한다.
    * `findByEmail()`
        - 소셜 로그인으로 반환되는 값 중 email을 통해 이미 생성된 사용자인지 처음 가입하는 사용자인지 판단하기 위한 메소드
```java
public interface UserRepository extends JpaRepository<User, Long> {

    Optional<User> findByEmail(String email);
}
```
- SecurityConfig 클래스 생성
    * `@EnableWebSecurity`
        - Spring Security 설정들을 활성화시켜 준다.
    * `authorizeRequests()`
        - URL별 권한 관리를 설정하는 옵션의 시작점이다.
        - `authorizeRequests()`가 선언되어야 `antMatchers()` 옵션 사용 가능하다.
    * `antMatchers`
        - 권한 관리 대상을 지정하는 옵션이다.
        - URL, HTTP 메소드별로 관리가 가능하다.
        - "/"등 지정된 URL들은 `permitAll()` 옵션을 통해 전체 열람 권한을 준다.
    * `anyRequest()`
        - 설정된 값들 이외 나머지 URL들을 나타낸다.
        - `authenticated()`를 추가해 나머지 URL들은 모두 인증된 사용자들에게만 허용하도록 한다.
    * `logout().logoutSuccessUrl("/")`
        - 로그아웃 기능에 대한 여러 설정의 진입점이다.
        - 로그아웃 성공 시 "/" 주소로 이동한다.
    * `oauth2Login()`
        - OAuth2 로그인 기능에 대한 여러 설정의 진입점이다.
    * `userInfoEndpoint()`
        - OAuth2 로그인 성공 이후 사용자 정보를 가져올 때의 설정들을 담당한다. 
    * `userService()`
        - 소셜 로그인 성공 시 후속 조치를 진행할 UserService 인터페이스의 구현체를 등록한다.
        - 리소스 서버(즉, 소셜 서비스들)에서 사용자 정보를 가져온 상태에서 추가로 진행하고자 하는 기능을 명시할 수 있다.
```java
@RequiredArgsConstructor
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private  final CustomOAuth2UserService customOAuth2UserService;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        CharacterEncodingFilter filter = new CharacterEncodingFilter();
        http
                .csrf().disable() // h2-console 화면을 사용하기 위해 해당 옵션을 disable
                .headers().frameOptions().disable()
                .and()
                    .authorizeRequests()
                    .antMatchers("/", "/css/**", "/images/**", "/js/**", "/h2-console/**").permitAll()
                    .antMatchers("/api/v1/**").hasRole(Role.USER.name())
                    .anyRequest().authenticated()
                .and()
                    .logout()
                        .logoutSuccessUrl("/")
                .and()
                    .oauth2Login()
                        .userInfoEndpoint()
                            .userService(customOAuth2UserService);
    }
}
```
- CustomOAuth2UserService 클래스 생성
    * 구글 로그인 이후 가져온 사용자의 정보들을 기반으로 가입 및 정보수정, 세션 저장 등의 기능을 지원한다.
    * `registrationId`
        - 현재 로그인 진행 중인 서비스를 구분하는 코드이다.
        - 네이버 로그인 연동 시에 네이버 로그인인지, 구글 로그인인지 구분하기 위해 사용한다.
    * `userNameAttributeName`
        - OAuth2 로그인 진행 시 키가 되는 필드값을 이야기한다.(Primary Key와 같은 의미)
        - 구글의 경우 기본적으로 코드를 지원하지만, 네이버 카카오 등은 기본 지원하지 않는다.
        - 이후 네이버 로그인과 구글 로그인을 동시 지원할 때 사용한다.
    * `OAuthAttributes`
        - OAuth2UserService를 통해 가져온 OAuth2User의 attribute를 담을 클래스
        - 네이버 등 다른 소셜 로그인도 이 클래스를 사용한다.
    * `SessionUser`
        - 세션에 사용자 정보를 저장하기 위한 Dto 클래스
```java
@RequiredArgsConstructor
@Service
public class CustomOAuth2UserService implements OAuth2UserService<OAuth2UserRequest, OAuth2User> {
    private final UserRepository userRepository;
    private final HttpSession httpSession;

    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
        OAuth2UserService delegate = new DefaultOAuth2UserService();
        OAuth2User oAuth2User = delegate.loadUser(userRequest);

        String registrationId = userRequest.getClientRegistration().getRegistrationId();
        String userNameAttributeName = userRequest.getClientRegistration().getProviderDetails()
                .getUserInfoEndpoint().getUserNameAttributeName();

        OAuthAttributes attributes = OAuthAttributes.of(registrationId, userNameAttributeName, oAuth2User.getAttributes());

        User user = saveOrUpdate(attributes);
        httpSession.setAttribute("user", new SessionUser(user));

        return new DefaultOAuth2User(
                Collections.singleton(new SimpleGrantedAuthority(user.getRoleKey())),
                attributes.getAttributes(),
                attributes.getNameAttributeKey());
    }


    private User saveOrUpdate(OAuthAttributes attributes) {
        User user = userRepository.findByEmail(attributes.getEmail())
                .map(entity -> entity.update(attributes.getName(), attributes.getPicture()))
                .orElse(attributes.toEntity());

        return userRepository.save(user);
    }
} 
```
- OAuthAttributes 클래스 생성
    * `of()`
        - OAuth2User에서 반환하는 사용자 정보는 Map이기 때문에 값 하나하나를 변환해야 한다.
    * `toEntity()`
        - User 엔티티를 생성한다.
        - OAuthAttributes에서 엔티티를 생성하는 시점은 처음 가입할 때이다.
```java
@Getter
public class OAuthAttributes {
    private Map<String, Object> attributes;
    private String nameAttributeKey;
    private String name;
    private String email;
    private String picture;

    @Builder
    public OAuthAttributes(Map<String, Object> attributes, String nameAttributeKey, String name, String email, String picture) {
        this.attributes = attributes;
        this.nameAttributeKey = nameAttributeKey;
        this.name = name;
        this.email = email;
        this.picture = picture;
    }

    public static OAuthAttributes of(String registrationId, String userNameAttributeName, Map<String, Object> attributes) {
        return ofGoogle(userNameAttributeName, attributes);
    }

    private static OAuthAttributes ofGoogle(String userNameAttributeName, Map<String, Object> attributes) {
        return OAuthAttributes.builder()
                .name((String) attributes.get("name"))
                .email((String) attributes.get("email"))
                .picture((String) attributes.get("picture"))
                .attributes(attributes)
                .nameAttributeKey(userNameAttributeName)
                .build();
    }

    public User toEntity() {
        return User.builder()
                .name(name)
                .email(email)
                .picture(picture)
                .role(Role.GUEST) // 가입할 때 기본 권한을 GUEST
                .build();
    }
} 
```
- SessionUser 클래스 생성
```java
@Getter
public class SessionUser {
    private String name;
    private String email;
    private String picture;

    public SessionUser(User user) {
        this.name = user.getName();
        this.email = user.getEmail();
        this.picture = user.getPicture();
    }
}
```
<br>

#### 로그인 테스트
- index.mustache에 로그인 버튼 추가
    * `{{#userName}}`
        - mustache는 다른 언어와 같은 if문을 제공하지 않는다. (true / false 여부만 판단한다.)_
        - mustache에 항상 최종 값을 넘겨줘야 한다.
        - userName이 있다면 userName을 노출하도록 구성한다.
    * `a href="/logout"`
        - 스프링 시큐리티에서 기본적으 제공하는 로그아웃 URL이다.
        - 개발자가 별도로 URL에 해당하는 컨트롤러를 만들 필요가 없다.
        - SecurityConfig 클래스에서 URL을 변경할 순 있지만 기본 URL을 사용해도 충분하다.
    * `{{^userName}}`
        - mustache에서 해당 값이 존재하지 않는 경우 ^를 사용
        - userName이 없다면 로그인 버튼을 노출하도록 구성
    * `a href="/oauth2/authorization/google"`
        - 스프링 시큐리티에서 기본적으로 제공하는 로그인 URL
        - 로그아웃 URL과 마찬가지로 개발자가 별도의 컨트롤러를 생성할 필요가 없다.
```mustache
{{>layout/header}}

    <h1>스프링 부트로 시작하는 웹 서비스</h1>
    <div class="col-md-12">
        <div class="row">
            <div class="col-md-6">
                <a href="/posts/save" role="button" class="btn btn-primary">글 등록</a>
                {{#userName}}
                    Logged in as: <span id="user">{{userName}}</span>
                    <a href="/logout" class="btn btn-info active" role="button">Logout</a>
                {{/userName}}
                {{^userName}}
                    <a href="/oauth2/authorization/google" class="btn btn-success active" role="button">Google Login</a>
                {{/userName}}
            </div>
        </div>

        <!-- 목록 출력 영역-->
        // ...
```
- IndexController에 userName을 model에 저장하는 코드 추가
```java
@RequiredArgsConstructor
@Controller
public class IndexController {

    private final PostsService postsService;
    private final HttpSession httpSession;

    @GetMapping("/")
    public String index(Model model) {
        model.addAttribute("posts", postsService.findAllDesc());
        // 로그인 성공 시 값을 가져온다.
        SessionUser user = (SessionUser) httpSession.getAttribute("user");

        if(user != null) { // 세션에 저장된 값이 있을 경우
            // 세션에 저장된 값이 있으면 model엔 아무런 값이 없는 상태이므로 로그인 버튼이 보이게 된다.
            model.addAttribute("userName", user.getName());
        }
        return "index";
    }
}
```
<br>

### 5-3. 어노테이션 기반으로 개선하기
- IndexCotroller에서 세션값을 가져오는 부분 애노테이션 기반으로 개선하기
- @LoginUser 애노테이션 생성
    * `@Target(ElementType.PARAMETER)`
        - 어노테이션이 생성될 수 있는 위치를 지정한다.
        - PARAMETER로 지정했으므로 메서소의 파라미터로 선언된 객체에서만 사용될 수 있다.
        - 이 외에도 클래스 선언시 사용할 수 있는 TYPE 등이 있다.
```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface LoginUser {

}
```
- LoginUserArgumentResolver 생성
    * `supportParameter()`
        - 컨트롤러 메소드의 특정 파라미터를 지원하는지 판단한다.
        - 파라미터에 @LoginUser 어노테이션이 붙어있고, 파라미터 클래스 타입이 SessionUser.class인 경우 true를 반환
    * `resolveArgument()`
        - 파라미터에 전달할 객체를 생성한다.
        - 아래 코드에서는 세션에서 객체를 가져온다.
```java
@RequiredArgsConstructor
@Component
public class LoginUserArgumentResolver implements HandlerMethodArgumentResolver {

    private final HttpSession httpSession;

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        boolean isLoginUserAnnotation = parameter.getParameterAnnotation(LoginUser.class) != null;
        boolean isUserClass = SessionUser.class.equals(parameter.getParameterType());

        return isLoginUserAnnotation && isUserClass;
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        return httpSession.getAttribute("user");
    }
}
```
- WebConfig 클래스 생성
    * LoginUserArgumentResolver를 스프링에서 인식할 수 있게 해준다.
```java
@RequiredArgsConstructor
@Configuration
public class WebConfig implements WebMvcConfigurer {
    private final LoginUserArgumentResolver loginUserArgumentResolver;

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
        argumentResolvers.add(loginUserArgumentResolver);
    }
}
```
- IndexController의 반복되는 부분 `@LoginUser` 애노테이션으로 개선
    * 이제 어느 컨트롤러든지 `@LoginUser` 애노테이션만 사용하면 세션 정보를 가져올 수 있다.
```java
@RequiredArgsConstructor
@Controller
public class IndexController {

    private final PostsService postsService;
    private final HttpSession httpSession;

    @GetMapping("/")

    public String index(Model model, @LoginUser SessionUser user) {
        model.addAttribute("posts", postsService.findAllDesc());

        //SessionUser user = (SessionUser) httpSession.getAttribute("user");
        if(user != null) {
            model.addAttribute("userName", user.getName());
        }
        return "index";
    }

    // ...
}
```
<br>

### 5-4. 네이버 로그인 연동하기
- application-oauth.properties에 키 값들을 등록
    * 네이버에서는 스프링 시큐리티를 공식 지원하지 않기 때문에 수동으로 입력해야 한다.
```properties
#registration

spring.security.oauth2.client.registration.naver.client-id=clientID값
spring.security.oauth2.client.registration.naver.client-secret=clientSecret값
spring.security.oauth2.client.registration.naver.redirect-uri={baseUrl}/{action}/oauth2/code/{registrationId}
spring.security.oauth2.client.registration.naver.authorization-grant-type=authorization_code
spring.security.oauth2.client.registration.naver.scope=name,email,profile_image
spring.security.oauth2.client.registration.naver.client-name=Naver

#provider
spring.security.oauth2.client.provider.naver.authorization-uri=https://nid.naver.com/oauth2.0/authorize
spring.security.oauth2.client.provider.naver.token-uri=https://nid.naver.com/oauth2.0/token
spring.security.oauth2.client.provider.naver.user-info-uri=https://openapi.naver.com/v1/nid/me
spring.security.oauth2.client.provider.naver.user-name-attribute=response
```
- OAuthAttributes에 네이버를 판단하는 코드와 네이버 생성자 코드 추가
```java
@Getter
public class OAuthAttributes {
    // ...

    public static OAuthAttributes of(String registrationId, String userNameAttributeName, Map<String, Object> attributes) {
        if("naver".equals(registrationId)) {
            return ofNaver("id", attributes);
        }

        return ofGoogle(userNameAttributeName, attributes);
    }

    // ...

    private static OAuthAttributes ofNaver(String userNameAttributeName, Map<String, Object> attributes) {
        Map<String, Object> response = (Map<String, Object>) attributes.get("response");

        return OAuthAttributes.builder()
                .name((String) response.get("name"))
                .email((String) response.get("email"))
                .picture((String) response.get("profile_image"))
                .attributes(response)
                .nameAttributeKey(userNameAttributeName)
                .build();
    }
}
```
- index.mustache에 네이버 로그인 버튼 추가
```mustache
{{>layout/header}}
    <h1>스프링 부트로 시작하는 웹 서비스</h1>
    <div class="col-md-12">
        <div class="row">
            <div class="col-md-6">
                <a href="/posts/save" role="button" class="btn btn-primary">글 등록</a>
                {{#userName}}
                    Logged in as: <span id="user">{{userName}}</span>
                    <a href="/logout" class="btn btn-info active" role="button">Logout</a>
                {{/userName}}
                {{^userName}}
                    <a href="/oauth2/authorization/google" class="btn btn-success active" role="button">Google Login</a>
                    <a href="/oauth2/authorization/naver" class="btn btn-secondary active" role="button">Naver Login</a> // 네이버 로그인 버튼 추가
                {{/userName}}
            </div>
        </div>
        <!-- 목록 출력 영역-->

        // ...
```
<br>

### 5-5. 기존 테스트에 시큐리티 적용하기
- 지금 상태에서 전체 테스트를 수행하면 롬복을 이용한 테스트 외에 스프링을 이용한 테스트는 모두 실패한다.

#### 5-5-1. CustomOAuth2UserService를 찾을 수 없음
- CustomOAuth2UserService를 생성하는데 필요한 소셜 로그인 관련 설정값들이 없기 때문에 발생(src/main과 src/test 환경의 차이)
- test 환경을 위한 application.properties 파일을 만들고 가짜 설정값을 등록한다.
```properties
spring.jpa.show_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.h2.console.enabled=true
spring.session.store-type=jdbc

# Test OAuth

spring.security.oauth2.client.registration.google.client-id=test
spring.security.oauth2.client.registration.google.client-secret=test
spring.security.oauth2.client.registration.google.scope=profile,email
```
<br>

#### 5-5-2. 302 Status Code
- 스프링 시큐리티 설정 때문에 인증되지 않은 사용자의 요청은 이동시켜 302 에러 코드가 뜬다.
- 임의로 인증된 사용자를 추가하여 API만 테스트하게 하면 해결 가능하다.
- build.gradle에 spring-security-test 추가
 ```gradle
 compile('org.springframework.security:spring-security-test')
 ```
 - PostsApiControllerTest에 임의의 사용자 인증 추가
 ```java
 @RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class PostsApiControllerTest {
    // ...

    @Autowired
    private WebApplicationContext context;

    private MockMvc mvc;

    @Before
    public void setup() {
        mvc = MockMvcBuilders
                .webAppContextSetup(context)
                .apply(springSecurity())
                .build();
    }

    // ...

    @Test
    @WithMockUser(roles="USER")
    public void Posts_동록된다() throws Exception {
        // ...

        //when
        mvc.perform(post(url)
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .content(new ObjectMapper().writeValueAsString(requestDto)))
                .andExpect(status().isOk());
    }

    // ...

    @Test
    @WithMockUser(roles="USER")
    public void Posts_수정된다() throws Exception {
        // ...

        //when
        mvc.perform(put(url)
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .content(new ObjectMapper().writeValueAsString(requestDto)))
                .andExpect(status().isOk());
    }
    
    // ...00
}
 ```
 <br>

#### 5-5-3. @WebMvcTest에서 CustomOAuth2UserService를 찾을 수 없음
- HelloControllerTest에서 `@WebMvcTest`를 사용했기 때문에 5-5-1로 인해 스프링 시큐리티 설정은 작동하지만 CustomOAuth2UserService를 스캔하지 않는다.
    * `@WebMvcTest`는 web과 관련된 것만 스캔한다.
- 따라서 다음과 같이 스캔 대상에서 SecurityConfig를 제거한다.
- 또한 마찬가지로 @WithMockUser를 사용해 가짜로 인증된 사용자를 생성한다.
```java
@RunWith(SpringRunner.class)
@WebMvcTest
@WebMvcTest(controllers = HelloController.class,
        excludeFilters = {
                @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = SecurityConfig.class)
        }
)
public class HelloControllerTest {
    // ...

    @WithMockUser(roles="USER")
    @Test
    public void hello가_리턴된다() throws Exception {
        
        //...
    }

    // ...

    @WithMockUser(roles="USER")
    @Test
    public void helloDto가_리턴된다() throws Exception {
        
        // ...
    }
}
```
<br>

#### 5-5-4. @EnableJpaAuditing 관련 에러
- 에러 코드 : `java.lang.IllegalArgumentException: At least one JPA metamodel must be present!`
- `@EnableJpaAuditing`를 사용하기 위해서는 최소 하나의 @Entity 클래스가 필요하지만 @WebMvcTest이기 떄문에 없다.
- `@EnableJpaAuditing`와 `@SpringBootApplication`이 함께 있기 때문에 `@WebMvcTest`에서도 스캔하므로 이 둘을 분리 시켜야 한다.
- Application.java에서 `@EnableJpaAuditing`을 제거한다.
```java
// @EnableJpaAuditing
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
- JpaConfig를 생성하여 `@EnableJpaAuditing`를 추가한다.
```java
@Configuration
@EnableJpaAuditing
public class JpaConfig {

}
```
<br>
