# 검색 및 첫 페이지

## 검색 기능 구현
- 키워드를 입력 받아서 스터디를 검색하며 로그인 없이도 사용 가능하다.
- 간단하게 페이징과 정렬 조건 없이 키워드에 해당하는 모든 스터디를 조회
- 공개된 스터디만 조회 가능해야 하고 키워드가 스터디 이름, 태그 이름, 도시 이름에 해당하는 스터디만 조회
- 보여줄 내용
    * 검색 키워드와 결과 개수, 없으면 없다고 표기
    * 스터디 당 보여줄 정보
        - 스터디 이름
        - 짧은 소개
        - 태그
        - 지역
        - 멤버 수
        - 스터디 공개 일시
<br>

### 구현
- MainController에 검색 맵핑 추가
```java
@Controller
@RequiredArgsConstructor
public class MainController {

    private final StudyRepository studyRepository;

    ...

    @GetMapping("/search/study")
    public String searchStudy(String keyword, Model model) {
        List<Study> studyList = studyRepository.findByKeyword(keyword);
        model.addAttribute(studyList);
        model.addAttribute("keyword", keyword);
        return "search";
    }
}
```
- StudyRepositoryExtension 인터페이스 생성
    * findByKeyword를 구현하기 위해 QueryDSL을 사용한다.
    * Predicate를 쓰는 것이 아닌 다른 방법으로 확장 구현체를 만드는 방법이다.
```java
@Transactional(readOnly = true)
public interface StudyRepositoryExtension {

    List<Study> findByKeyword(String keyword);

}
```
- StudyRepository가 StudyRepositoryExtension을 상속받도록 한다.
    * 이렇게만 해서는 동작하지 않는다. 구현해야 한다.
```java
@Transactional(readOnly = true)
public interface StudyRepository extends JpaRepository<Study, Long>, StudyRepositoryExtension {

    ...
}
```
- StudyRepositoryExtension을 구현하는 구현체 StudyRepositoryExtensionImpl 생성
    * QuerydslRepositorySupport
        - querydsl로 구현할 때 유용하게 쓸 수 있는 클래스
```java
public class StudyRepositoryExtensionImpl extends QuerydslRepositorySupport implements StudyRepositoryExtension {

    // 부모의 기본 생성자가 없기 때문에 만들어준다.
    public StudyRepositoryExtensionImpl() {
        super(Study.class);
    }

    @Override
    public List<Study> findByKeyword(String keyword) {
        // Predicate를 쓰는 것이 아니라 from으로 시작할 수 있다.
        QStudy study = QStudy.study;
        JPQLQuery<Study> query = from(study).where(study.published.isTrue() // study의 공개가 된 것들 중에서 
                .and(study.title.containsIgnoreCase(keyword)) // 스터디 제목이 keyword를 가지고 있거나
                .or(study.tags.any().title.containsIgnoreCase(keyword)) // 태그 중에 keyword가 있거나
                .or(study.zones.any().localNameOfCity.containsIgnoreCase(keyword))); // 지역 중 keyword가 있으면
        return query.fetch();
    }
}
```
- 검색 뷰 생성
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments.html :: head"></head>
<body class="bg-light">
    <div th:replace="fragments.html :: main-nav"></div>
    <div class="container">
        <div class="py-5 text-center">
            <p class="lead" th:if="${#lists.isEmpty(studyList)}">
                <strong th:text="${keyword}" id="keyword" class="context"></strong>에 해당하는 스터디가 없습니다.
            </p>
            <p class="lead" th:if="${!#lists.isEmpty(studyList)}">
                <strong th:text="${keyword}" id="keyword" class="context"></strong>에 해당하는 스터디를
                <span th:text="${studyList.size()}"></span>개
                찾았습니다.
            </p>
        </div>
        <div class="row justify-content-center">
            <div class="col-sm-10">
                <div class="row">
                    <div class="col-md-4" th:each="study: ${studyList}">
                        <div class="card mb-4 shadow-sm">
                            <img th:src="${study.image}" class="context card-img-top" th:alt="${study.title}" >
                            <div class="card-body">
                                <a th:href="@{'/study/' + ${study.path}}" class="text-decoration-none">
                                    <h5 class="card-title context" th:text="${study.title}"></h5>
                                </a>
                                <p class="card-text" th:text="${study.shortDescription}">Short description</p>
                                <p class="card-text context">
                                        <span th:each="tag: ${study.tags}" class="font-weight-light text-monospace badge badge-pill badge-info mr-3">
                                            <a th:href="@{'/search/tag/' + ${tag.title}}" class="text-decoration-none text-white">
                                                <i class="fa fa-tag"></i> <span th:text="${tag.title}">Tag</span>
                                            </a>
                                        </span>
                                    <span th:each="zone: ${study.zones}" class="font-weight-light text-monospace badge badge-primary mr-3">
                                            <a th:href="@{'/search/zone/' + ${zone.id}}" class="text-decoration-none text-white">
                                                <i class="fa fa-globe"></i> <span th:text="${zone.localNameOfCity}" class="text-white">City</span>
                                            </a>
                                        </span>
                                </p>
                                <div class="d-flex justify-content-between align-items-center">
                                    <small class="text-muted">
                                        <i class="fa fa-user-circle"></i>
                                        <span th:text="${study.members.size()}"></span>명
                                    </small>
                                    <small class="text-muted date" th:text="${study.publishedDateTime}">9 mins</small>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <div th:replace="fragments.html :: footer"></div>
    <script th:replace="fragments.html :: date-time"></script>
</body>
</html>
```
- 로그인 하지 않은 사용자도 이용할 수 있도록 SecurityConfig설정
```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    ...

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .mvcMatchers("/", "/login", "/sign-up", "check-email-token",
                        "/email-login", "/login-by-email", "/search/study").permitAll()
                ...
    }

    ...
}
```
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/search_firstPage_1.jpg"></p>

<br>

## N+1 Select 문제 해결
- 현재 검색 쿼리는 N + 1 문제가 발생한다.
    * QueryDSL 쿼리 1개
    * 알림 조회
    * 스터디 당
        - 태그 조회
        - 지역 정보 조회
        - 멤버 조회
- 지금까지는 EntityGraph를 사용해 해결했지만  QueryDSL을 썼기 때문에 fetching할 내용도 쿼리에서 설정해야 한다.
    * left (outer) join + fetchJoin + distinct로 해결한다.
<br>

### 구현
- 스터디를 기준으로 연관돼있는 데이터를 가져오는 left join 설정
```java
public class StudyRepositoryExtensionImpl extends QuerydslRepositorySupport implements StudyRepositoryExtension {

    ...

    @Override
    public List<Study> findByKeyword(String keyword) {
        QStudy study = QStudy.study;
        JPQLQuery<Study> query = from(study).where(study.published.isTrue()
                .and(study.title.containsIgnoreCase(keyword))
                .or(study.tags.any().title.containsIgnoreCase(keyword))
                .or(study.zones.any().localNameOfCity.containsIgnoreCase(keyword)))
                .leftJoin(study.tags, QTag.tag).fetchJoin() // study의 tags는 QTag에 join
                .leftJoin(study.zones, QZone.zone).fetchJoin() // study의 zones는 Qzone에 join
                .leftJoin(study.members, QAccount.account).fetchJoin() // study의 members는 QAccount에 join
                .distinct(); // 중복 데이터 제거
        return query.fetch();
    }
}
```
- 실행해서 쿼리를 확인하면 QueryDSL 실행 쿼리 1개, 알림 조회 쿼리 1개 총 2개만 나온다.

<br>

## 페이징 적용
- 페이징을 위해 임의의 데이터(스터디 30개)를 추가해서 사용한다.
- 고전적인 방식의 페이징
    * SQL의 limit과 offset 사용하기
- 스프링 데이터 JPA가 제공하는 Pageable 사용하기
    * page와 size
    * sort도 지원한다.
    * 기본값을 설정하는 방법 `@PageableDefault`
<br>

### 테스트용 데이터 생성
- 기능 확인을 위해 데이터를 넣어주는 기능 구현
    * 잠시 넣었다가 데이터를 삭제하면 된다.
```java
@Controller
@RequiredArgsConstructor
public class StudyController {

    ...

    @GetMapping("/study/data")
    public String generateTestData(@CurrentAccount Account account) {
        studyService.generateTestStudies(account);
        return "redirect:/";
    }
}
```
```java
@Service
@Transactional
@RequiredArgsConstructor
public class StudyService {

    ...
    private final TagRepository tagRepository;

    ...

    public void generateTestStudies(Account account) {
        // 스터디 30개 생성
        for(int i = 0; i < 30; i++) {
            String randomValue = RandomString.make(5);
            Study study = Study.builder()
                    .title("테스트 스터디 " + randomValue)
                    .path("test-" + randomValue)
                    .shortDescription("테스트용 스터디입니다.")
                    .fullDescription("test")
                    .tags(new HashSet<>())
                    .zones(new HashSet<>())
                    .managers(new HashSet<>())
                    .build();
            study.publish(); // 스터디 공개 설정
            Study newStudy = this.createNewStudy(study, account);
            // 30개의 스터디 모두 JPA라는 태그를 가지고 있도록 설정
            Tag jpa = tagRepository.findByTitle("JPA");
            newStudy.getTags().add(jpa);

        }
    }
}
```
- 실행해서 데이터를 추가하고 작성했던 코드를 되돌린다.
    * 실행하면 다음과 같이 DB에 랜덤값이 달린 스터디 30개가 생성된다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/search_firstPage_2.jpg"></p>

<br>

### 페이징 구현
- Pageable 사용
    * Pageable로 받을 수 있는 파라미터
        - size
        - page
        - sort
    * `@PageableDefault`
        - 페이징 기본값 설정
```java
@Controller
@RequiredArgsConstructor
public class MainController {

    ...

    @GetMapping("/search/study")
    public String searchStudy(String keyword, Model model,
                              @PageableDefault(size = 9, sort = "publishedDateTime", direction = Sort.Direction.DESC)
                                      Pageable pageable) {
        Page<Study> studyPage = studyRepository.findByKeyword(keyword, pageable);
        model.addAttribute("studyPage", studyPage);
        model.addAttribute("keyword", keyword);
        return "search";
    }
}
```
- StudyRepositoryExtension, StudyRepositoryExtensionImpl에 Pageable 설정
    * IDE에서 QuickFix 이용하면 편하다.
```java
@Transactional(readOnly = true)
public interface StudyRepositoryExtension {

    Page<Study> findByKeyword(String keyword, Pageable pageable);

}
```
```java
public class StudyRepositoryExtensionImpl extends QuerydslRepositorySupport implements StudyRepositoryExtension {

    ...

    @Override
    public Page<Study> findByKeyword(String keyword, Pageable pageable) {
        ...
    }
}
```
- 
```java
public class StudyRepositoryExtensionImpl extends QuerydslRepositorySupport implements StudyRepositoryExtension {

    ...

    @Override
    public Page<Study> findByKeyword(String keyword, Pageable pageable) {
        QStudy study = QStudy.study;
        JPQLQuery<Study> query = from(study).where(study.published.isTrue()
                .and(study.title.containsIgnoreCase(keyword))
                .or(study.tags.any().title.containsIgnoreCase(keyword))
                .or(study.zones.any().localNameOfCity.containsIgnoreCase(keyword)))
                .leftJoin(study.tags, QTag.tag).fetchJoin()
                .leftJoin(study.zones, QZone.zone).fetchJoin()
                .leftJoin(study.members, QAccount.account).fetchJoin()
                .distinct();
        // 페이징 적용
        JPQLQuery<Study> pageableQuery = getQuerydsl().applyPagination(pageable, query); // pagination 적용
        QueryResults<Study> fetchResults = pageableQuery.fetchResults(); // fetch가 아닌 fetchResults를 사용해야 한다.(fetch는 데이터만 가져온다.)
        return new PageImpl<>(fetchResults.getResults(), pageable, fetchResults.getTotal());
    }
}
```
- 화면에서도 List를 참조하는 대신 Page 타입을 사용하기 때문에 Page가 제공하는 기능을 사용한다.
```html
...

<p class="lead" th:if="${studyPage.getTotalElements() == 0}">
                <strong th:text="${keyword}" id="keyword" class="context"></strong>에 해당하는 스터디가 없습니다.
            </p>
            <p class="lead" th:if="${studyPage.getTotalElements() > 0}">
                <strong th:text="${keyword}" id="keyword" class="context"></strong>에 해당하는 스터디를
                <span th:text="${studyPage.getTotalElements()}"></span>개
                찾았습니다.
            </p>
        </div>
        <div class="row justify-content-center">
            <div class="col-sm-10">
                <div class="row">
                    <div class="col-md-4" th:each="study: ${studyPage.getContent()}">
                    ...
```
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/search_firstPage_3.jpg"></p>

<br>

## 검색 뷰 개선
- 부트스트랩을 사용하여 페이지 네비게이션 추가
- 정렬 조건 추가
    * 스터디 공개 일시
    * 멤버 수
- 키워드 하이라이팅
    * [mark.js](https://markjs.io/) 사용
<br>

### 구현
- 부트스트랩의 Pagination라는 뷰를 사용해서 페이지 네비게이션 추가
```html
<!-- search.html -->
<div class="row justify-content-center">
    <div class="col-sm-10">
        <nav>
            <ul class="pagination justify-content-center">
                <li class="page-item" th:classappend="${!studyPage.hasPrevious()}? disabled">
                    <a th:href="@{'/search/study?keyword=' + ${keyword} + '&sort=' + ${sortProperty} + ',desc&page=' + ${studyPage.getNumber() - 1}}"
                       class="page-link" tabindex="-1" aria-disabled="true">
                        Previous
                    </a>
                </li>
                <li class="page-item" th:classappend="${i == studyPage.getNumber()}? active"
                    th:each="i: ${#numbers.sequence(0, studyPage.getTotalPages() - 1)}">
                    <a th:href="@{'/search/study?keyword=' + ${keyword} + '&sort=' + ${sortProperty} + ',desc&page=' + ${i}}"
                       class="page-link" href="#" th:text="${i + 1}">1</a>
                </li>
                <li class="page-item" th:classappend="${!studyPage.hasNext()}? disabled">
                    <a th:href="@{'/search/study?keyword=' + ${keyword} + '&sort=' + ${sortProperty} + ',desc&page=' + ${studyPage.getNumber() + 1}}"
                       class="page-link">
                        Next
                    </a>
                </li>
            </ul>
        </nav>
    </div>
</div>
```
- 부트스트랩의 Dropdowns을 사용해 정렬 조건 추가
```html
<div class="dropdown">
    <button class="btn btn-light dropdown-toggle" type="button" id="dropdownMenuButton" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
        검색 결과 정렬 방식
    </button>
    <div class="dropdown-menu" aria-labelledby="dropdownMenuButton">
        <a class="dropdown-item" th:classappend="${#strings.equals(sortProperty, 'publishedDateTime')}? active"
           th:href="@{'/search/study?sort=publishedDateTime,desc&keyword=' + ${keyword}}">
            스터디 공개일
        </a>
        <a class="dropdown-item" th:classappend="${#strings.equals(sortProperty, 'memberCount')}? active"
           th:href="@{'/search/study?sort=memberCount,desc&keyword=' + ${keyword}}">
            멤버 수
        </a>
    </div>
</div>
```
- 서버 쪽에서는 Model에 sortProperty를 넘겨줘야 한다.
```java
@Controller
@RequiredArgsConstructor
public class MainController {

    ...

    @GetMapping("/search/study")
    public String searchStudy(String keyword, Model model,
                              @PageableDefault(size = 9, sort = "publishedDateTime", direction = Sort.Direction.DESC)
                                      Pageable pageable) {
        Page<Study> studyPage = studyRepository.findByKeyword(keyword, pageable);
        model.addAttribute("studyPage", studyPage);
        model.addAttribute("keyword", keyword);
        model.addAttribute("sortProperty",
                pageable.getSort().toString().contains("publishedDateTime") ? "publishedDateTime" : "memberCount");
        return "search";
    }
}
```
- 스터디의 멤버 수를 자주 쿼리하는데 쿼리 개수를 줄이기 위해 Study엔티티에 memberCount추가
```java
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
@Builder @AllArgsConstructor @NoArgsConstructor
public class Study {

    ...

    private int memberCount;

    ...

    public void addMember(Account account) {
        this.members.add(account);
        this.memberCount++;

    }

    public void removeMember(Account account) {
        this.getMembers().remove(account);
        this.memberCount--;
    }

    ...
}
```
- 키워드 하이라이팅(mark.js 사용)
    * `npm install mark.js --save`
```html
<img th:src="${study.image}" class="context card-img-top" th:alt="${study.title}" >

...

<script src="/node_modules/mark.js/dist/jquery.mark.min.js"></script>
    <script type="application/javascript">
        $(function(){
            var mark = function() {
                // Read the keyword
                var keyword = $("#keyword").text();

                // Determine selected options
                var options = {
                    "each": function(element) {
                        setTimeout(function() {
                            $(element).addClass("animate");
                        }, 150);
                    }
                };

                // Mark the keyword inside the context
                $(".context").unmark({
                    done: function() {
                        $(".context").mark(keyword, options);
                    }
                });
            };

            mark();
        });
    </script>
```
```html
<style>
    ...
    
    mark {
        padding: 0;
        background: transparent;
        background: linear-gradient(to right, #f0ad4e 50%, transparent 50%);
        background-position: right bottom;
        background-size: 200% 100%;
        transition: all .5s ease;
        color: #fff;
    }
    mark.animate {
        background-position: left bottom;
        color: #000;
    }
</style>
```
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/search_firstPage_4.jpg"></p>

<br>

## 첫페이지 - 로그인 전
- 최근 9개의 스터디를 조회해서 화면에 띄운다.
    * 공개했고, 아직 종료하지 않은 스터디 중에서 9개
- 페이징 없고 List<Study>로 조회
- 쿼리 만들지 말고, 스프링 데이터 JPA 쿼리 메소드로 만들기
- 뷰 코드는 최대한 재사용
<br>

### 구현
- MainController에서 studyList를 넘겨준다.
```java
@Controller
@RequiredArgsConstructor
public class MainController {

    private final StudyRepository studyRepository;

    @GetMapping("/")
    public String home(@CurrentAccount Account account, Model model) {
        if (account != null) {
            model.addAttribute(account);
        }

        model.addAttribute("studyList", studyRepository.findFirst9ByPublishedAndClosedOrderByPublishedDateTimeDesc(true, false));
        return "index";
    }

    ...
}
```
- StudyRepository에 `findFirst9ByPublishedAndClosedOrderByPublishedDateTimeDesc()` 추가
```java
@Transactional(readOnly = true)
public interface StudyRepository extends JpaRepository<Study, Long>, StudyRepositoryExtension {

    ...

    @EntityGraph(attributePaths = {"zones", "tags"})
    List<Study> findFirst9ByPublishedAndClosedOrderByPublishedDateTimeDesc(boolean published, boolean closed);
}
```
- 첫 페이지 뷰 수정
    *  부트스트랩의 Jumbotron을 사용
```html
<style>

    ...

    .jumbotron {
        padding-top: 3rem;
        padding-bottom: 3rem;
        margin-bottom: 0;
        background-color: #fff;
    }
    @media (min-width: 768px) {
        .jumbotron {
            padding-top: 6rem;
            padding-bottom: 6rem;
        }
    }
    .jumbotron p:last-child {
        margin-bottom: 0;
    }
    .jumbotron h1 {
        font-weight: 300;
    }
    .jumbotron .container {
        max-width: 40rem;
    }
</style>

...

<div th:fragment="study-list (studyList)" class="col-sm-10">
    <div class="row">
        <div class="col-md-4" th:each="study: ${studyList}">
            <div class="card mb-4 shadow-sm">
                <img th:src="${study.image}" class="card-img-top" th:alt="${study.title}" >
                <div class="card-body">
                    <a th:href="@{'/study/' + ${study.path}}" class="text-decoration-none">
                        <h5 class="card-title context" th:text="${study.title}"></h5>
                    </a>
                    <p class="card-text" th:text="${study.shortDescription}">Short description</p>
                    <p class="card-text context">
                                <span th:each="tag: ${study.tags}" class="font-weight-light text-monospace badge badge-pill badge-info mr-3">
                                    <a th:href="@{'/search/tag/' + ${tag.title}}" class="text-decoration-none text-white">
                                        <i class="fa fa-tag"></i> <span th:text="${tag.title}">Tag</span>
                                    </a>
                                </span>
                        <span th:each="zone: ${study.zones}" class="font-weight-light text-monospace badge badge-primary mr-3">
                                    <a th:href="@{'/search/zone/' + ${zone.id}}" class="text-decoration-none text-white">
                                        <i class="fa fa-globe"></i> <span th:text="${zone.localNameOfCity}" class="text-white">City</span>
                                    </a>
                                </span>
                    </p>
                    <div class="d-flex justify-content-between align-items-center">
                        <small class="text-muted">
                            <i class="fa fa-user-circle"></i>
                            <span th:text="${study.memberCount}"></span>명
                        </small>
                        <small class="text-muted date" th:text="${study.publishedDateTime}">9 mins</small>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
```
```html
<!DOCTYPE html>
<html lang="en"
      xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments.html :: head"></head>

<body class="bg-light">
    <div th:replace="fragments.html :: main-nav"></div>
    <section class="jumbotron text-center">
        <div class="container">
            <h1>스터디올래</h1>
            <p class="lead text-muted">
                태그와 지역 기반으로 스터디를 찾고 참여하세요.<br/>
                스터디 모임 관리 기능을 제공합니다.
            </p>
            <p>
                <a th:href="@{/signup}" class="btn btn-primary my-2">회원 가입</a>
            </p>
        </div>
    </section>
    <div class="container">
        <div class="row justify-content-center pt-3">
            <div th:replace="fragments.html :: study-list (studyList=${studyList})"></div>
        </div>
    </div>
    <div th:replace="fragments.html :: footer"></div>
    <div th:replace="fragments.html :: date-time"></div>
</body>
</html>
```
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/search_firstPage_5.jpg"></p>

<br>

## 첫 페이지 - 로그인 후
- 화면을 N+1 Select 문제 없이, 쿼리 6개를 사용해서 만들기
    * 계정 조회 (관심 주제, 지역 정보 포함)
    * 참석할 모임 조회
    * 나의 주요 활동 지역과 관심 주제에 해당하는 스터디 조회 (Tag와 Zone은 && 조건이다.)
    * 관리중인 스터디 조회
    * 참여중인 스터디 조회
    * 알림 조회
- 뷰에 전달해야 할 모델 데이터
    * account: 현재 로그인한 사용자의 정보
    * enrollmentList: 참석 확정된 참가 신청 정보를 통해 참석할 모임 목록 출력
    * studyList: 주요 활동 지역의 관심 주제 스터디 목록 출력
    * studyManagerOf: 관리중인 스터디 목록 출력
    * studyMemberOf: 참여중인 스터디 목록 출력
<br>

### 구현
- MainController에 맵핑 코드 수정
    * 모임 목록은 enrollment에서 가져온다.
```java
@Controller
@RequiredArgsConstructor
public class MainController {

    private final StudyRepository studyRepository;
    private final EnrollmentRepository enrollmentRepository;
    private final AccountRepository accountRepository;

    @GetMapping("/")
    public String home(@CurrentAccount Account account, Model model) {
        if (account != null) {
            // account는 detached 상태이므로 새롭게 받아온다.
            Account accountLoaded = accountRepository.findAccountWithTagsAndZonesById(account.getId());
            model.addAttribute(accountLoaded);
            // 참가신청을 했고, 수락된 모임을 정렬해서 가져온다.
            model.addAttribute("enrollmentList", enrollmentRepository.findByAccountAndAcceptedOrderByEnrolledAtDesc(accountLoaded, true));
            model.addAttribute("studyList", studyRepository.findByAccount(
                    accountLoaded.getTags(),
                    accountLoaded.getZones()));
            model.addAttribute("studyManagerOf",
                    studyRepository.findFirst5ByManagersContainingAndClosedOrderByPublishedDateTimeDesc(account, false));
            model.addAttribute("studyMemberOf",
                    studyRepository.findFirst5ByMembersContainingAndClosedOrderByPublishedDateTimeDesc(account, false));
            return "index-after-login";
        }

        model.addAttribute("studyList", studyRepository.findFirst9ByPublishedAndClosedOrderByPublishedDateTimeDesc(true, false));
        return "index";
    }

    ...

}
```
- AccountRepository에 `findAccountWithTagsAndZonesById()` 메서드 추가
```java
@Transactional(readOnly = true)
public interface AccountRepository extends JpaRepository<Account, Long>, QuerydslPredicateExecutor<Account> {

    ...

    @EntityGraph(attributePaths = {"tags", "zones"})
    Account findAccountWithTagsAndZonesById(Long id);
}
```
- EnrollmentRepository에 `findByAccountAndAcceptedOrderByEnrolledAtDesc()` 메서드 추가
    * find~ 메서드는 아무리 Enrollment가 `@ManyToOne`으로 Event와 Account 정보를 가지고 있다 하더라도 가져오지 않는다.
    * FETCH 모드는 JPQL로 쿼리가 만들어져서 조회해오는 경우 조회가 되지 않는다. 
    * 따라서 `@EntityGraph`를 사용해야 한다.
```java
@Transactional(readOnly = true)
public interface EnrollmentRepository extends JpaRepository<Enrollment, Long> {
    
    ...

    @EntityGraph("Enrollment.withEventAndStudy")
    List<Enrollment> findByAccountAndAcceptedOrderByEnrolledAtDesc(Account account, boolean accepted);
}
```
- Enrollment에 `@NamedEntityGraph` 애노테이션 추가
    * enrollment가 가지고 있는 event의 study까지 가져와야 하므로 `@NamedSubgraph`를 사용해야 한다.
    * 직접적인 연관관계가 있는 event, event가 가지고 있는 study까지 같이 가져오고 싶은 경우 subgraph를 사용한다.
```java
@NamedEntityGraph(
        name = "Enrollment.withEventAndStudy",
        attributeNodes = {
                @NamedAttributeNode(value = "event", subgraph = "study")
        },
        subgraphs = @NamedSubgraph(name = "study", attributeNodes = @NamedAttributeNode("study"))
)
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
public class Enrollment {

    ...

}
```
- 스터디는 QueryDSL을 사용하여 Account가 가지고 있는 Zone과 Tag에 대한 스터디 목록을 가져와야 한다.
    * 페이징이 필요없기 때문에 쿼리가 1개만 날아간다.
```java
@Transactional(readOnly = true)
public interface StudyRepositoryExtension {

    ...

    List<Study> findByAccount(Set<Tag> tags, Set<Zone> zones);
}
```
```java
public class StudyRepositoryExtensionImpl extends QuerydslRepositorySupport implements StudyRepositoryExtension {

    ...

    @Override
    public List<Study> findByAccount(Set<Tag> tags, Set<Zone> zones) {
        QStudy study = QStudy.study;
        JPQLQuery<Study> query = from(study).where(study.published.isTrue()
                .and(study.closed.isFalse())
                .and(study.tags.any().in(tags)) 
                .and(study.zones.any().in(zones)))
                .leftJoin(study.tags, QTag.tag).fetchJoin()
                .leftJoin(study.zones, QZone.zone).fetchJoin()
                .orderBy(study.publishedDateTime.desc())
                .distinct()
                .limit(9);
        return query.fetch();
    }
}
```
- StudyRepository에 스프링 데이터 JPA를 사용해 메서드 구현
    * `findFirst5ByManagersContainingAndClosedOrderByPublishedDateTimeDesc()`
    * `findFirst5ByMembersContainingAndClosedOrderByPublishedDateTimeDesc()`
    * `@EntityGraph`를 사용해 연관관계를 추가할 필요없다.
        - zone이나 tag정보를 안보여주고 스터디 이름과 링크만 필요하기 때문.
        - 스터디가 가지고 있는 기본정보들만 보여준다.
```java
@Transactional(readOnly = true)
public interface StudyRepository extends JpaRepository<Study, Long>, StudyRepositoryExtension {

    ...

    @EntityGraph(attributePaths = {"zones", "tags"})
    List<Study> findFirst9ByPublishedAndClosedOrderByPublishedDateTimeDesc(boolean published, boolean closed);

    List<Study> findFirst5ByManagersContainingAndClosedOrderByPublishedDateTimeDesc(Account account, boolean closed);

    List<Study> findFirst5ByMembersContainingAndClosedOrderByPublishedDateTimeDesc(Account account, boolean closed);
}
```
- 로그인 한 사용자를 위한 첫 페이지 뷰 생성
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments.html :: head"></head>
<body class="bg-light">
    <div th:replace="fragments.html :: main-nav"></div>
    <div class="alert alert-warning" role="alert" th:if="${account != null && !account?.emailVerified}">
        스터디올레 가입을 완료하려면 <a href="#" th:href="@{/check-email(email=${account.email})}" class="alert-link">계정 인증 이메일을 확인</a>하세요.
    </div>
    <div class="container mt-4">
        <div class="row">
            <div class="col-md-2">
                <h5 class="font-weight-light">관심 스터디 주제</h5>
                <ul class="list-group list-group-flush">
                    <li class="list-group-item" th:each="tag: ${account.tags}">
                        <i class="fa fa-tag"></i> <span th:text="${tag.title}"></span>
                    </li>
                    <li class="list-group-item" th:if="${account.tags.size() == 0}">
                        <a th:href="@{/settings/tags}" class="btn-text">관심 스터디 주제</a>를 등록하세요.
                    </li>
                </ul>
                <h5 class="mt-3 font-weight-light">주요 활동 지역</h5>
                <ul class="list-group list-group-flush">
                    <li class="list-group-item" th:each="zone: ${account.zones}">
                        <i class="fa fa-globe"></i> <span th:text="${zone.getLocalNameOfCity()}">Zone</span>
                    </li>
                    <li class="list-group-item" th:if="${account.zones.size() == 0}">
                        <a th:href="@{/settings/zones}" class="btn-text">주요 활동 지역</a>을 등록하세요.
                    </li>
                </ul>
            </div>
            <div class="col-md-7">
                <h5 th:if="${#lists.isEmpty(enrollmentList)}" class="font-weight-light">참석할 모임이 없습니다.</h5>
                <h5 th:if="${!#lists.isEmpty(enrollmentList)}" class="font-weight-light">참석할 모임</h5>
                <div class="row row-cols-1 row-cols-md-2" th:if="${!#lists.isEmpty(enrollmentList)}">
                    <div class="col mb-4" th:each="enrollment: ${enrollmentList}">
                        <div class="card">
                            <div class="card-body">
                                <h5 class="card-title" th:text="${enrollment.event.title}">Event title</h5>
                                <h6 class="card-subtitle mb-2 text-muted" th:text="${enrollment.event.study.title}">Study title</h6>
                                <p class="card-text">
                                    <span>
                                        <i class="fa fa-calendar-o"></i>
                                        <span class="calendar" th:text="${enrollment.event.startDateTime}">Last updated 3 mins ago</span>
                                    </span>
                                </p>
                                <a th:href="@{'/study/' + ${enrollment.event.study.path} + '/events/' + ${enrollment.event.id}}" class="card-link">모임 조회</a>
                                <a th:href="@{'/study/' + ${enrollment.event.study.path}}" class="card-link">스터디 조회</a>
                            </div>
                        </div>
                    </div>
                </div>
                <h5 class="font-weight-light mt-3" th:if="${#lists.isEmpty(studyList)}">관련 스터디가 없습니다.</h5>
                <h5 class="font-weight-light mt-3" th:if="${!#lists.isEmpty(studyList)}">주요 활동 지역의 관심 주제 스터디</h5>
                <div class="row justify-content-center">
                    <div th:replace="fragments.html :: study-list (studyList=${studyList})"></div>
                </div>
            </div>
            <div class="col-md-3">
                <h5 class="font-weight-light" th:if="${#lists.isEmpty(studyManagerOf)}">관리중인 스터디가 없습니다.</h5>
                <h5 class="font-weight-light" th:if="${!#lists.isEmpty(studyManagerOf)}">관리중인 스터디</h5>
                <div class="list-group" th:if="${!#lists.isEmpty(studyManagerOf)}">
                    <a href="#" th:href="@{'/study/' + ${study.path}}" th:text="${study.title}"
                       class="list-group-item list-group-item-action" th:each="study: ${studyManagerOf}">
                        Study title
                    </a>
                </div>

                <h5 class="font-weight-light mt-3" th:if="${#lists.isEmpty(studyMemberOf)}">참여중인 스터디가 없습니다.</h5>
                <h5 class="font-weight-light mt-3" th:if="${!#lists.isEmpty(studyMemberOf)}">참여중인 스터디</h5>
                <div class="list-group" th:if="${!#lists.isEmpty(studyMemberOf)}">
                    <a href="#" th:href="@{'/study/' + ${study.path}}" th:text="${study.title}"
                       class="list-group-item list-group-item-action" th:each="study: ${studyManagerOf}">
                        Study title
                    </a>
                </div>
            </div>
        </div>
    </div>
    <div th:replace="fragments.html :: footer"></div>
    <div th:replace="fragments.html :: date-time"></div>
</body>
</html>
```
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/search_firstPage_6.jpg"></p>
