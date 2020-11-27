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
