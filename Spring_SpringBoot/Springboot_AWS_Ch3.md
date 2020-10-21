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
        - 참고) [Builder Pattern](https://johngrib.github.io/wiki/builder-pattern/)
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