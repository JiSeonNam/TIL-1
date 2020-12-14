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
