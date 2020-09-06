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

#### 5-2-1. 로그인 테스트
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
    
    // ..
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

## 6. AWS 서버 환경을 만들어보자 - AWS EC2
- 리전 서울로 변경
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_1.jpg"></p>
<br>

### 6-1. EC2 인스턴스 생성하기
- 인스턴스란 EC2 서비스에 생성된 가상머신이다.
- 가장 먼저 ec2를 검색해 인스턴스 시작을 누른 후 AMI를 선택한다.
    * AMI(Amazon Machine Image)는 EC2 인스턴스를 시작하는 데 필요한 정보를 이미지로 만들어 둔 것이다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_2.jpg"></p>

<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_3.jpg"></p>

- 인스턴스 유형에서는 프리티어로 표기된 t2.micro를 선택한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_4.jpg"></p>

- 세부정보 구성
    * 별다른 설정을 하지 않고 넘어간다.
- 스토리지 추가
    * 서버의 용량을 얼마나 정할지 선택하는 단계
    * 기본값은 8GB지만 프리티어로 30GB까지 가능하다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_5.jpg"></p>

- 태그 추가
    * 웹 콘솔에서 표기될 태그인 Name 태그를 등록한다. 
    * 태그는 해당 인스턴스를 표현하는 여러 이름으로 사용될 수 있다.(EC2의 이름을 정한다고 생각하면 쉽다)
    * 서비스의 인스턴스를 나타낼 수 있는 값으로 등록한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_6.jpg"></p>

- 보안 그룹
    * 방화벽을 이야기한다.
    * 기존에 생성된 보안 그룹이 없으므로 유의미한 이름으로 변경한다.
    * pem 키 관리를 잘해야하고 지정된 IP에서만 ssh 접속이 가능하도록 구성하는 것이 안전하다.
    * 현재 접속한 장소의 IP를 등록하고 카페 또는 다른 장소에서 접속할 때는 해당 장소의 IP를 다시 SSH규칙에 추가하는 것이 안전하다.
    * 현재 프로젝트의 기본 포트인 8080도 추가하고 검토 및 시작 버튼을 누른다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_7.jpg"></p>

- 인스턴스 시작 검토
    * 보안 그룹 경고를 하는데 이는 8080이 전체 오픈 되어서 발생한다.
    * 8080을 열어 놓는 것은 위험한 일이 아니므로 시작하기 버튼 클릭
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_8.jpg"></p>

- 할당할 pem 키 선택
    * 인스턴스로 접근하기 위해서는 pem 키(비밀키)가 필요하다.
    * 인스턴스는 지정된 pem 키와 매칭되는 공개키를 가지고 있어, 해당 pem 키 외에는 접근을 허용하지 않는다.
    * 일종의 마스터 키로 절대 유출되서는 안되며 EC2 서버로 접속할 때 필수 파일이기 떄문에 디렉토리로 저장한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_9.jpg"></p>

- pem키를 다운받으면 인스턴스 id를 눌러 EC2 목록으로 이동
- 인스턴스 생성이 완료되면 다음과 같이 IP와 도메인이 할당된다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_10.jpg"></p>

- 고정 IP 할당
    * AWS의 고정 IP를 EIP(Elastic IP, 탄력적IP)라고 한다.
    * 카테고리에서 탄력적 IP를 선택하고 새 주소 할당을 한다.
    * IPv4 주소 풀은 Amazon 풀
- 생성된 탄력적 IP와 EC2 주소를 연결
    * 탄력적 Ip를 생성하고 EC2 서버에 연결하지 않으면 비용이 발생한다.(반드시 연결해야한다)
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_11.jpg"></p>

- EC2 이름과 IP를 선택하고 연결한 뒤 인스턴스 목록 페이지로 이동하면 다음과 같이 연결된 모습을 볼 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_12.jpg"></p>
<br>

### 6-2. EC2 서버에 접속하기(window 기준)
- putty 설치
    * putty.exe
    * puttygen.exe
- puttygen.exe 파일을 실행
    * putty는 pem키로 사용이 안되고 pem키를 ppk파일로 변환해야 한다.
    * puttygen이 그 과정을 진행해주는 클라이언트이다.
- puttygen 화면에서 다운로드 받은 pem키를 선택한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_13.jpg"></p>

- 자동으로 변환이 진행되며 Save private key 버튼을 눌러 ppk파일을 생성한다.
    * 경고 창이 뜨면 예를 누르고 넘어간다.
- ppk 파일이 저장될 위치와 ppk 이름을 등록
- ppk 파일이 생성되었으면 putty.exe를 실행하여 다음과 같이 각 항목을 등록한다.
    * HostName : username@public_Ip를 등록한다.
        - ec2-user@탄력적IP주소
    * Port : ssh 접속 포트인 22를 등록한다.
    * Connection type : SSH 선택
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_14.jpg"></p> 

- 왼쪽 사이드바에 Auth를 클릭해서 ppk 파일을 로드할 수 있는 화면으로 이동
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_15.jpg"></p> 

- 방금 생성한 ppk파일을 불러오고 Session 탭으로 이동해서 Saved Sessions에 현재 설정들을 저장할 이름을 등록하고 Save버튼을 클릭한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_16.jpg"></p>

- 저장한 뒤 Load 버튼을 클릭하면 SSH 접속 알림이 뜨고 예를 클릭하면 다음과 같이 SSH 접속이 성공한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_17.jpg"></p>
<br>

### 6-3. 아마존 리눅스 1 서버 생성 시 꼭 해야 할 설정들
- 프로젝트에 맞는 자바 설치
    * (블로그에 정리 해놓음)
    * [Putty에 jdk 11 설치하기](https://qlalzl9.tistory.com/344)
- 타임존 변경
    * EC2 서버의 기본 타임존은 UTC(세계 표준 시간)으로 한국의 시간과 9시간 차이난다.
    * 서버에서 수행되는 Java 애플리케이션에서 생성되는 시간도 모두 9시간 차이가 나기 때문에 반드시 수정해야 한다.

```
sudo rm /etc/localtime
sudo ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime
```
- Hostname 변경
    * 여러 서버를 관리 중일 경우 IP만으로 어떤 서비스의 서버인지 확인이 어렵기 때문에 변경한다.
    * 명령어를 실행 후 HOSTNAME 부분을 원하는 서비스명으로 변경하면 된다.
```
sudo vim /etc/sysconfig/network
```
- 호스트 주소를 찾을 때 가장 먼저 검색해 보는 /etc/hosts에 변경한 hostname을 등록
    * `127.0.0.1` 부분에 등록한 HOSTNAME을 등록한다.
    * 등록 후 `curl 등록한 호스트 이름`으로 확인 가능하다.
    * 잘 등록헀다면 80포트로 접근이 안된다는 에러가 발생한다.
        - 80포트로 실행된 서비스가 없음을 의미하고 curl 호스트 이름으로 실행은 잘 되었음을 의미한다.
```
sudo vim /etc/hosts
```
<br>

## 7. AWS 서버 환경을 만들어보자 - AWS RDS
- RDS
    * AWS에서 지원하는 클라우드 기반 관계형 데이터베이스
    * 하드웨어 프로비저닝, 데이터베이스 설정, 패치 및 백업과 같이 잦은 운영 작업을 자동화하여 개발자가 개발에 집중할 수 있게 지원하는 서비스이다.
    * 모니터링, 알람, 백업, HA구성 등 또한 지원한다.

### 7-1. RDS 인스턴스 생성하기
- RDS를 검색하고 RDS 대시보드에서 데이터베이스 생성 버튼을 클릭한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_18.jpg"></p>

- DB 엔진 옵션과 템플릿 선택
    * 각각 MariaDB와 프리 티어 선택한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_19.jpg"></p>
 
- 상세 설정에서 스토리지를 20으로 입력하고 DB인스턴스와 마스터 사용자 정보를 등록한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_20.jpg"></p>

- 네트워크 및 보안
    * 퍼블릭 엑세스를 예로 변경한다.
    * 이후 보안 그룹에서 지정된 IP만 접근하도록 막을 예정
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_21.jpg"></p>

- 데이터베이스 옵션 
    * 데이터베이스 이름을 작성하고 포트는 3306으로 입력
    * 파라미터 그룹의 변경을 이후에 따로 진행하므로 기본으로 놔둔다.
<br>

### 7-2. RDS 운영환경에 맞는 파라미터 설정하기
- RDS를 처음 생성하면 필수로 해야하는 설정들이 있다.
    * 타임존
    * Character Set
    * Max Connection
- 카테고리에서 파라미터 그룹을 선택한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_22.jpg"></p>

- 파라미터 그룹 생성을 클릭하고 DB엔진을 선택하는 항목에서 전에 생성한 MaridDB와 같은 버전을 맞춰야 한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_23.jpg"></p>

- 생성이 완료되면 해당 파라미터 그룹을 클릭하고 파라미터를 편집한다.
    * 각 항목들을 검색해서 편집한다.
    * time_zone : Asia/Seoul
    * character-set-client : utf8mb4
    * character-set-connection : utf8mb4
    * character-set-database : utf8mb4
    * character-set-filesystem : utf8mb4
    * character-set-results : utf8mb4
    * collation_connection : utf8mb4
    * collation_server : utf8mb4
    * max_connections : 150

- 완성된 파라미터 그룹을 데이터베이스에 연결한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_24.jpg"></p>

- 데이터베이스 옵션 항목에서 DB 파라미터 그룹을 default에서 방금 생성한 신규 파라미터 그룹으로 변경한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_25.jpg"></p>

- 저장을 누르고 수정사항 요약을 즉시 적용으로 한다.
    * 예약된 다음 유지 관리 기간에 적용하면 지금 하지 않고, 새벽 시간대에 진행된다.
    * 수정사항이 반영되는 동안 데이터베이스가 작동하지 않을 수 있으므로 예약 시간을 걸어두는 의미지만 서비스가 오픈되지 않았기 때문에 즉시 적용한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_26.jpg"></p>

- 간혹 파라미터 그룹이 제대로 반영되지 않을 때가 있어 정상 적용을 위해 재부팅을 진행한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_27.jpg"></p>
<br>

### 7-3. 내 PC에서 RDS 접속하기
- 로컬 PC에서 RDS로 접근하기 위해 RDS의 보안 그룹에 본인 PC의 IP를 추가한다.
    * RDS 세부정보 페이지 - 보안 그룹
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_28.jpg"></p>

- 브라우저를 새로 열어 보안 그룹 목록 중 EC2에 사용된 보안 그룹의그룹 ID를 복사한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_29.jpg"></p>

- 복사된 보안 그룹 ID와 본인의 IP를 RDS보안 그룹의 인바운드로 추가한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_30.jpg"></p>
<br>

#### 7-3-1. 로컬에서 테스트하기
- 실습 기준 IntelliJ Community 버전에서 Database Navigator 플러그인 설치
- Database Navigator를 실행하고 RDS 접속 정보 등록을 한다.
    * Host에는 RDS의 엔드 포인트를 등록한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_31.jpg"></p>

- Test Connection을 통해 연결 테스트를 하고 SQL을 실행할 콘솔창을 연다.
- 쿼리가 수행될 database를 선택
    * 만약 database명을 잊었다면 Schema 항목에 추가되어 있다.
```sql
use AWS RDS 웹 콘솔에서 지정한 데이터베이스명;
```
- 현재의 charater_set, collation 설정을 확인한다.
```sql
show variables like 'c%';
```
- 만약 utfmb4가 적용이 안된 필드가 있다면 다음과 같이 직접 변경한다.
```sql
ALTER DATABASE 데이터베이스명
CHARACTER SET = 'utf8mb4'
COLLATE = 'utf8mb4_general_ci';
```
- 타임존 확인
```sql
select @@time_zone, now();
```
- 테이블 생성 및 insert 쿼리 실행
```sql
CREATE TABLE test (
   id bigint(20) NOT NULL AUTO_INCREMENT,
   content varchar(255) DEFAULT NULL,
   PRIMARY KEY (id)
) ENGINE=InnoDB;
```
```sql
insert into test(content) values ('테스트');
```
```sql
select * from test;
```
<br>

### 7-4. EC2에서 RDS 접근 확인
- putty에서 EC2에 ssh 접속을 진행한다. 
- MySQL설치
```
sudo yum install mysql
```
- MySQL에 계정, 비밀번호, 호스트 주소를 사용해 RDS에 접속한다.
```
mysql -u 계정 -p -h Host주소
```
- RDS에 접속되었으면 실제로 생성한 RDS가 맞는지 확인
```sql
show database
```
<br>

## 8. EC2 서버에 프로젝트 배포하기

### 8-1. EC2에 프로젝트 clone 받기
- EC2에 깃 설치
```
sudo yum install git
```
- 프로젝트 clone할 디렉토리 생성
```
mkdir ~/app && mkdir ~/app/step1
```
- 디렉토리로 이동한 후 clone 진행
```
git clone 프로젝트주소
```
- 코드가 정상 수행되는지 테스트
    * 만약 gradlew 실행 권한이 없다는 메세지(`-bash: ./gradlew: Permission denied`)가 뜨면 실행 권한을 추가(`chmod +x ./gradlew`)한 뒤 다시 테스트를 수행한다.
```
./gradlew test
```
<br>

### 8-2. 배포 스크립트 만들기
- ~/app/step1/에 deploy.sh 파일 생성
```
vim ~/app/git/deploy.sh
```
- deploy.sh 파일 작성
```vim
#!/bin/bash

REPOSITORY=/home/ec2-user/app/step1
PROJECT_NAME=springboot-webservice

cd $REPOSITORY/$PROJECT_NAME/

echo "> Git Pull"

git pull

echo "> 프로젝트 Build 시작"

./gradlew build

echo "> step1 디렉토리 이동"

cd $REPOSITORY

echo "> Build 파일 복사"

cp $REPOSITORY/$PROJECT_NAME/build/libs/*.jar $REPOSITORY/

echo "> 현재 구동중인 애플리케이션 pid 확인"

CURRENT_PID=$(pgrep -f ${PROJECT_NAME}*.jar)

echo "현재 구동 중인 애플리케이션 pid: $CURRENT_PID"

if [ -z "$CURRENT_PID" ]; then
   echo "> 현재 구동 중인 애플리케이션이 없으므로 종료하지 않습니다."
else
   echo "> kill -15 $CURRENT_PID"
   kill -15 $CURRENT_PID
   sleep 5
fi

echo "> 새 애플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/ | grep *.jar | tail -n 1)

echo "> JAR Name: $JAR_NAME"

nohup java -jar $REPOSITORY/$JAR_NAME 2>&1 &
```
- 스크립트에 실행 권한 추가
```
chmod +x ./deploy.sh
```
- 스프립트 실행
    * nohup.out 파일을 열어보면 로그를 볼 수 있다.
    * `vim nohup.out`
```
./deploy.sh
```
- nohup.out에는 ClientRegistrationRepository를 찾을 수 없다는 에러가 난다. 
    * 8-3에서 고친다.
<br>

### 8-3. 외부 Security 파일 등록하기
- 에러가 난 이유는 로컬에는 application-oauth.properties가 있지만 깃에는 올라가 있지 않아 서버가 가지고 있지 않기 때문에 서버에서 직접 이 설정들을 가지고 있게 해야한다.
- app 디렉토리에 properties 파일 생성
```
vim /home/ec2-user/app/application-oauth.properties
```
- 로컬에 있는 application-oauth.properties파일 내용을 그대로 붙여넣는다.
- application-oauth.properties를 사용하도록 deploy.sh파일을 수정한다.
```vim
...
nohup java -jar \
        -Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties \
        $REPOSITORY/$JAR_NAME 2>&1 &
```
- deploy.sh를 실행해서 정상적으로 실행되는지 확인
<br>

### 8-4. 스프링 부트 프로젝트로 RDS 접근하기

#### 8-4-1. RDS 테이블 생성
- JPA가 사용될 엔티티 테이블 생성
```sql
create table posts (
    id bigint not null auto_increment,
    create_date datetime, 
    modified_date datetime, 
    author varchar(255), 
    content TEXT not null, 
    title varchar(500) not null, 
    primary key (id)
    ) engine=InnoDB;
```
```sql
create table user (
    id bigint not null auto_increment,
    create_date datetime,
    modified_date datetime,
    email varchar(255) not null, 
    name varchar(255) not null, 
    picture varchar(255), 
    role varchar(255) not null, 
    primary key (id)
    ) engine=InnoDB;
```
- 스프링 세션이 사용될 테이블 생성
```sql
CREATE TABLE SPRING_SESSION (
	PRIMARY_ID CHAR(36) NOT NULL,
	SESSION_ID CHAR(36) NOT NULL,
	CREATION_TIME BIGINT NOT NULL,
	LAST_ACCESS_TIME BIGINT NOT NULL,
	MAX_INACTIVE_INTERVAL INT NOT NULL,
	EXPIRY_TIME BIGINT NOT NULL,
	PRINCIPAL_NAME VARCHAR(100),
	CONSTRAINT SPRING_SESSION_PK PRIMARY KEY (PRIMARY_ID)
) ENGINE=InnoDB ROW_FORMAT=DYNAMIC;
```
```sql
CREATE UNIQUE INDEX SPRING_SESSION_IX1 ON SPRING_SESSION (SESSION_ID);
CREATE INDEX SPRING_SESSION_IX2 ON SPRING_SESSION (EXPIRY_TIME);
CREATE INDEX SPRING_SESSION_IX3 ON SPRING_SESSION (PRINCIPAL_NAME);
```
```sql
CREATE TABLE SPRING_SESSION_ATTRIBUTES (
	SESSION_PRIMARY_ID CHAR(36) NOT NULL,
	ATTRIBUTE_NAME VARCHAR(200) NOT NULL,
	ATTRIBUTE_BYTES BLOB NOT NULL,
	CONSTRAINT SPRING_SESSION_ATTRIBUTES_PK PRIMARY KEY (SESSION_PRIMARY_ID, ATTRIBUTE_NAME),
	CONSTRAINT SPRING_SESSION_ATTRIBUTES_FK FOREIGN KEY (SESSION_PRIMARY_ID) REFERENCES SPRING_SESSION(PRIMARY_ID) ON DELETE CASCADE
) ENGINE=InnoDB ROW_FORMAT=DYNAMIC;
```
<br>

#### 8-4-2. 프로젝트 설정
- MariaDB 드라이버를 build.gradle에 등록한다.
```gradle
compile("org.mariadb.jdbc:mariadb-java-client")
```
- 서버에서 구동될 환경 구성
    * scr/main/resources/에 application-real.properties 파일 생성
    * profile=real인 환경이 구성된다고 생각하면 된다.
    * 실제 운영될 환경이기 때문에 보안/로그상 이슈가 될 만한 설정들을 모두 제거하며 RDS환경 profile 설정이 추가된다.
```properties
spring.profiles.include=oauth,real-db
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.session.store-type=jdbc 
```
- 모든 설정이 되었다면 깃허브로 푸시한다.
<br>

#### 8-4-3. EC2 설정
- OAuth와 마찬가지로 RDS 접속 정보도 보호해야할 정보이므로 EC2 서버에 직접 설정 파일을 둔다.
- app디렉토리에 application-real-db.properties 파일 생성
```
vim ~/app/application-real-db.properties
```
- application-real-db.properties 파일 작성
```properties
spring.jpa.hibernate.ddl-auto=none
spring.datasource.url=jdbc:mariadb://rds주소:포트명(기본은 3306)/database명
spring.datasource.username=db계정
spring.datasource.password=db계정 비밀번호
spring.datasource.driver-class-name=org.mariadb.jdbc.Driver
```
- deploy.sh가 real profile을 쓸 수 있도록 수정
```vim
...
nohup java -jar \
   -Dspring.config.location=classpath:/application.properties,classpath:/application-real.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
   -Dspring.profiles.active=real \
   $JAR_NAME > $REPOSITORY/nohup.out 2>&1 &
```
- deploy.sh 파일을 실행하고 nohup.out에 성공적으로 로그가 나오는지 확인한다.
- curl 명령어로 html 코드가 정상적으로 보인다면 성공한 것이다.
```
curl localhost:8080
```
<br>

### 8-5. EC2에서 소셜 로그인하기
- EC2에 스프링 부트 프로젝트가 8080포트로 배포되었으므로 8080포트가 보안 그룹에 열려있는지 확인한다.
    * 만약 열려 있지 않다면 편집 버튼을 눌러 추가해 준다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_32.jpg"></p>

- 인스턴스 메뉴를 클릭하면 다음과 같이 퍼블릭 DNS를 확인할 수 있다.
    * 이 주소가 EC2에 자동으로 할당된 도메인이다.
    * 어디에서나 이 주소를 입력하면 EC2서버에 접근할 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_33.jpg"></p>

<br>

#### 8-5-1. 구글 로그인 서비스 등록
- 구글 웹 콘솔로 접속하여 본인의 프로젝트로 이동한 다음 API 및 서비스 -> 사용자 인증 정보로 이동한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_34.jpg"></p>

- OAuth 동의 화면 탭을 선택하고 승인된 도메인에 http:// 없이 EC2의 퍼블릭 DNS를 등록한다. 
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_35.jpg"></p>

- 사용자 인증 정보 탭을 클릭하여 본인의 서비스 이름을 선택하고 퍼블릭 DNS 주소에 :8080/login/oauth2/code/google주소를 추가하여 승인된 리디렉션 URI에 등록한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_36.jpg"></p>
<br>

#### 8-5-2. 네이버 로그인 서비스 등록
- 네이버 개발자 센터로 접속해서 본인의 프로젝트로 이동한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_37.jpg"></p>

- PC 웹 항목의 서비스 URL과 Callback URL 2개를 수정한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_38.jpg"></p>

- 완료하면 구글 로그인과 네이버 로그인이 정상적으로 연동 완료된다.
<br>

## 9. 코드가 푸시되면 자동으로 배포하기(Travis CI 배포 자동화)

### 9-1. Travis CI 연동하기

#### TRavis CI 웹 서비스 설정
- [Travis](https://travis-ci.org/)에서 깃허브 계정으로 로그인 한뒤 Setting에서 저장소 이름을 검색한 뒤 상태바를 활성화 시킨다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_39.jpg"></p>
<br>

#### 프로젝트 설정
- travis CI의 상세한 설정은 프로젝트에 존재하는 .travis.yml파일로 할 수 있다.
- 프로젝트의 build.gradle과 같은 위치에 .travis.yml파일을 생성하고 다음과 같은 코드를 추가한다.
```yml
language: java
jdk:
  - openjdk11

# Travis Ci를 어느 브랜치가 푸시될 때 수행할지 지정
branches: 
  only:
    - master

# Travis CI 서버의 Home
# 그레이들을 통해 의존성을 받게 되면 이를 해당 디렉토리에 캐시하여, 같은 의존성은 다음 배포 때부터 다시 받지 않도록 설정
cache:
  directories:
    - '$HOME/.m2/repository'
    - '$HOME/.gradle'

# master 브랜치에 푸시되었을 때 수행하는 명령어로 여기서는 프로젝트 내부에 둔 gradlew을 통해 clean & build를 수행
script: "./gradlew clean build"

# CI 실행 완료 시 메일로 알람
notifications:
  email:
    recipients:
      - khy07181@gmail.com 
```
- master 브랜치에 커밋과 푸시하고 Travis CI 저장소 페이지를 확인하고 이메일을 확인하면 빌드가 성공했다고 알려준다.
<br>

### 9-2. Travis CI와 AWS S3 연동하기
- S3는 AWS에서 제공하는 일종의 파일 서버이다.
- 실제 배포는 AWS의 CodeDeploy라는 서비스를 이용한다.
- Jar파일을 전달하기 위해 S3연동을 먼저하며, CodeDeploy는 저장 기능이 없기 때문에 Travis CI가 빌드한 결과물을 받아서 CodeDeploy가 가져갈 수 있도록 보관할 수 있는 공간이 필요해서 AWS S3를 사용한다.
<br>

#### AWS Key 발급
- 일반적으로 AWS 서비스에 외부 서비스가 접근할 수 없기 때문에 접근 가능한 권한을 가진 Key를 생성해서 사용해야 한다.
    * AWS에서는 이러한 인증과 관련된 기능을 제공하는 서비스로 IAM이 있다.
- IAM은 AWS에서 제공하는 서비스의 접근 방식과 권한을 관리한다. 
- IAM을 통해 Travis CI가 AWS의 S3와 CodeDeply에 접근할 수 있도록 한다.
- AWS에서 IAM을 검색하고 사용자 추가
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_40.jpg"></p>

- 생성할 사용자의 이름과 엑세스 유형을 선택한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_41.jpg"></p>

- 권한 설정 방식은 기존 정책 직접 연결을 선택하고 정책 검색 화면에서 s3full과 CodeDeployFull을 검색하여 각각 체크한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_42.jpg"></p>

- 태그 추가에서는 Name값을 본인이 인지 가능한 정도의 이름으로 만든다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_43.jpg"></p>

- 권한 최종 확인
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_44.jpg"></p>

- 최종 생성이 완료되면 엑세스 키와 비밀 엑세스 키가 생성된다.
    * 이 두 값이 Travis CI에서 사용될 Key이다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_45.jpg"></p>
<br>

#### Travis CI에 Key 등록
- Travis 설정 화면에서 Environment Variables 항목에 AWS_ACESS_KEY, AWS_SECRET_KEY를 변수로 IAM 사용자에서 발급받은 키 값들을 등록한다.
    * AWS_ACESS_KEY : 엑세스 키 ID
    * AWS_SECRET_KEY : 비밀 엑세스 키
    * 등록하면 이제 .travis.yml에서 `$AWS_ACESS_KEY`, `$AWS_SECRET_KEY`로 사용할 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_46.jpg"></p>
<br>

#### S3 버킷 생성
 - AWS에서 S3를 검색 후 버킷 만들기를 선택한다.
 - 원하는 버킷명을 작성한다.
    * 이 버킷에 배포할 Zip 파일이 모여있ㄴ튼 장소임을 의미하도록 짓는 것이 좋다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_47.jpg"></p>

- 버전관리 설정은 별다른 설정 할 것 없이 넘어가고 버킷의 보안과 권한 설정 부분에서 모든 차단을 해야 한다.
    * 실제 서비스를 햘 경우 Jar 파일이 public일 경우 누구나 내려받을 수 있어 코드나 설정값, 주요 키값들이 발취될 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_48.jpg"></p>

- 버킷이 생성되면 다음과 같이 버킷 목록에서 확인할 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_49.jpg"></p>

#### .travis.yml 수정
- Travis CI에서 빌드하여 만든 Jar 파일을 S3에 올릴 수 있도록 다음 코드를 .travis.yml에 추가한다.
```yml
# ...

# deploy 명령어가 실행되기 전에 수행되며 CodeDeploy는 Jar 파일을 인식하지 못하므로 Jar+기타 설정 파일들을 모아 압축한다.
before_deploy:
  - zip -r aws-springboot2-webservice *
  - mkdir -p deploy
  - mv aws-springboot2-webservice.zip deploy/aws-springboot2-webservice.zip

# S3로 파일 업로드 혹은 CodeDeploy로 배포 등 외부 서비스와 연동될 행위들을 선언한다.
deploy:
  - provider: s3
    access_key_id: $AWS_ACCESS_KEY # Travis repo settings에 설정된 값
    secret_access_key: $AWS_SECRET_KEY # Travis repo settings에 설정된  값
    bucket: aws-springboot2-build # S3 버킷
    region: ap-northeast-2
    skip_cleanup: true
    acl: private # zip 파일 접근을 private으로
    local_dir: deploy # before_deploy에서 생성한 디렉토리
    wait-until-deploy: true

# ...
```
- 설정이 완료되면 깃허브로 푸시한다. Travis CI에서 자동으로 진행되는 것과 모든 빌드과 성공하는지 확인한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_50.jpg"></p>

- S3 버킷에도 업로드가 성공한 것을 확인할 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_51.jpg"></p>
<br>

### 9-4. Travis CI와 AWS S3, CodeDeploy 연동하기
- AWS의 배포 시스템인 CodeDeploy를 이용하기 전에 배포 대상인 EC2가 CodeDeploy를 연동 받을 수 있게 IAM 역할을 하나 생성한다.
<br>

#### EC2에 IAM 역할 추가하기
- IAM을 검색하고 역할 -> 역할만들기를 차례로 클릭한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_52.jpg"></p>

- 서비스 선택에서 AWS 서비스, EC2를 선택한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_53.jpg"></p>

- 정책에선 EC2RoleForA를 검색하여 AmazonEc2RoleforAWS-CodeDeploy를 선택한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_54.jpg"></p>

- 태그는 원하는 이름으로 만든다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_55.jpg"></p>

- 마지막으로 역할의 이름을 등록하고 나머지 등록 정보를 최종적으로 확인한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_56.jpg"></p>

- 만든 역할을 EC2 서비스에 등록하기
    * EC2 인스턴스 목록이르 이동하고 인스턴스를 다음과 같이 선택한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_57.jpg"></p>

- 방금 생성한 역할을 선택한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_58.jpg"></p>

- 역할 선택이 완료되면 해당 EC2 인스턴스를 재부팅한다.
    * 재부팅해야만 역할이 정상적으로 적용된다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_59.jpg"></p>
<br>

#### CodeDeploy 에이전트 설치
- EC2 서버에 접속해 다음 명령어를 입력한다.
```
aws s3 cp s3://aws-codedeploy-ap-northeast-2/latest/install . --region ap-northeast-2
```
- install 파일에 실행권한을 추가한다.
```
chmod +x ./install
```
- insatll 파일로 설치 진행하기
```
sudo ./install auto
```
- 설치가 끝나면 Agent가 정상적으로 실행되고 있는지 상태 검사를 한다.
    * `The AWS CodeDeploy angent is running as PID XXX`라고 메세지가 출력되면 정상이다.
```
sudo service codedeploy-agent status
```
<br>

#### CodeDeploy를 위한 권한 생성
- CodeDeploy에서 EC2의 접근도 권한이 필요하므로 IAM 역할을 생성한다.
- 서비스는 AWS 서비스 - CodeDeploy를 차례로 선택한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_60.jpg"></p>

- CodeDeploy는 권한이 하나뿐이라서 선택 없이 다음으로 넘어간다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_61.jpg"></p>

- 태그도 원하는 이름으로 짓고 생성완료한다.
<br>

#### CodeDeploy 생성
- CodeDeploy 서비스로 이동해 애플리케이션 생성을 한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_62.jpg"></p>

- 생성할 CodeDeploy 이름과 컴퓨팅 플랫폼을 선택한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_63.jpg"></p>

- 생성이 완료되면 배포 그룹 생성을 한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_64.jpg"></p>

- 배포 그룹 이름과 서비스 역할 등록한다. 
    * 서비스 역할은 CodeDeploy용 IAM 역할을 선택하면 된다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_65.jpg"></p>

- 배포 유형에서는 현재 위치를 선택한다.
    * 배포할 서비스가 2대 이상이라면 블루/그린 선택
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_66.jpg"></p>

- 환경 구성에서는 Amazon EC2 인스턴스에 체크한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_67.jpg"></p>

- 마지막으로 배포 설정과 로드 밸런서를 선택한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_68.jpg"></p>
<br>

#### Travic CI, S3, CodeDeploy 연동
- EC2 서버에 S3에서 넘겨줄 zip 파일을 저장할 디렉토리 생성
```
mkdir ~/app/step2 && mkdir ~/app/step2/zip
```
- 프로젝트에 AWS CodeDeploy 설정을 위해 appspec.yml 작성
```yml
version: 0.0
os: linux
files:
  - source: /
    destination: /home/ec2-user/app/step2/zip/
    overwrite: yes 

```
- Travis CI 설정을 위해 .travis.yml에 코드 추가
```yml
# ...

  - provider: codedeploy
    access_key_id: $AWS_ACCESS_KEY # Travis repo settings에 설정된 값
    secret_access_key: $AWS_SECRET_KEY # Travis repo settings에 설정된  값
    bucket: aws-springboot2-build # S3 버킷
    key: aws-springboot2-webservice.zip # 빌드 파일을 압축해서 전달
    bundle_type: zip
    application: aws-springboot2-webservice # 웹 콘솔에서 등록한 CodeDeploy 애플리케이션
    deployment_group: aws-springboot2-webservice-group # 웹 콘솔에서 등록한 CodeDeploy 배포 그룹
    region: ap-northeast-2
    wait-until-deployed: true

# ...
```
- 프로젝트를 커밋하고 푸시한다.
- Travis Ci가 끝나면 CodeDeploy 화면에서 배포가 수행되는 것을 확인할 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_69.jpg"></p>

- 배포가 끝나면 EC2 서버에서 파일들이 잘 도착했는지 확인
```
cd /home/ec2-user/app/step2/zip
```
- `ll`명령어를 사용해 파일들을 확인한다.
<br>

### 9-5 배포 자동화 구성
- Travis CI, S3, CodeDeploy 연동까지 구현 후 실제로 Jar를 배포하여 실행
<br>

- deploy.sh 파일 추가
    * 프로젝트의 scripts 디렉토리를 생성하고 여기에 스크립트를 생성한다.
```sh
#!/bin/bash

REPOSITORY=/home/ec2-user/app/step2
PROJECT_NAME=aws-springboot2-webservice

echo "> Build 파일 복사"

cp $REPOSITORY/zip/*.jar $REPOSITORY/

echo "> 현재 구동 중인 애플리케이션 pid 확인"

CURRENT_PID=$(pgrep -fl aws-springboot2-webservice | grep jar | awk '{print $1}')

echo "현재 구동 중인 애플리케이션 pid: $CURRENT_PID"

if [ -z "$CURRENT_PID" ]; then
    echo "> 현재 구동 중인 애플리케이션이 없으므로 종료하지 않습니다."
else
    echo "> kill -15 $CURRENT_PID"
    kill -15 $CURRENT_PID
    sleep 5
fi

echo "> 새 애플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/*.jar | tail -n 1)

echo "> JAR Name: $JAR_NAME"

echo "> $JAR_NAME 에 실행권한 추가"

chmod +x $JAR_NAME

echo "> $JAR_NAME 실행"

nohup java -jar \
    -Dspring.config.location=classpath:/application.properties,classpath:/application-real.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
    -Dspring.profiles.active=real \
    $JAR_NAME > $REPOSITORY/nohup.out 2>&1 & 
```
- .travis.yml 파일 수정
```yml
# ...

before_deploy:
  - mkdir -p before-deploy # zip에 포함시킬 파일들을 담을 디렉토리 생성
  - cp scripts/*.sh before-deploy/
  - cp appspec.yml before-deploy/
  - cp build/libs/*.jar before-deploy/
  - cd before-deploy && zip -r before-deploy * # before-deploy로 이동 후 전체 압축
  - cd ../ && mkdir -p deploy # 상위 디렉토리로 이동 후 deploy 디렉토리 생성
  - mv before-deploy/before-deploy.zip deploy/aws-springboot2-webservice.zip # deploy로 zip파일 이동

# ...
```
- appspec.yml 파일 수정
```yml
# ...

permissions:
  - object: /
    pattern: "**"
    owner: ec2-user
    group: ec2-user

hooks:
  ApplicationStart:
    - location: deploy.sh
      timeout: 60
      runas: ec2-user 
```
- 모든 설정이 완료되면 깃허브로 커밋과 푸시해서 Travis와 CodeDeploy의 성공 메세지를 확인한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_70.jpg"></p>
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_71.jpg"></p>
<br>

- 웹 브라우저에서 EC2 도메인을 입력해서 확인하기
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_72.jpg"></p>
<br>

#### 실제 배포 과정 체험
- build.gradle의 프로젝 버전을 변경한다.
```gradle
version '1.0.1-SNAPSHOT'
```
- 변경 내용을 알 수 있게 index.mustache에 Ver.2 테스트를 추가한다.
```html
<h1>스프링 부트로 시작하는 웹 서비스 Ver.2</h1>
```
- 깃허브로 커밋과 푸시하면 다음과 같이 변경된 코드가 배포된 것을 확인할 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_73.jpg"></p>
<br>

### 9-6 CodeDeploy 로그 확인
- CodeDeploy와 같이 AWS가 지원하는 서비스에서 오류가 발생하면 오류를 해결하기 힘들다. 
- 따라서 배포가 실패하면 로그를 보고 오류를 해결하는 방법이 좋다.
- CodeDeploy의 대부분 내용은 /opt/codedeploy-agent/deployment-root에 있다.
- 이 디렉토리로 이동한 뒤 목록을 확인하면 로그를 확인할 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_74.jpg"></p>

<br>
