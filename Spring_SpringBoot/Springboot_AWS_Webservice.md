# 스프링 부트와 AWS로 혼자 구현하는 웹 서비스

## 1. 인텔리제이로 스프링 부트 시작하기

### 1-1. 그레이들 프로젝트 생성 후 스프링 부트 프로젝트로 변경하기
- build.gradle 파일에 다음 코드와 같이 작성한다.
    * ext 키워드는 build.gradle에서 사용하는 전역변수를 설정하겠다는 의미
    * repositoris는 각종 의존성들을 어떠한 원격 저장소에서 받을 지를 지정하는 것
        - 버젼을 명시하지 않아야 `dependencies { classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")`에서 작성한 대로 버젼을 따라간다.
    }
```java
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
```java
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
        - Junit에서 단위 테스트가 끝날 때마다 수행되는 메서드를 지정한다.
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
        // 테이블 posts에 있는 모든 데이터를 조회해오는 메서드
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

### 4-1. 기본 index 페이지 만들기

### 4-2. 게시글 등록 화면 만들기

### 4-3. 전체 조회 화면 만들기

### 4-4. 게시글 수정, 삭제 화면 만들기

## 5. 스프링 시큐리티와 OAuth2.0으로 로그인 기능 구현하기

### 5-1. 구글 로그인 연동하기

### 5-2. 어노테이션 기반으로 개선하기

### 5-3. 네이버 로그인 연동하기

### 5-4. 기존 테스트에 시큐리티 적용하기