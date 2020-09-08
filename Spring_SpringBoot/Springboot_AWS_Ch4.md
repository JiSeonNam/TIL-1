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