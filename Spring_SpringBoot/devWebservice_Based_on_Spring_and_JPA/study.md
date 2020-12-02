# 스터디

### 주요 기능
- 스터디 만들기
- 스터디 공개 및 종료
- 스터디 인원 모집
- 스터디 설정
    * 배너 이미지
    * 스터디 주제 (Tag)
    * 활동 지역 (Zone)
    * 스터디 관리 (공개 / 경로 변경 / 이름 변경 / 스터디 삭제)
- 스터디 참여 / 떠나기
<br>

## 스터디 도메인
- 객체 관점에서 Study와 다른 엔티티의 관계
    * Study에서 Account 쪽으로 @ManyToMany 단방향 관계 두 개 (managers, members)
    * Study에서 Zone으로 @ManyToMany 단방향 관계
    * Study에서 Tag로 @ManyToMany 단방향 관계
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/study_1.jpg"></p>

```java
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
@Builder @AllArgsConstructor @NoArgsConstructor
public class Study {

    @Id @GeneratedValue
    private Long id;

    @ManyToMany
    private Set<Account> managers = new HashSet<>(); // 스터디를 관리하는 스터디장, 매니저

    @ManyToMany
    private Set<Account> members = new HashSet<>(); // 스터디원

    @Column(unique = true)
    private String path; // URL 경로

    private String title; // 제목

    private String shortDescription; // 짧은 소개

    @Lob @Basic(fetch = FetchType.EAGER)
    private String fullDescription; // 상세 소개

    @Lob @Basic(fetch = FetchType.EAGER)
    private String image; // 배너 이미지

    @ManyToMany
    private Set<Tag> tags = new HashSet<>(); // 관심 주제(태그)

    @ManyToMany
    private Set<Zone> zones = new HashSet<>(); // 지역 정보

    private LocalDateTime publishedDateTime; // 스터디 공개 시간

    private LocalDateTime closedDateTime; // 스터디 종료 시간

    private LocalDateTime recruitingUpdatedDateTime; // 인원 모집 시간

    private boolean recruiting; // 인원 모집 중인지 여부

    private boolean published; // 공개 여부

    private boolean closed; // 종료 여부

    private boolean useBanner; // 배너 사용 여부
}
```
<br>

## 스터디 개설

### 스터디 생성
- StudyController 생성
```java
@Controller
public class StudyController {

    @GetMapping("/new-study")
    public String newStudyForm(@CurrentAccount Account account, Model model) {
        model.addAttribute(account);
        model.addAttribute(new StudyForm());

        return "study/form";
    }
}
```
- StudyForm 생성
```java
@Data
public class StudyForm {

    @NotBlank
    @Length(min = 2, max = 20)
    @Pattern(regexp = "^[ㄱ-ㅎ가-힣a-z0-9_-]{2,20}$")
    private String path;

    @NotBlank
    @Length(max = 50)
    private String title;

    @NotBlank
    @Length(max = 100)
    private String shortDescription;

    @NotBlank
    private String fullDescription;

}
```
- 스터디 정보를 입력 받기 위해 StudyForm을 보여줄 뷰 생성
    * 상세 소개를 에디터로 작성하기 위해 이지윅 에디터를 사용한다.
        - 그 중에서도 [Summernote](https://summernote.org/)
        - 부트스트랩과 연동이 편리하다.
        - 한국 개발자들의 오픈 소스이다.
        - `npm install summernote`
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments.html :: head"></head>
<body class="bg-light">
    <div th:replace="fragments.html :: main-nav"></div>

    <div class="container">
        <div class="py-5 text-center">
            <h2>스터디 개설</h2>
        </div>
        <div class="row justify-content-center">
            <form class="needs-validation col-sm-10" th:action="@{/new-study}" th:object="${studyForm}" method="post" novalidate>
                <div class="form-group">
                    <label for="path">스터디 URL</label>
                    <input id="path" type="text" th:field="*{path}" class="form-control"
                           placeholder="예) study-path" aria-describedby="pathHelp" required min="2" max="20">
                    <small id="pathHelp" class="form-text text-muted">
                        공백없이 문자, 숫자, 대시(-)와 언더바(_)만 2자 이상 20자 이내로 입력하세요. 스터디 홈 주소에 사용합니다. 예) /study/<b>study-path</b>
                    </small>
                    <small class="invalid-feedback">스터디 경로를 입력하세요.</small>
                    <small class="form-text text-danger" th:if="${#fields.hasErrors('path')}" th:errors="*{path}">Path Error</small>
                </div>

                <div class="form-group">
                    <label for="title">스터디 이름</label>
                    <input id="title" type="text" th:field="*{title}" class="form-control"
                           placeholder="스터디 이름" aria-describedby="titleHelp" required max="50">
                    <small id="titleHelp" class="form-text text-muted">
                        스터디 이름을 50자 이내로 입력하세요.
                    </small>
                    <small class="invalid-feedback">스터디 이름을 입력하세요.</small>
                    <small class="form-text text-danger" th:if="${#fields.hasErrors('title')}" th:errors="*{title}">Title Error</small>
                </div>

                <div class="form-group">
                    <label for="shortDescription">짧은 소개</label>
                    <textarea id="shortDescription" type="textarea" th:field="*{shortDescription}" class="form-control"
                              placeholder="스터디를 짧게 소개해 주세요." aria-describedby="shortDescriptionHelp" required maxlength="100"></textarea>
                    <small id="shortDescriptionHelp" class="form-text text-muted">
                        100자 이내로 스터디를 짧은 소개해 주세요.
                    </small>
                    <small class="invalid-feedback">짧은 소개를 입력하세요.</small>
                    <small class="form-text text-danger" th:if="${#fields.hasErrors('shortDescription')}" th:errors="*{shortDescription}">ShortDescription Error</small>
                </div>

                <div class="form-group">
                    <label for="fullDescription">상세 소개</label>
                    <textarea id="fullDescription" type="textarea" th:field="*{fullDescription}" class="form-control"
                              placeholder="스터디를 자세히 설명해 주세요." aria-describedby="fullDescriptionHelp" required></textarea>
                    <small id="fullDescriptionHelp" class="form-text text-muted">
                        스터디의 목표, 일정, 진행 방식, 사용할 교재 또는 인터넷 강좌 그리고 모집중인 스터디원 등 스터디에 대해 자세히 적어 주세요.
                    </small>
                    <small class="invalid-feedback">상세 소개를 입력하세요.</small>
                    <small class="form-text text-danger" th:if="${#fields.hasErrors('fullDescription')}" th:errors="*{fullDescription}">FullDescription Error</small>
                </div>

                <div class="form-group">
                    <button class="btn btn-primary btn-block" type="submit"
                            aria-describedby="submitHelp">스터디 만들기</button>
                </div>
            </form>
        </div>

        <div class="fragments.html :: footer"></div>
    </div>
    <script th:replace="fragments.html :: form-validation"></script>
    <script src="/node_modules/summernote/dist/summernote-bs4.js"></script>
    <script type="application/javascript">
        $(function () {
            $('#fullDescription').summernote({
                fontNames: ['Arial', 'Arial Black', 'Comic Sans MS', 'Courier New', 'Noto Sans KR', 'Merriweather'],
                placeholder: '스터디의 목표, 일정, 진행 방식, 사용할 교재 또는 인터넷 강좌 그리고 모집중인 스터디원 등 스터디에 대해 자세히 적어 주세요.',
                tabsize: 2,
                height: 300
            });
        });
    </script>
</body>
</html>
```
- summernote stylesheet 적용
```html
<link rel="stylesheet" href="/node_modules/summernote/dist/summernote-bs4.min.css">
```
- 실행해서 스터디 개설 버튼을 클릭하면 다음과 같이 스터디를 생성할 수 있는 페이지가 나온다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/study_2.jpg"></p>

- **계속해서 스터디 만들기 submit 버튼을 눌러 스터리를 생성한다면?**
- StudyController POST 맵핑 추가
    * `URLEncoder.encode()`
        - 스터디의 path가 한글이 있는 경우도 있기 때문에 encoding을 해줘야 한다.
```java
@Controller
@RequiredArgsConstructor
public class StudyController {

    private final StudyService studyService;
    private final ModelMapper modelMapper;
    
    ...

    @PostMapping("/new-study")
    public String newStudySubmit(@CurrentAccount Account account, @Valid StudyForm studyForm, Errors errors) {
        if (errors.hasErrors()) {
            model.addAttribute(account);
            return "study/form";
        }

        Study newStudy = studyService.createNewStudy(modelMapper.map(studyForm, Study.class), account);
        return "redirect:/study/" + URLEncoder.encode(newStudy.getPath(), StandardCharsets.UTF_8);
    }

}
```
- StudyService 생성
```java
@Service
@Transactional
@RequiredArgsConstructor
public class StudyService {

    private final StudyRepository repository;

    public Study createNewStudy(Study study, Account account) {
        Study newStudy = repository.save(study);
        newStudy.addManager(account);
        return newStudy;
    }
}
```
- StudyRepository 생성
```java
@Transactional(readOnly = true)
public interface StudyRepository extends JpaRepository<Study, Long> {

    boolean existsByPath(String path);

}
```
- Study 엔티티에 `addManager()` 생성
```java
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
@Builder @AllArgsConstructor @NoArgsConstructor
public class Study {

    ...

    public void addManager(Account account) {
        this.managers.add(account);
    }

    public void addMember(Account account) {
        this.members.add(account);
    }
}
```
- StudyForm을 검증하기 위한 Validator 생성
```java
@Component
@RequiredArgsConstructor
public class StudyFormValidator implements Validator {

    private final StudyRepository studyRepository;

    @Override
    public boolean supports(Class<?> clazz) {
        return StudyForm.class.isAssignableFrom(clazz);
    }

    @Override
    public void validate(Object target, Errors errors) {
        StudyForm studyForm = (StudyForm)target;
        if (studyRepository.existsByPath(studyForm.getPath())) { // 경로가 있는지 없는지만 validation
            errors.rejectValue("path", "wrong.path", "해당 스터디 경로 값을 사용할 수 없습니다.");
        }
    }
}
```
- StudyController에 커스텀 Validator 등록
```java
@Controller
@RequiredArgsConstructor
public class StudyController {

    ...
    private final StudyFormValidator studyFormValidator;

    @InitBinder("studyForm")
    public void studyFormInitBinder(WebDataBinder webDataBinder) {
        webDataBinder.addValidators(studyFormValidator);
    }

    ...

}
```
<br>

## 스터디 조회
- 스터디 조회 맵핑
```java
@Controller
@RequiredArgsConstructor
public class StudyController {

    ...
    private final StudyRepository studyRepository;

    ...

    @GetMapping("/study/{path}")
    public String viewStudy(@CurrentAccount Account account, @PathVariable String path, Model model) {
        model.addAttribute(account);
        model.addAttribute(studyRepository.findByPath(path));
        return "study/view";
    }
}
```
- Study 엔티티에 메서드 추가
```java
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
@Builder @AllArgsConstructor @NoArgsConstructor
public class Study {

    ...

    public boolean isJoinable(UserAccount userAccount) { // 가입 가능한지
        Account account = userAccount.getAccount();
        return this.isPublished() && this.isRecruiting()
                && !this.members.contains(account) && !this.managers.contains(account);

    }

    public boolean isMember(UserAccount userAccount) { // 스터디 구성원인지
        return this.members.contains(userAccount.getAccount());
    }

    public boolean isManager(UserAccount userAccount) { // 스터디 관리자인지
        return this.managers.contains(userAccount.getAccount());
    }
}
```
- 스터디 조회 뷰 생성
```html
<!-- fragments.html-->
<div th:fragment="study-banner" th:if="${study.useBanner}" class="row" id="study-logo">
    <img th:src="${study.image}"/>
</div>

<div th:fragment="study-info">
    <div class="row pt-4 text-left justify-content-center bg-light">
        <div class="col-6">
            <a href="#" class="text-decoration-none" th:href="@{'/study/' + ${study.path}}">
                <span class="h2" th:text="${study.title}">스터디 이름</span>
            </a>
        </div>
        <div class="col-4 text-right justify-content-end">
                <span th:if="${!study.published}"
                      class="d-inline-block" tabindex="0" data-toggle="tooltip" data-placement="bottom"
                      title="스터디 공개 준비중">
                    <button class="btn btn-primary btn-sm" style="pointer-events: none;" type="button" disabled>DRAFT</button>
                </span>
            <span th:if="${study.closed}"
                  class="d-inline-block" tabindex="0" data-toggle="tooltip" data-placement="bottom" title="스터디 종료함">
                    <button class="btn btn-primary btn-sm" style="pointer-events: none;" type="button" disabled>CLOSED</button>
                </span>
            <span th:if="${!study.recruiting}"
                  class="d-inline-block ml-1" tabindex="0" data-toggle="tooltip" data-placement="bottom" title="팀원 모집중 아님">
                    <button class="btn btn-primary btn-sm" style="pointer-events: none;" type="button" disabled>OFF</button>
                </span>
            <span sec:authorize="isAuthenticated()" th:if="${study.isJoinable(#authentication.principal)}"
                  class="btn-group" role="group" aria-label="Basic example">
                    <a class="btn btn-primary" th:href="@{'/study/' + ${study.path} + '/join'}">
                        스터디 가입
                    </a>
                    <a class="btn btn-outline-primary" th:href="@{'/study/' + ${study.path} + '/members'}"
                       th:text="${study.members.size()}">1</a>
                </span>
            <span sec:authorize="isAuthenticated()"
                  th:if="${!study.closed && study.isMember(#authentication.principal)}" class="btn-group" role="group">
                    <a class="btn btn-outline-warning" th:href="@{'/study/' + ${study.path} + '/leave'}">
                        스터디 탈퇴
                    </a>
                    <a class="btn btn-outline-primary" th:href="@{'/study/' + ${study.path} + '/members'}"
                       th:text="${study.members.size()}">1</a>
                </span>
            <span sec:authorize="isAuthenticated()"
                  th:if="${study.published && !study.closed && study.isManager(#authentication.principal)}">
                    <a class="btn btn-outline-primary" th:href="@{'/study/' + ${study.path} + '/new-event'}">
                        <i class="fa fa-plus"></i> 모임 만들기
                    </a>
                </span>
        </div>
    </div>
    <div class="row justify-content-center bg-light">
        <div class="col-10">
            <p class="lead" th:text="${study.shortDescription}"></p>
        </div>
    </div>
    <div class="row justify-content-center bg-light">
        <div class="col-10">
            <p>
                <span th:each="tag: ${study.tags}"
                      class="font-weight-light text-monospace badge badge-pill badge-info mr-3">
                    <a th:href="@{'/search/tag/' + ${tag.title}}" class="text-decoration-none text-white">
                        <i class="fa fa-tag"></i> <span th:text="${tag.title}">Tag</span>
                    </a>
                </span>
                <span th:each="zone: ${study.zones}" class="font-weight-light text-monospace badge badge-primary mr-3">
                    <a th:href="@{'/search/zone/' + ${zone.id}}" class="text-decoration-none text-white">
                        <i class="fa fa-globe"></i> <span th:text="${zone.localNameOfCity}">City</span>
                    </a>
                </span>
            </p>
        </div>
    </div>
</div>

<div th:fragment="study-menu (studyMenu)" class="row px-3 justify-content-center bg-light">
    <nav class="col-10 nav nav-tabs">
        <a class="nav-item nav-link" href="#" th:classappend="${studyMenu == 'info'}? active" th:href="@{'/study/' + ${study.path}}">
            <i class="fa fa-info-circle"></i> 소개
        </a>
        <a class="nav-item nav-link" href="#" th:classappend="${studyMenu == 'members'}? active" th:href="@{'/study/' + ${study.path} + '/members'}">
            <i class="fa fa-user"></i> 구성원
        </a>
        <a class="nav-item nav-link" th:classappend="${studyMenu == 'events'}? active" href="#" th:href="@{'/study/' + ${study.path} + '/events'}">
            <i class="fa fa-calendar"></i> 모임
        </a>
        <a sec:authorize="isAuthenticated()" th:if="${study.isManager(#authentication.principal)}"
           class="nav-item nav-link" th:classappend="${studyMenu == 'settings'}? active" href="#" th:href="@{'/study/' + ${study.path} + '/settings/description'}">
            <i class="fa fa-cog"></i> 설정
        </a>
    </nav>
</div>
```
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments.html :: head"></head>
<body class="bg-light">
    <div th:replace="fragments.html :: main-nav"></div>
    <div th:replace="fragments.html :: study-banner"></div>
    <div class="container">
        <div th:replace="fragments.html :: study-info"></div>
        <div th:replace="fragments.html :: study-menu(studyMenu='info')"></div>

        <div class="row px-3 justify-content-center">
            <div class="col-10 pt-3" th:utext="${study.fullDescription}"></div>
        </div>

        <div th:replace="fragments.html :: footer"></div>
    </div>
    <script type="application/javascript">
        $(function () {
            $('[data-toggle="tooltip"]').tooltip()
        })
    </script>
</body>
</html>
```
- 실행해서 스터디를 개설하면 다음과 같이 스터디 조회 화면이 나온다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/study_3.jpg"></p>

<br>

### 성능 개선
- 현재 스터디를 조회할 경우 쿼리가 5개 발생한다. 
    * 스터디 조회 1개
    * member인지 확인 1개
    * manager인지 확인 1개
    * 스터디의 tag 1개
    * 스터디의 zone 1개
- 스터디를 조회할 경우 연결되어 있는 데이터를 가져와야 한다.
    * 따라서 어차피 조회할 데이터라면 쿼리 개수를 줄이고 join을 해서 가져오자.
    * Left outer join으로 연관 데이터를 한번에 조회할 수도 있다.
- **Entity Graph 정의**
```java
@NamedEntityGraph(name = "Study.withAll", attributeNodes = {
        @NamedAttributeNode("tags"),
        @NamedAttributeNode("zones"),
        @NamedAttributeNode("managers"),
        @NamedAttributeNode("members")})
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
@Builder @AllArgsConstructor @NoArgsConstructor
public class Study {
    
    ...

}
```
```java
@Transactional(readOnly = true)
public interface StudyRepository extends JpaRepository<Study, Long> {

    ...
    
    // EntityGraph에 명시한 연관관계는 EAGER로 가져오고 나머지는 기본 FetchType에 따른다.
    @EntityGraph(value = "Study.withAll", type = EntityGraph.EntityGraphType.LOAD)
    Study findByPath(String path);
}
```
- 실행해보면 한꺼번에 데이터를 가져온다.
<br>

### 테스트 코드 작성
```java
class StudyTest {

    Study study;
    Account account;
    UserAccount userAccount;

    @BeforeEach
    void beforeEach() {
        study = new Study();
        account = new Account();
        account.setNickname("hayoung");
        account.setPassword("123");
        userAccount = new UserAccount(account);

    }

    @DisplayName("스터디를 공개했고 인원 모집 중이고, 이미 멤버나 스터디 관리자가 아니라면 스터디 가입 가능")
    @Test
    void isJoinable() {
        study.setPublished(true);
        study.setRecruiting(true);

        assertTrue(study.isJoinable(userAccount));
    }

    @DisplayName("스터디를 공개했고 인원 모집 중이더라도, 스터디 관리자는 스터디 가입이 불필요하다.")
    @Test
    void isJoinable_false_for_manager() {
        study.setPublished(true);
        study.setRecruiting(true);
        study.addManager(account);

        assertFalse(study.isJoinable(userAccount));
    }

    @DisplayName("스터디를 공개했고 인원 모집 중이더라도, 스터디 멤버는 스터디 재가입이 불필요하다.")
    @Test
    void isJoinable_false_for_member() {
        study.setPublished(true);
        study.setRecruiting(true);
        study.addMember(account);

        assertFalse(study.isJoinable(userAccount));
    }

    @DisplayName("스터디가 비공개거나 인원 모집 중이 아니면 스터디 가입이 불가능하다.")
    @Test
    void isJoinable_false_for_non_recruiting_study() {
        study.setPublished(true);
        study.setRecruiting(false);

        assertFalse(study.isJoinable(userAccount));

        study.setPublished(false);
        study.setRecruiting(true);

        assertFalse(study.isJoinable(userAccount));
    }

    @DisplayName("스터디 관리자인지 확인")
    @Test
    void isManager() {
        study.addManager(account);
        assertTrue(study.isManager(userAccount));
    }

    @DisplayName("스터디 멤버인지 확인")
    @Test
    void isMember() {
        study.addMember(account);
        assertTrue(study.isMember(userAccount));
    }

}
```
```java
@Transactional
@SpringBootTest
@AutoConfigureMockMvc
@RequiredArgsConstructor
class StudyControllerTest {

    @Autowired MockMvc mockMvc;
    @Autowired StudyService studyService;
    @Autowired StudyRepository studyRepository;
    @Autowired AccountRepository accountRepository;

    @AfterEach
    void afterEach() {
        accountRepository.deleteAll();
    }

    @Test
    @WithAccount("hayoung")
    @DisplayName("스터디 개설 폼 조회")
    void createStudyForm() throws Exception {
        mockMvc.perform(get("/new-study"))
                .andExpect(status().isOk())
                .andExpect(view().name("study/form"))
                .andExpect(model().attributeExists("account"))
                .andExpect(model().attributeExists("studyForm"));
    }

    @Test
    @WithAccount("hayoung")
    @DisplayName("스터디 개설 - 완료")
    void createStudy_success() throws Exception {
        mockMvc.perform(post("/new-study")
                .param("path", "test-path")
                .param("title", "study title")
                .param("shortDescription", "short description of a study")
                .param("fullDescription", "full description of a study")
                .with(csrf()))
                .andExpect(status().is3xxRedirection())
                .andExpect(redirectedUrl("/study/test-path"));

        Study study = studyRepository.findByPath("test-path");
        assertNotNull(study);
        Account account = accountRepository.findByNickname("hayoung");
        assertTrue(study.getManagers().contains(account));
    }

    @Test
    @WithAccount("hayoung")
    @DisplayName("스터디 개설 - 실패")
    void createStudy_fail() throws Exception {
        mockMvc.perform(post("/new-study")
                .param("path", "wrong path")
                .param("title", "study title")
                .param("shortDescription", "short description of a study")
                .param("fullDescription", "full description of a study")
                .with(csrf()))
                .andExpect(status().isOk())
                .andExpect(view().name("study/form"))
                .andExpect(model().hasErrors())
                .andExpect(model().attributeExists("studyForm"))
                .andExpect(model().attributeExists("account"));

        Study study = studyRepository.findByPath("test-path");
        assertNull(study);
    }

    @Test
    @WithAccount("hayoung")
    @DisplayName("스터디 조회")
    void viewStudy() throws Exception {
        Study study = new Study();
        study.setPath("test-path");
        study.setTitle("test study");
        study.setShortDescription("short description");
        study.setFullDescription("<p>full description</p>");

        Account hayoung = accountRepository.findByNickname("hayoung");
        studyService.createNewStudy(study, hayoung);

        mockMvc.perform(get("/study/test-path"))
                .andExpect(view().name("study/view"))
                .andExpect(model().attributeExists("account"))
                .andExpect(model().attributeExists("study"));
    }
}
```
<br>

## 스터디 구성원 조회
- 구성원 조회 맵핑 추가
```java
@Controller
@RequiredArgsConstructor
public class StudyController {

    ...

    @GetMapping("/study/{path}/members")
    public String viewStudyMembers(@CurrentAccount Account account, @PathVariable String path, Model model) {
        model.addAttribute(account);
        model.addAttribute(studyRepository.findByPath(path));
        return "study/members";
    }
}
```
- 구성원 조회 뷰 추가
```html
<!-- fragments.html -->
<div th:fragment="member-list (members, isManager)" class="row px-3 justify-content-center">
    <ul class="list-unstyled col-10">
        <li class="media mt-3" th:each="member: ${members}">
            <svg th:if="${#strings.isEmpty(member?.profileImage)}" th:data-jdenticon-value="${member.nickname}" width="64" height="64" class="rounded border bg-light mr-3"></svg>
            <img th:if="${!#strings.isEmpty(member?.profileImage)}" th:src="${member?.profileImage}" width="64" height="64" class="rounded border mr-3"/>
            <div class="media-body">
                <h5 class="mt-0 mb-1"><span th:text="${member.nickname}"></span> <span th:if="${isManager}" class="badge badge-primary">관리자</span></h5>
                <span th:text="${member.bio}"></span>
            </div>
        </li>
    </ul>
</div>
```
```html
<!-- members.html -->
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments.html :: head"></head>
<body class="bg-light">
    <div th:replace="fragments.html :: main-nav"></div>
    <div th:replace="fragments.html :: study-banner"></div>
    <div class="container">
        <div th:replace="fragments.html :: study-info"></div>
        <div th:replace="fragments.html :: study-menu(studyMenu='members')"></div>
    
        <div th:replace="fragments.html :: member-list(members=${study.managers},isManager=${true})"></div>
        <div th:replace="fragments.html :: member-list(members=${study.members},isManager=${false})"></div>
    
        <div th:replace="fragments.html :: footer"></div>
    </div>
    <script type="application/javascript">
        $(function () {
            $('[data-toggle="tooltip"]').tooltip()
        })
    </script>
</body>
</html> 
```
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/study_4.jpg"></p>

<br>

## 스터디 설정 - 소개 수정
- 스터디 설정 관련 메뉴가 많아서 컨트롤러를 분리
<br>

### 구현
- account와 study 확인하는 작업을 Service로 위임
    * account와 study를 getter로 가져오는 과정에서 없을 경우 처리를 해준다.
```java
@Service
@Transactional
@RequiredArgsConstructor
public class StudyService {

    ...

    public Study getStudy(String path) {
        Study study = this.repository.findByPath(path);
        if (study == null) {
            throw new IllegalArgumentException(path + "에 해당하는 스터디가 없습니다.");
        }

        return study;
    }
}
```
```java
@Controller
@RequiredArgsConstructor
public class StudyController {

    ...

    @GetMapping("/study/{path}")
    public String viewStudy(@CurrentAccount Account account, @PathVariable String path, Model model) {
        Study study = studyService.getStudy(path);

        model.addAttribute(account);
        model.addAttribute(study);
        return "study/view";
    }

    @GetMapping("/study/{path}/members")
    public String viewStudyMembers(@CurrentAccount Account account, @PathVariable String path, Model model) {
        Study study = studyService.getStudy(path);

        model.addAttribute(account);
        model.addAttribute(study);
        return "study/members";
    }
}
```
```java
@Service
@Transactional
@RequiredArgsConstructor
public class AccountService implements UserDetailsService {

    ...

    public Account getAccount(String nickname) {
        Account account = accountRepository.findByNickname(nickname);
        if (account == null) {
            throw new IllegalArgumentException(nickname + "에 해당하는 사용자가 없습니다.");
        }
        return account;
    }
}
```
```java
@Controller
@RequiredArgsConstructor
public class AccountController {

    ...

    @GetMapping("/profile/{nickname}")
    public String viewProfile(@PathVariable String nickname, Model model, @CurrentAccount Account account) {
        Account accountToView = accountService.getAccount(nickname);
        model.addAttribute(accountToView);
        model.addAttribute("isOwner", accountToView.equals(account));

        model.addAttribute(accountToView);
        model.addAttribute("isOwner", accountToView.equals(account));
        return "account/profile";

    }

    ...
}
```
- AccountService에 스터디 수정을 하는 것이 매니저인지 확인
    * account가 manager인지 확인
```java
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
@Builder @AllArgsConstructor @NoArgsConstructor
public class Account {

    ...

    public boolean isManagerOf(Study study) {
        return study.getManagers().contains(this);
    }
}
```
```java
@Service
@Transactional
@RequiredArgsConstructor
public class StudyService {

    ...

    public Study getStudyToUpdate(Account account, String path) {
        Study study = this.getStudy(path);
        if (!account.isManagerOf(study)) {
            throw new AccessDeniedException("해당 기능을 사용할 수 없습니다.");
        }

        return study;
    }
}
```
- 스터디 소개 수정을 위한 폼 생성
```java
@Data
@NoArgsConstructor
public class StudyDescriptionForm {

    @NotBlank
    @Length(max = 100)
    private String shortDescription;

    @NotBlank
    private String fullDescription;

}
```
- 스터디 소개 수정 맵핑
```java
@Controller
@RequestMapping("/study/{path}/settings")
@RequiredArgsConstructor
public class StudySettingsController {

    private final StudyService studyService;
    private final ModelMapper modelMapper;

    @GetMapping("/description")
    public String viewStudySetting(@CurrentAccount Account account, @PathVariable String path, Model model) {
        Study study = studyService.getStudyToUpdate(account, path);
        model.addAttribute(account);
        model.addAttribute(study);
        // StudyDescriptionForm에 대한 정보를 modelMapper로 생성해서 화면에 전달
        model.addAttribute(modelMapper.map(study, StudyDescriptionForm.class));
        return "study/settings/description";
    }

    @PostMapping("/description")
    public String updateStudyInfo(@CurrentAccount Account account, @PathVariable String path,
                                  @Valid StudyDescriptionForm studyDescriptionForm, Errors errors,
                                  Model model, RedirectAttributes attributes) {
        Study study = studyService.getStudyToUpdate(account, path);
        // 여기서 study의 상태는 persist 상태이다.
        // StudyService는 Transaction 안에서 가져온 것이다.

        if (errors.hasErrors()) {
            // 폼에서 받았던 데이터와 에러는 model에 기본으로 담아주지만 account와 study는 담아줘야 한다.
            model.addAttribute(account);
            model.addAttribute(study);
            return "study/settings/description";
        }

        studyService.updateStudyDescription(study, studyDescriptionForm);
        attributes.addFlashAttribute("message", "스터디 소개를 수정했습니다.");
        return "redirect:/study/" + getPath(path) + "/settings/description";
    }

    private String getPath(String path) {
        return URLEncoder.encode(path, StandardCharsets.UTF_8);
    }
}
```
- StudyService에 스터디 정보를 수정하는 `updateStudyDescription()` 생성
```java
@Service
@Transactional
@RequiredArgsConstructor
public class StudyService {

    ...
    private final ModelMapper modelMapper;

    ...

    public void updateStudyDescription(Study study, StudyDescriptionForm studyDescriptionForm) {
        modelMapper.map(studyDescriptionForm, study);
    }
}
```
- 스터디 소개 수정 뷰 생성
```html
<!-- fragments.html-->
<div th:fragment="study-settings-menu (currentMenu)" class="list-group">
    <a class="list-group-item list-group-item-action" th:classappend="${currentMenu == 'description'}? active"
       href="#" th:href="@{'/study/' + ${study.path} + '/settings/description'}">소개</a>
    <a class="list-group-item list-group-item-action" th:classappend="${currentMenu == 'image'}? active"
       href="#" th:href="@{'/study/' + ${study.path} + '/settings/image'}">배너 이미지</a>
    <a class="list-group-item list-group-item-action" th:classappend="${currentMenu == 'tags'}? active"
       href="#" th:href="@{'/study/' + ${study.path} + '/settings/tags'}">스터디 주제</a>
    <a class="list-group-item list-group-item-action" th:classappend="${currentMenu == 'zones'}? active"
       href="#" th:href="@{'/study/' + ${study.path} + '/settings/zones'}">활동 지역</a>
    <a class="list-group-item list-group-item-action list-group-item-danger" th:classappend="${currentMenu == 'study'}? active"
       href="#" th:href="@{'/study/' + ${study.path} + '/settings/study'}">스터디</a>
</div>

<div th:fragment="editor-script">
    <script src="/node_modules/summernote/dist/summernote-bs4.js"></script>
    <script type="application/javascript">
        $(function () {
            $('#fullDescription').summernote({
                fontNames: ['Arial', 'Arial Black', 'Comic Sans MS', 'Courier New', 'Noto Sans KR', 'Merriweather'],
                placeholder: '스터디의 목표, 일정, 진행 방식, 사용할 교재 또는 인터넷 강좌 그리고 모집중인 스터디원 등 스터디에 대해 자세히 적어 주세요.',
                tabsize: 2,
                height: 300
            });
        });
    </script>
</div>

<div th:fragment="message" th:if="${message}" class="alert alert-info alert-dismissible fade show mt-3" role="alert">
    <span th:text="${message}">완료</span>
    <button type="button" class="close" data-dismiss="alert" aria-label="Close">
        <span aria-hidden="true">&times;</span>
    </button>
</div>
```
```html
<!-- description.html-->
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments.html :: head"></head>
<body>
    <nav th:replace="fragments.html :: main-nav"></nav>
    <div th:replace="fragments.html :: study-banner"></div>
    <div class="container">
        <div th:replace="fragments.html :: study-info"></div>
        <div th:replace="fragments.html :: study-menu(studyMenu='settings')"></div>
        <div class="row mt-3 justify-content-center">
            <div class="col-2">
                <div th:replace="fragments.html :: study-settings-menu(currentMenu='description')"></div>
            </div>
            <div class="col-8">
                <div th:replace="fragments.html :: message"></div>
                <form class="needs-validation" th:action="@{'/study/' + ${study.getPath()} + '/settings/description'}"
                      th:object="${studyDescriptionForm}" method="post" novalidate>
                    <div class="form-group">
                        <label for="shortDescription">짧은 소개</label>
                        <textarea id="shortDescription" type="textarea" th:field="*{shortDescription}" class="form-control"
                                  placeholder="스터디를 짧게 소개해 주세요." aria-describedby="shortDescriptionHelp" required maxlength="100">
                            </textarea>
                        <small id="shortDescriptionHelp" class="form-text text-muted">
                            100자 이내로 스터디를 짧은 소개해 주세요.
                        </small>
                        <small class="invalid-feedback">짧은 소개를 입력하세요.</small>
                        <small class="form-text text-danger" th:if="${#fields.hasErrors('shortDescription')}" th:errors="*{shortDescription}">ShortDescription Error</small>
                    </div>

                    <div class="form-group">
                        <label for="fullDescription">상세 소개</label>
                        <textarea id="fullDescription" type="textarea" th:field="*{fullDescription}" class="form-control"
                                  placeholder="스터디를 자세히 설명해 주세요." aria-describedby="fullDescriptionHelp" required></textarea>
                        <small id="fullDescriptionHelp" class="form-text text-muted">
                            스터디의 목표, 일정, 진행 방식, 사용할 교재 또는 인터넷 강좌 그리고 모집중인 스터디원 등 스터디에 대해 자세히 적어 주세요.
                        </small>
                        <small class="invalid-feedback">상세 소개를 입력하세요.</small>
                        <small class="form-text text-danger" th:if="${#fields.hasErrors('fullDescription')}" th:errors="*{fullDescription}">FullDescription Error</small>
                    </div>

                    <div class="form-group">
                        <button class="btn btn-primary btn-block" type="submit"
                                aria-describedby="submitHelp">수정하기</button>
                    </div>
                </form>
            </div>
        </div>
        <div th:replace="fragments.html :: footer"></div>
    </div>
    <script th:replace="fragments.html :: form-validation"></script>
    <script th:replace="fragments.html :: editor-script"></script>
</body>
</html> 
```
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/study_5.jpg"></p>

<br>

## 스터디 설정 - 배너
- 이미지 파일 업로드 시 고려할 점
    * 이미지 파일인지 확인 (이미지가 아닌 파일을 업로드 하려는건 아닌지 확인)
    * 이미지 크기 확인 (너무 큰 이미지 업로드 하지 않도록)
<br>

### 구현
- 이미지 파일 확인 및 크기 관련 설정
```html
<!-- framents.html-->
#study-logo {
    height: 200px;
    width: 100%;
    overflow: hidden;
    padding: 0;
    margin: 0;
}

#study-logo img {
    height: auto;
    width: 100%;
    overflow: hidden;
}

<script th:fragment="tooltip" type="application/javascript">
    $(function () {
        $('[data-toggle="tooltip"]').tooltip()
    })
</script>
```
- 톰캣 기본 요청 사이즈 수정
```properties
# 톰캣 기본 요청 사이즈는 2MB 입니다. 그것보다 큰 요청을 받고 싶은 경우에 이 값을 조정해야 합니다.
server.tomcat.max-http-form-post-size=5MB
```
- 스터디 배너 관련 맵핑
```java
@Controller
@RequestMapping("/study/{path}/settings")
@RequiredArgsConstructor
public class StudySettingsController {

    ...

    @GetMapping("/banner")
    public String studyImageForm(@CurrentAccount Account account, @PathVariable String path, Model model) {
        Study study = studyService.getStudyToUpdate(account, path);
        model.addAttribute(account);
        model.addAttribute(study);
        return "study/settings/banner";
    }

    @PostMapping("/banner")
    public String studyImageSubmit(@CurrentAccount Account account, @PathVariable String path,
                                   String image, RedirectAttributes attributes) {
        Study study = studyService.getStudyToUpdate(account, path);
        studyService.updateStudyImage(study, image);
        attributes.addFlashAttribute("message", "스터디 이미지를 수정했습니다.");
        return "redirect:/study/" + getPath(path) + "/settings/banner";
    }

    @PostMapping("/banner/enable")
    public String enableStudyBanner(@CurrentAccount Account account, @PathVariable String path) {
        Study study = studyService.getStudyToUpdate(account, path);
        studyService.enableStudyBanner(study);
        return "redirect:/study/" + getPath(path) + "/settings/banner";
    }

    @PostMapping("/banner/disable")
    public String disableStudyBanner(@CurrentAccount Account account, @PathVariable String path) {
        Study study = studyService.getStudyToUpdate(account, path);
        studyService.disableStudyBanner(study);
        return "redirect:/study/" + getPath(path) + "/settings/banner";
    }
}
```
- 만약 배너를 사용하지만 배너 이미지가 없는 경우 기본 이미지 사용
```java
...
public class Study {

    ...

    public void addMember(Account account) {
        this.members.add(account);
    }

    public String getImage() {
        return image != null ? image : "/images/default_banner.jpg";
    }
}
```
- 배너 이미지 설정 뷰 생성
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments.html :: head"></head>
<body>
    <nav th:replace="fragments.html :: main-nav"></nav>
    <div th:replace="fragments.html :: study-banner"></div>
    <div class="container">
        <div th:replace="fragments.html :: study-info"></div>
        <div th:replace="fragments.html :: study-menu(studyMenu='settings')"></div>
        <div class="row mt-3 justify-content-center">
            <div class="col-2">
                <div th:replace="fragments.html :: study-settings-menu(currentMenu='image')"></div>
            </div>
            <div class="col-8">
                <div th:replace="fragments.html :: message"></div>
                <div class="row">
                    <h2 class="col-sm-12">배너 이미지 사용</h2>
                </div>
                <form th:if="${!study.useBanner}" action="#" th:action="@{'/study/' + ${study.getPath()} + '/settings/banner/enable'}" method="post" novalidate>
                    <div class="alert alert-primary" role="alert">
                        스터디 메뉴에서 스터디 배너 이미지를 사용합니다. 스터디 배너 이미지를 아직 설정하지 않았다면, 기본 배너 이미지를 사용합니다.
                    </div>
                    <div class="form-group">
                        <button class="btn btn-outline-primary btn-block" type="submit" aria-describedby="submitHelp">배너 이미지 사용하기</button>
                    </div>
                </form>
                <form th:if="${study.useBanner}" action="#" th:action="@{'/study/' + ${study.getPath()} + '/settings/banner/disable'}" method="post" novalidate>
                    <div class="alert alert-info" role="alert">
                        스터디 메뉴에서 스터디 배너 이미지를 사용하지 않습니다. 스터디 목록에서는 배너 이미지를 사용합니다.
                    </div>
                    <div class="form-group">
                        <button class="btn btn-outline-primary btn-block" type="submit" aria-describedby="submitHelp">배너 이미지 사용하지 않기</button>
                    </div>
                </form>
                <hr/>
                <div class="row">
                    <h2 class="col-sm-12">배너 이미지 변경</h2>
                </div>
                <form id="imageForm" action="#" th:action="@{'/study/' + ${study.getPath()} + '/settings/banner'}" method="post" novalidate>
                    <div class="form-group">
                        <input id="studyImage" type="hidden" name="image" class="form-control" />
                    </div>
                </form>
                <div class="card text-center">
                    <div id="current-study-image" class="mt-3">
                        <img class="rounded" th:src="${study.image}" width="640" alt="name" th:alt="${study.title}"/>
                    </div>
                    <div id="new-study-image" class="mt-3"></div>
                    <div class="card-body">
                        <div class="custom-file">
                            <input type="file" class="custom-file-input" id="study-image-file">
                            <label class="custom-file-label" for="study-image-file">스터디 이미지 변경</label>
                        </div>
                        <div id="new-study-image-control" class="mt-3">
                            <button class="btn btn-outline-primary btn-block" id="cut-button">자르기</button>
                            <button class="btn btn-outline-success btn-block" id="confirm-button">확인</button>
                            <button class="btn btn-primary btn-block" id="save-button">저장</button>
                            <button class="btn btn-outline-warning btn-block" id="reset-button">취소</button>
                        </div>
                        <div id="cropped-new-study-image" class="mt-3"></div>
                    </div>
                </div>
            </div>
        </div>
        <div th:replace="fragments.html :: footer"></div>
    </div>
    <script th:replace="fragments.html :: tooltip"></script>
    <link  href="/node_modules/cropper/dist/cropper.min.css" rel="stylesheet">
    <script src="/node_modules/cropper/dist/cropper.min.js"></script>
    <script src="/node_modules/jquery-cropper/dist/jquery-cropper.min.js"></script>
    <script type="application/javascript">
        $(function() {
            cropper = '';
            let $confirmBtn = $("#confirm-button");
            let $resetBtn = $("#reset-button");
            let $cutBtn = $("#cut-button");
            let $saveBtn = $("#save-button");
            let $newStudyImage = $("#new-study-image");
            let $currentStudyImage = $("#current-study-image");
            let $resultImage = $("#cropped-new-study-image");
            let $studyImage = $("#studyImage");

            $newStudyImage.hide();
            $cutBtn.hide();
            $resetBtn.hide();
            $confirmBtn.hide();
            $saveBtn.hide();

            $("#study-image-file").change(function(e) {
                if (e.target.files.length === 1) {
                    const reader = new FileReader();
                    reader.onload = e => {
                        if (e.target.result) {
                            if (!e.target.result.startsWith("data:image")) {
                                alert("이미지 파일을 선택하세요.");
                                return;
                            }

                            let img = document.createElement("img");
                            img.id = 'new-study';
                            img.src = e.target.result;
                            img.setAttribute('width', '100%');

                            $newStudyImage.html(img);
                            $newStudyImage.show();
                            $currentStudyImage.hide();

                            let $newImage = $(img);
                            $newImage.cropper({aspectRatio: 13/2});
                            cropper = $newImage.data('cropper');

                            $cutBtn.show();
                            $confirmBtn.hide();
                            $resetBtn.show();
                        }
                    };

                    reader.readAsDataURL(e.target.files[0]);
                }
            });

            $resetBtn.click(function() {
                $currentStudyImage.show();
                $newStudyImage.hide();
                $resultImage.hide();
                $resetBtn.hide();
                $cutBtn.hide();
                $confirmBtn.hide();
                $saveBtn.hide();
                $studyImage.val('');
            });

            $cutBtn.click(function () {
                let dataUrl = cropper.getCroppedCanvas().toDataURL();

                if (dataUrl.length > 1000 * 1024) {
                    alert("이미지 파일이 너무 큽니다. 1024000 보다 작은 파일을 사용하세요. 현재 이미지 사이즈 " + dataUrl.length);
                    return;
                }

                let newImage = document.createElement("img");
                newImage.id = "cropped-new-study-image";
                newImage.src = dataUrl;
                newImage.width = 640;
                $resultImage.html(newImage);
                $resultImage.show();
                $confirmBtn.show();

                $confirmBtn.click(function () {
                    $newStudyImage.html(newImage);
                    $cutBtn.hide();
                    $confirmBtn.hide();
                    $studyImage.val(dataUrl);
                    $saveBtn.show();
                });
            });

            $saveBtn.click(function() {
                $("#imageForm").submit();
            })
        });
    </script>
</body>
</html> 
```
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/study_6.jpg"></p>

<br>

## 스터디 설정 - 태그 / 지역
- 데이터를 필요한 만큼만 읽어오기.
    * 태그와 지역 정보를 Ajax로 수정할 때 스터디 (+멤버, +매니저, +태그, +지역) 정보를 전부 가져올 필요가 있는가?
    * 스프링 데이터 JPA 메소드 작명, @EntityGraph와 @NamedEntityGraph 활용하기
<br>

### 구현
- 태그, 지역 관련 맵핑 추가
```java
@Controller
@RequestMapping("/study/{path}/settings")
@RequiredArgsConstructor
public class StudySettingsController {

    ...
    private final TagService tagService;
    private final TagRepository tagRepository;
    private final ZoneRepository zoneRepository;
    private final ObjectMapper objectMapper;

    ...

    @GetMapping("/tags")
    public String studyTagsForm(@CurrentAccount Account account, @PathVariable String path, Model model)
            throws JsonProcessingException {
        Study study = studyService.getStudyToUpdate(account, path);
        model.addAttribute(account);
        model.addAttribute(study);

        model.addAttribute("tags", study.getTags().stream()
                .map(Tag::getTitle).collect(Collectors.toList()));
        List<String> allTagTitles = tagRepository.findAll().stream()
                .map(Tag::getTitle).collect(Collectors.toList());
        model.addAttribute("whitelist", objectMapper.writeValueAsString(allTagTitles));
        return "study/settings/tags";
    }

    @PostMapping("/tags/add")
    @ResponseBody
    public ResponseEntity addTag(@CurrentAccount Account account, @PathVariable String path,
                                 @RequestBody TagForm tagForm) {
        Study study = studyService.getStudyToUpdateTag(account, path);
        Tag tag = tagService.findOrCreateNew(tagForm.getTagTitle());
        studyService.addTag(study, tag);
        return ResponseEntity.ok().build();
    }

    @PostMapping("/tags/remove")
    @ResponseBody
    public ResponseEntity removeTag(@CurrentAccount Account account, @PathVariable String path,
                                    @RequestBody TagForm tagForm) {
        Study study = studyService.getStudyToUpdateTag(account, path);
        Tag tag = tagRepository.findByTitle(tagForm.getTagTitle());
        if (tag == null) {
            return ResponseEntity.badRequest().build();
        }

        studyService.removeTag(study, tag);
        return ResponseEntity.ok().build();
    }

    @GetMapping("/zones")
    public String studyZonesForm(@CurrentAccount Account account, @PathVariable String path, Model model)
            throws JsonProcessingException {
        Study study = studyService.getStudyToUpdate(account, path);
        model.addAttribute(account);
        model.addAttribute(study);
        model.addAttribute("zones", study.getZones().stream()
                .map(Zone::toString).collect(Collectors.toList()));
        List<String> allZones = zoneRepository.findAll().stream().map(Zone::toString).collect(Collectors.toList());
        model.addAttribute("whitelist", objectMapper.writeValueAsString(allZones));
        return "study/settings/zones";
    }

    @PostMapping("/zones/add")
    @ResponseBody
    public ResponseEntity addZone(@CurrentAccount Account account, @PathVariable String path,
                                  @RequestBody ZoneForm zoneForm) {
        Study study = studyService.getStudyToUpdateZone(account, path);
        Zone zone = zoneRepository.findByCityAndProvince(zoneForm.getCityName(), zoneForm.getProvinceName());
        if (zone == null) {
            return ResponseEntity.badRequest().build();
        }

        studyService.addZone(study, zone);
        return ResponseEntity.ok().build();
    }

    @PostMapping("/zones/remove")
    @ResponseBody
    public ResponseEntity removeZone(@CurrentAccount Account account, @PathVariable String path,
                                     @RequestBody ZoneForm zoneForm) {
        Study study = studyService.getStudyToUpdateZone(account, path);
        Zone zone = zoneRepository.findByCityAndProvince(zoneForm.getCityName(), zoneForm.getProvinceName());
        if (zone == null) {
            return ResponseEntity.badRequest().build();
        }

        studyService.removeZone(study, zone);
        return ResponseEntity.ok().build();
    }
}
```
- TagService 생성
    * `findOrCreateNew()`
        - DB의 태그 확인 메서드
        - 참고) SettingsController에 tag관련 코드에도 중복 코드 대신 `findOrCreateNew()` 메서드 사용
```java
@Service
@Transactional
@RequiredArgsConstructor
public class TagService {

    private final TagRepository tagRepository;

    public Tag findOrCreateNew(String tagTitle) {
        Tag tag = tagRepository.findByTitle(tagTitle);
        if (tag == null) {
            tag = tagRepository.save(Tag.builder().title(tagTitle).build());
        }
        return tag;
    }
}
```
- StudyService에 태그/지역 관련 메서드 추가
    * `getStudyToUpdateTag()`
        - 스터디를 조회할 경우 모든 데이터를 가져와야 하지만 태그/지역 정보의 추가, 삭제는 매니저 여부, 태그 정보만 있으면 된다.
    * account의 태그를 변경할 때 accountRepository를 통해서 가져오기 때문에 persist상태였지만 지금 study의 상태는 이미 repository에서 가져오기 때문에 persist 상태이다.
        - 따라서 StudyService에서 repository 통해서 조회할 필요가 없다.
```java
@Service
@Transactional
@RequiredArgsConstructor
public class StudyService {

    ...

    public void addTag(Study study, Tag tag) {
        study.getTags().add(tag);
    }

    public void removeTag(Study study, Tag tag) {
        study.getTags().remove(tag);
    }

    public void addZone(Study study, Zone zone) {
        study.getZones().add(zone);
    }

    public void removeZone(Study study, Zone zone) {
        study.getZones().remove(zone);
    }

    public Study getStudyToUpdateTag(Account account, String path) {
        Study study = repository.findStudyWithTagsByPath(path);
        checkIfExistingStudy(path, study);
        checkIfManager(account, study);
        return study;
    }

    public Study getStudyToUpdateZone(Account account, String path) {
        Study study = repository.findStudyWithZonesByPath(path);
        checkIfExistingStudy(path, study);
        checkIfManager(account, study);
        return study;
    }

    private void checkIfManager(Account account, Study study) {
        if (!account.isManagerOf(study)) {
            throw new AccessDeniedException("해당 기능을 사용할 수 없습니다.");
        }
    }

    private void checkIfExistingStudy(String path, Study study) {
        if (study == null) {
            throw new IllegalArgumentException(path + "에 해당하는 스터디가 없습니다.");
        }
    }
}
```
- StudyRepository에 EntityGraph 관련 코드 추가
    * 사실 WithTags와 WithZones는 스프링 데이터 JPA가 처리하는 키워드가 아니기 때문에 findByPath와 같은 역할을 한다.
    * 같은 쿼리가 발생하지만 다른 엔티티 그래프를 사용하기 위해 이름을 다르게 하는 것이다.
```java
@Transactional(readOnly = true)
public interface StudyRepository extends JpaRepository<Study, Long> {

    boolean existsByPath(String path);

    @EntityGraph(value = "Study.withAll", type = EntityGraph.EntityGraphType.LOAD)
    Study findByPath(String path);

    @EntityGraph(value = "Study.withTagsAndManagers", type = EntityGraph.EntityGraphType.FETCH)
    Study findStudyWithTagsByPath(String path);

    @EntityGraph(value = "Study.withZonesAndManagers", type = EntityGraph.EntityGraphType.FETCH)
    Study findStudyWithZonesByPath(String path);
}
```
- Study 엔티티에 EntityGraph 관련 코드 추가
```java
...
@NamedEntityGraph(name = "Study.withTagsAndManagers", attributeNodes = {
        @NamedAttributeNode("tags"),
        @NamedAttributeNode("managers")})
@NamedEntityGraph(name = "Study.withZonesAndManagers", attributeNodes = {
        @NamedAttributeNode("zones"),
        @NamedAttributeNode("managers")})
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
@Builder @AllArgsConstructor @NoArgsConstructor
public class Study {
    ...
}
```
- fragments.html에 태그/지역 관련 뷰 추가
```html
<!-- fragments.html-->
<script src="/node_modules/@yaireo/tagify/dist/tagify.min.js"></script>
<script type="application/javascript" th:inline="javascript">
    $(function() {
        var studyPath = "[(${study.path})]";
        function tagRequest(url, tagTitle) {
            $.ajax({
                dataType: "json",
                autocomplete: {
                    enabled: true,
                    rightKey: true,
                },
                contentType: "application/json; charset=utf-8",
                method: "POST",
                url: "/study/" + studyPath + "/settings/tags" + url,
                data: JSON.stringify({'tagTitle': tagTitle})
            }).done(function (data, status) {
                console.log("${data} and status is ${status}");
            });
        }

        function onAdd(e) {
            tagRequest("/add", e.detail.data.value);
        }

        function onRemove(e) {
            tagRequest("/remove", e.detail.data.value);
        }

        var tagInput = document.querySelector("#tags");
        var tagify = new Tagify(tagInput, {
            pattern: /^.{0,20}$/,
            whitelist: JSON.parse(document.querySelector("#whitelist").textContent),
            dropdown : {
                enabled: 1, // suggest tags after a single character input
            } // map tags
        });
        tagify.on("add", onAdd);
        tagify.on("remove", onRemove);
        // add a class to Tagify's input element
        tagify.DOM.input.classList.add('form-control');
        // re-place Tagify's input element outside of the  element (tagify.DOM.scope), just before it
        tagify.DOM.scope.parentNode.insertBefore(tagify.DOM.input, tagify.DOM.scope);
    });
</script>

<div th:fragment="update-tags (baseUrl)">
    <script src="/node_modules/@yaireo/tagify/dist/tagify.min.js"></script>
    <script type="application/javascript" th:inline="javascript">
        $(function() {
            function tagRequest(url, tagTitle) {
                $.ajax({
                    dataType: "json",
                    autocomplete: {
                        enabled: true,
                        rightKey: true,
                    },
                    contentType: "application/json; charset=utf-8",
                    method: "POST",
                    url: "[(${baseUrl})]" + url,
                    data: JSON.stringify({'tagTitle': tagTitle})
                }).done(function (data, status) {
                    console.log("${data} and status is ${status}");
                });
            }

            function onAdd(e) {
                tagRequest("/add", e.detail.data.value);
            }

            function onRemove(e) {
                tagRequest("/remove", e.detail.data.value);
            }

            var tagInput = document.querySelector("#tags");
            var tagify = new Tagify(tagInput, {
                pattern: /^.{0,20}$/,
                whitelist: JSON.parse(document.querySelector("#whitelist").textContent),
                dropdown : {
                    enabled: 1, // suggest tags after a single character input
                } // map tags
            });
            tagify.on("add", onAdd);
            tagify.on("remove", onRemove);
            // add a class to Tagify's input element
            tagify.DOM.input.classList.add('form-control');
            // re-place Tagify's input element outside of the  element (tagify.DOM.scope), just before it
            tagify.DOM.scope.parentNode.insertBefore(tagify.DOM.input, tagify.DOM.scope);
        });
    </script>
</div>

<div th:fragment="update-zones (baseUrl)">
    <script src="/node_modules/@yaireo/tagify/dist/tagify.min.js"></script>
    <script type="application/javascript">
        $(function () {
            function tagRequest(url, zoneName) {
                $.ajax({
                    dataType: "json",
                    autocomplete: {
                        enabled: true,
                        rightKey: true,
                    },
                    contentType: "application/json; charset=utf-8",
                    method: "POST",
                    url: "[(${baseUrl})]" + url,
                    data: JSON.stringify({'zoneName': zoneName})
                }).done(function (data, status) {
                    console.log("${data} and status is ${status}");
                });
            }

            function onAdd(e) {
                tagRequest("/add", e.detail.data.value);
            }

            function onRemove(e) {
                tagRequest("/remove", e.detail.data.value);
            }

            var tagInput = document.querySelector("#zones");

            var tagify = new Tagify(tagInput, {
                enforceWhitelist: true,
                whitelist: JSON.parse(document.querySelector("#whitelist").textContent),
                dropdown : {
                    enabled: 1, // suggest tags after a single character input
                } // map tags
            });

            tagify.on("add", onAdd);
            tagify.on("remove", onRemove);

            // add a class to Tagify's input element
            tagify.DOM.input.classList.add('form-control');
            // re-place Tagify's input element outside of the  element (tagify.DOM.scope), just before it
            tagify.DOM.scope.parentNode.insertBefore(tagify.DOM.input, tagify.DOM.scope);
        });
    </script>
</div>
```
- 설정의 태그/지역에 중복 코드 제거
```html
<!-- settings/tags.html-->
<script th:replace="fragments.html :: update-tags(baseUrl='/settings/tags')"></script>
```
```html
<!-- settings/zones.html-->
<script th:replace="fragments.html :: update-zones(baseUrl='/settings/zones')"></script>
```
- 스터디의 태그/지역 관련 뷰 추가
```html
<!-- tags.html-->
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments.html :: head"></head>
<body>
    <nav th:replace="fragments.html :: main-nav"></nav>
    <div th:replace="fragments.html :: study-banner"></div>
    <div class="container">
        <div th:replace="fragments.html :: study-info"></div>
        <div th:replace="fragments.html :: study-menu(studyMenu='settings')"></div>
        <div class="row mt-3 justify-content-center">
            <div class="col-2">
                <div th:replace="fragments.html :: study-settings-menu(currentMenu='tags')"></div>
            </div>
            <div class="col-8">
                <div class="row">
                    <h2 class="col-sm-12">스터디 주제</h2>
                </div>
                <div class="row">
                    <div class="col-sm-12">
                        <div class="alert alert-info" role="alert">
                            스터디에서 주로 다루는 주제를 태그로 등록하세요. 태그를 입력하고 콤마(,) 또는 엔터를 입력하세요.
                        </div>
                        <div id="whitelist" th:text="${whitelist}" hidden>
                        </div>
                        <input id="tags" type="text" name="tags" th:value="${#strings.listJoin(tags, ',')}"
                               class="tagify-outside" aria-describedby="tagHelp">
                    </div>
                </div>
            </div>
        </div>
        <div th:replace="fragments.html :: footer"></div>
    </div>
    <script th:replace="fragments.html :: ajax-csrf-header"></script>
    <script th:replace="fragments.html :: update-tags(baseUrl='/study/' + ${study.path} + '/settings/tags')"></script>
</body>
</html>
```
```html
<!-- zones.html-->
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments.html :: head"></head>
<body>
    <nav th:replace="fragments.html :: main-nav"></nav>
    <div th:replace="fragments.html :: study-banner"></div>
    <div class="container">
        <div th:replace="fragments.html :: study-info"></div>
        <div th:replace="fragments.html :: study-menu(studyMenu='settings')"></div>
        <div class="row mt-3 justify-content-center">
            <div class="col-2">
                <div th:replace="fragments.html :: study-settings-menu(currentMenu='zones')"></div>
            </div>
            <div class="col-8">
                <div class="row">
                    <h2 class="col-sm-12">주요 활동 지역</h2>
                </div>
                <div class="row">
                    <div class="col-sm-12">
                        <div class="alert alert-info" role="alert">
                            주로 스터디를 하는 지역을 등록하세요.<br/>
                            시스템에 등록된 지역만 선택할 수 있습니다.
                        </div>
                        <div id="whitelist" th:text="${whitelist}" hidden></div>
                        <input id="zones" type="text" name="zones" th:value="${#strings.listJoin(zones, ',')}"
                               class="tagify-outside">
                    </div>
                </div>
            </div>
        </div>
        <div th:replace="fragments.html :: footer"></div>
    </div>
    <script th:replace="fragments.html :: ajax-csrf-header"></script>
    <script th:replace="fragments.html :: update-zones(baseUrl='/study/' + ${study.path} + '/settings/zones')"></script>
</body>
</html>
```
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/study_7.jpg"></p>

<br>

## 스터디 설정 - 상태 변경
- 스터디 공개 및 종료
    * 드래프트 상태에서 공개 상태로 전환 가능.
    * 공개 상태에서 종료 상태로 전환 가능.
    * 종료 상태에서는 상태 변경 불가.
- 스터디 공개 상태
    * 팀원 모집 시작 및 중단 가능
    * 다른 사용자에게 스터디 정보가 공개 됩니다. (조회 가능)
    * 해당 스터디의 주제와 지역에 대응하는 사용자에게 알림을 전달 한다.
    * 모임을 만들 수 있다.
- 스터디 종료 상태
    * 조회만 가능.
    * 모임을 만들거나, 팀원을 모집할 수 없다.
<br>

### 구현
- StudySettingsController에 상태 관련 맵핑 추가
```java
@Controller
@RequestMapping("/study/{path}/settings")
@RequiredArgsConstructor
public class StudySettingsController {

    ...

    @GetMapping("/study")
    public String studySettingForm(@CurrentAccount Account account, @PathVariable String path, Model model) {
        Study study = studyService.getStudyToUpdate(account, path);
        model.addAttribute(account);
        model.addAttribute(study);
        return "study/settings/study";
    }

    @PostMapping("/study/publish")
    public String publishStudy(@CurrentAccount Account account, @PathVariable String path,
                               RedirectAttributes attributes) {
        Study study = studyService.getStudyToUpdateStatus(account, path);
        studyService.publish(study);
        attributes.addFlashAttribute("message", "스터디를 공개했습니다.");
        return "redirect:/study/" + getPath(path) + "/settings/study";
    }

    @PostMapping("/study/close")
    public String closeStudy(@CurrentAccount Account account, @PathVariable String path,
                             RedirectAttributes attributes) {
        Study study = studyService.getStudyToUpdateStatus(account, path);
        studyService.close(study);
        attributes.addFlashAttribute("message", "스터디를 종료했습니다.");
        return "redirect:/study/" + getPath(path) + "/settings/study";
    }

    @PostMapping("/recruit/start")
    public String startRecruit(@CurrentAccount Account account, @PathVariable String path, Model model,
                               RedirectAttributes attributes) {
        Study study = studyService.getStudyToUpdateStatus(account, path);
        if (!study.canUpdateRecruiting()) {
            attributes.addFlashAttribute("message", "1시간 안에 인원 모집 설정을 여러번 변경할 수 없습니다.");
            return "redirect:/study/" + getPath(path) + "/settings/study";
        }

        studyService.startRecruit(study);
        attributes.addFlashAttribute("message", "인원 모집을 시작합니다.");
        return "redirect:/study/" + getPath(path) + "/settings/study";
    }

    @PostMapping("/recruit/stop")
    public String stopRecruit(@CurrentAccount Account account, @PathVariable String path, Model model,
                              RedirectAttributes attributes) {
        Study study = studyService.getStudyToUpdate(account, path);
        if (!study.canUpdateRecruiting()) {
            attributes.addFlashAttribute("message", "1시간 안에 인원 모집 설정을 여러번 변경할 수 없습니다.");
            return "redirect:/study/" + getPath(path) + "/settings/study";
        }

        studyService.stopRecruit(study);
        attributes.addFlashAttribute("message", "인원 모집을 종료합니다.");
        return "redirect:/study/" + getPath(path) + "/settings/study";
    }
}
```
- StudyService에 상태 관련 메서드 추가
    * 스터디의 상태를 변경하는 것은 매니저인지만 확인하면 된다.
```java

@Service
@Transactional
@RequiredArgsConstructor
public class StudyService {

    ...

    public Study getStudyToUpdateStatus(Account account, String path) {
        Study study = repository.findStudyWithManagersByPath(path);
        checkIfExistingStudy(path, study);
        checkIfManager(account, study);
        return study;
    }

    ...

    public void publish(Study study) {
        study.publish();
    }

    public void close(Study study) {
        study.close();
    }

    public void startRecruit(Study study) {
        study.startRecruit();
    }

    public void stopRecruit(Study study) {
        study.stopRecruit();
    }
}
```
- StudyRepository
```java
@Transactional(readOnly = true)
public interface StudyRepository extends JpaRepository<Study, Long> {

    ...

    @EntityGraph(value = "Study.withManagers", type = EntityGraph.EntityGraphType.FETCH)
    Study findStudyWithManagersByPath(String path);
}
```
- Study 엔티티
    * 악의적인 사용자가 잘못된 요청을 보낼 경우를 대비해 에러를 던져서 예외처리
```java
...
@NamedEntityGraph(name = "Study.withManagers", attributeNodes = {
        @NamedAttributeNode("managers")})
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
@Builder @AllArgsConstructor @NoArgsConstructor
public class Study {

    ...

    public void publish() {
        if (!this.closed && !this.published) {
            this.published = true;
            this.publishedDateTime = LocalDateTime.now();
        } else {
            throw new RuntimeException("스터디를 공개할 수 없는 상태입니다. 스터디를 이미 공개했거나 종료했습니다.");
        }
    }

    public void close() {
        if (this.published && !this.closed) {
            this.closed = true;
            this.closedDateTime = LocalDateTime.now();
        } else {
            throw new RuntimeException("스터디를 종료할 수 없습니다. 스터디를 공개하지 않았거나 이미 종료한 스터디입니다.");
        }
    }

    public void startRecruit() {
        if (canUpdateRecruiting()) {
            this.recruiting = true;
            this.recruitingUpdatedDateTime = LocalDateTime.now();
        } else {
            throw new RuntimeException("인원 모집을 시작할 수 없습니다. 스터디를 공개하거나 한 시간 뒤 다시 시도하세요.");
        }
    }

    public void stopRecruit() {
        if (canUpdateRecruiting()) {
            this.recruiting = false;
            this.recruitingUpdatedDateTime = LocalDateTime.now();
        } else {
            throw new RuntimeException("인원 모집을 멈출 수 없습니다. 스터디를 공개하거나 한 시간 뒤 다시 시도하세요.");
        }
    }

    public boolean canUpdateRecruiting() {
        return this.published && this.recruitingUpdatedDateTime == null || this.recruitingUpdatedDateTime.isBefore(LocalDateTime.now().minusHours(1));
    }
}
```
- 스터디 설정 관련 뷰 생성
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments.html :: head"></head>
<body>
    <nav th:replace="fragments.html :: main-nav"></nav>
    <div th:replace="fragments.html :: study-banner"></div>
    <div class="container">
        <div th:replace="fragments.html :: study-info"></div>
        <div th:replace="fragments.html :: study-menu(studyMenu='settings')"></div>
        <div class="row mt-3 justify-content-center">
            <div class="col-2">
                <div th:replace="fragments.html :: study-settings-menu(currentMenu='study')"></div>
            </div>
            <div class="col-8">
                <div th:replace="fragments.html :: message"></div>
                <div class="row">
                    <h5 class="col-sm-12">스터디 공개 및 종료</h5>
                    <form th:if="${!study.published && !study.closed}" class="col-sm-12" action="#" th:action="@{'/study/' + ${study.getPath()} + '/settings/study/publish'}" method="post" novalidate>
                        <div class="alert alert-info" role="alert">
                            스터디를 다른 사용자에게 공개할 준비가 되었다면 버튼을 클릭하세요.<br/>
                            소개, 배너 이미지, 스터디 주제 및 활동 지역을 등록했는지 확인하세요.<br/>
                            스터디를 공개하면 주요 활동 지역과 스터디 주제에 관심있는 다른 사용자에게 알림을 전송합니다.
                        </div>
                        <div class="form-group">
                            <button class="btn btn-outline-primary" type="submit" aria-describedby="submitHelp">스터디 공개</button>
                        </div>
                    </form>
                    <form th:if="${study.published && !study.closed}" class="col-sm-12" action="#" th:action="@{'/study/' + ${study.getPath()} + '/settings/study/close'}" method="post" novalidate>
                        <div class="alert alert-warning" role="alert">
                            스터디 활동을 마쳤다면 스터디를 종료하세요.<br/>
                            스터디를 종료하면 더이상 팀원을 모집하거나 모임을 만들 수 없으며, 스터디 경로와 이름을 수정할 수 없습니다.<br/>
                            스터디 모임과 참여한 팀원의 기록은 그대로 보관합니다.
                        </div>
                        <div class="form-group">
                            <button class="btn btn-outline-warning" type="submit" aria-describedby="submitHelp">스터디 종료</button>
                        </div>
                    </form>
                    <div th:if="${study.closed}" class="col-sm-12 alert alert-info">
                        이 스터디는 <span class="date-time" th:text="${study.closedDateTime}"></span>에 종료됐습니다.<br/>
                        다시 스터디를 진행하고 싶다면 새로운 스터디를 만드세요.<br/>
                    </div>
                </div>

                <hr th:if="${!study.closed && !study.recruiting && study.published}"/>
                <div class="row" th:if="${!study.closed && !study.recruiting && study.published}">
                    <h5 class="col-sm-12">팀원 모집</h5>
                    <form class="col-sm-12" action="#" th:action="@{'/study/' + ${study.getPath()} + '/settings/recruit/start'}" method="post" novalidate>
                        <div class="alert alert-info" role="alert">
                            팀원을 모집합니다.<br/>
                            충분한 스터디 팀원을 모집했다면 모집을 멈출 수 있습니다.<br/>
                            팀원 모집 정보는 1시간에 한번만 바꿀 수 있습니다.
                        </div>
                        <div class="form-group">
                            <button class="btn btn-outline-primary" type="submit" aria-describedby="submitHelp">팀원 모집 시작</button>
                        </div>
                    </form>
                </div>

                <hr th:if="${!study.closed && study.recruiting && study.published}"/>
                <div class="row" th:if="${!study.closed && study.recruiting && study.published}">
                    <h5 class="col-sm-12">팀원 모집</h5>
                    <form class="col-sm-12" action="#" th:action="@{'/study/' + ${study.getPath()} + '/settings/recruit/stop'}" method="post" novalidate>
                        <div class="alert alert-primary" role="alert">
                            팀원 모집을 중답합니다.<br/>
                            팀원 충원이 필요할 때 다시 팀원 모집을 시작할 수 있습니다.<br/>
                            팀원 모집 정보는 1시간에 한번만 바꿀 수 있습니다.
                        </div>
                        <div class="form-group">
                            <button class="btn btn-outline-primary" type="submit" aria-describedby="submitHelp">팀원 모집 중단</button>
                        </div>
                    </form>
                </div>

                <hr th:if="${!study.closed}"/>
                <div class="row" th:if="${!study.closed}">
                    <h5 class="col-sm-12">스터디 경로</h5>
                    <form class="col-sm-12 needs-validation" action="#" th:action="@{'/study/' + ${study.path} + '/settings/study/path'}" method="post" novalidate>
                        <div class="alert alert-warning" role="alert">
                            스터디 경로를 수정하면 이전에 사용하던 경로로 스터디에 접근할 수 없으니 주의하세요. <br/>
                        </div>
                        <div class="form-group">
                            <input id="path" type="text" name="newPath" th:value="${newPath}" class="form-control"
                                   placeholder="예) study-path" aria-describedby="pathHelp" required>
                            <small id="pathHelp" class="form-text text-muted">
                                공백없이 문자, 숫자, 대시(-)와 언더바(_)만 3자 이상 20자 이내로 입력하세요. 스터디 홈 주소에 사용합니다. 예) /study/<b>study-path</b>
                            </small>
                            <small class="invalid-feedback">스터디 경로를 입력하세요.</small>
                            <small class="form-text text-danger" th:if="${studyPathError}" th:text="${studyPathError}">Path Error</small>
                        </div>
                        <div class="form-group">
                            <button class="btn btn-outline-warning" type="submit" aria-describedby="submitHelp">경로 수정</button>
                        </div>
                    </form>
                </div>

                <hr th:if="${!study.closed}"/>
                <div class="row" th:if="${!study.closed}">
                    <h5 class="col-12">스터디 이름</h5>
                    <form class="needs-validation col-12" action="#" th:action="@{'/study/' + ${study.path} + '/settings/study/title'}" method="post" novalidate>
                        <div class="alert alert-warning" role="alert">
                            스터디 이름을 수정합니다.<br/>
                        </div>
                        <div class="form-group">
                            <label for="title">스터디 이름</label>
                            <input id="title" type="text" name="newTitle" th:value="${study.title}" class="form-control"
                                   placeholder="스터디 이름" aria-describedby="titleHelp" required maxlength="50">
                            <small id="titleHelp" class="form-text text-muted">
                                스터디 이름을 50자 이내로 입력하세요.
                            </small>
                            <small class="invalid-feedback">스터디 이름을 입력하세요.</small>
                            <small class="form-text text-danger" th:if="${studyTitleError}" th:text="${studyTitleError}">Title Error</small>
                        </div>
                        <div class="form-group">
                            <button class="btn btn-outline-warning" type="submit" aria-describedby="submitHelp">스터디 이름 수정</button>
                        </div>
                    </form>
                </div>

                <hr/>
                <div class="row">
                    <h5 class="col-sm-12 text-danger">스터디 삭제</h5>
                    <form class="col-sm-12" action="#" th:action="@{'/study/' + ${study.getPath()} + '/settings/study/remove'}" method="post" novalidate>
                        <div class="alert alert-danger" role="alert">
                            스터디를 삭제하면 스터디 관련 모든 기록을 삭제하며 복구할 수 없습니다. <br/>
                            <b>다음에 해당하는 스터디는 자동으로 삭제 됩니다.</b>
                            <ul>
                                <li>만든지 1주일이 지난 비공개 스터디</li>
                                <li>스터디 공개 이후, 한달 동안 모임을 만들지 않은 스터디</li>
                                <li>스터디 공개 이후, 모임을 만들지 않고 종료한 스터디</li>
                            </ul>
                        </div>
                        <div class="form-group">
                            <button class="btn btn-outline-danger" type="submit" aria-describedby="submitHelp">스터디 삭제</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
        <div th:replace="fragments.html :: footer"></div>
    </div>
    <script th:replace="fragments.html :: tooltip"></script>
    <script th:replace="fragments.html :: form-validation"></script>
</body>
</html>
```
- 실행하면 다음 화면과 같이 스터디 공개 및 종료, 팀원 모집/중단 기능을 사용할 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/study_8.jpg"></p>

<br>
