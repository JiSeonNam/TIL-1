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
        JPQLQuery<Study> pageableQuery = getQuerydsl().applyPagination(pageable, query); // pagenation 적용
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
