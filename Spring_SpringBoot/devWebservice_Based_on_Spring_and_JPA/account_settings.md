# 계정 설정

## 프로필 수정 폼

- SettingsController 생성
    * bio, link, occupation, location 정보만 입력받아서 Account 정보를 수정한다.
```java
@Controller
public class SettingsController {

    @GetMapping("/settings/profile")
    public String profileUpdateForm(@CurrentUser Account account, Model model) {
        model.addAttribute(account);
        model.addAttribute(new Profile(account));
        return "/settings/profile";
    }
}
```
- 폼을 채울 객체 생성
```java
@Data
public class Profile {

    @Length(max = 35)
    private String bio;

    @Length(max = 50)
    private String url;

    @Length(max = 50)
    private String occupation;

    @Length(max = 50)
    private String location;

    public Profile(Account account) {
        this.bio = account.getBio();
        this.url = account.getUrl();
        this.occupation = account.getOccupation();
        this.location = account.getLocation();
    }
}
```
- fragments.html에 action관련 코드 추가
    * 메뉴를 누를 때마다 서버에 요청하며 현재 탭을 알려준다.
```html
<div th:fragment="settings-menu (currentMenu)" class="list-group">
    <a class="list-group-item list-group-item-action" th:classappend="${currentMenu == 'profile'}? active" href="#" th:href="@{/settings/profile}">프로필</a>
    <a class="list-group-item list-group-item-action" th:classappend="${currentMenu == 'password'}? active" href="#" th:href="@{/settings/password}">패스워드</a>
    <a class="list-group-item list-group-item-action" th:classappend="${currentMenu == 'notifications'}? active" href="#" th:href="@{/settings/notifications}">알림</a>
    <a class="list-group-item list-group-item-action" th:classappend="${currentMenu == 'tags'}? active" href="#" th:href="@{/settings/tags}">관심 주제</a>
    <a class="list-group-item list-group-item-action" th:classappend="${currentMenu == 'zones'}? active" href="#" th:href="@{/settings/zones}">활동 지역</a>
    <a class="list-group-item list-group-item-action list-group-item-danger" th:classappend="${currentMenu == 'account'}? active" href="#" th:href="@{/settings/account}">계정</a>
</div>
```
- 프로필 수정 폼 생성
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments.html :: head"></head>

<body class="bg-light">
<div th:replace="fragments.html :: main-nav"></div>
    <div class="container">
        <div class="row mt-5 justify-content-center">
            <div class="col-2">
                <div th:replace="fragments.html :: settings-menu(currentMenu='profile')"></div>
            </div>
            <div class="col-8">
                <div class="row">
                    <h2 class="col-sm-12" th:text="${account.nickname}">whiteship</h2>
                </div>
                <div class="row mt-3">
                    <form class="col-sm-6" action="#"
                          th:action="@{/settings/profile}" th:object="${profile}" method="post" novalidate>
                        <div class="form-group">
                            <label for="bio">한 줄 소개</label>
                            <input id="bio" type="text" th:field="*{bio}" class="form-control"
                                   placeholder="간략한 소개를 부탁합니다." aria-describedby="bioHelp" required>
                            <small id="bioHelp" class="form-text text-muted">
                                길지 않게 35자 이내로 입력하세요.
                            </small>
                            <small class="form-text text-danger" th:if="${#fields.hasErrors('bio')}" th:errors="*{bio}">
                                조금 길어요.
                            </small>
                        </div>

                        <div class="form-group">
                            <label for="url">링크</label>
                            <input id="url" type="url" th:field="*{url}" class="form-control"
                                   placeholder="http://studyolle.com" aria-describedby="urlHelp" required>
                            <small id="urlHelp" class="form-text text-muted">
                                블로그, 유튜브 또는 포트폴리오나 좋아하는 웹 사이트 등 본인을 표현할 수 있는 링크를 추가하세요.
                            </small>
                            <small class="form-text text-danger" th:if="${#fields.hasErrors('url')}" th:errors="*{url}">
                                올바른 URL이 아닙니다. 예시처럼 입력해 주세요.
                            </small>
                        </div>

                        <div class="form-group">
                            <label for="company">직업</label>
                            <input id="company" type="text" th:field="*{occupation}" class="form-control"
                                   placeholder="어떤 일을 하고 계신가요?" aria-describedby="occupationHelp" required>
                            <small id="occupationHelp" class="form-text text-muted">
                                개발자/매니저/취준생/대표/기타 등등
                            </small>
                        </div>

                        <div class="form-group">
                            <label for="location">활동 지역</label>
                            <input id="location" type="text" th:field="*{location}" class="form-control"
                                   placeholder="서울, 경기, 부산"
                                   aria-describedby="locationdHelp" required>
                            <small id="locationdHelp" class="form-text text-muted">
                                주요 활동 지역의 도시 이름을 알려주세요.
                            </small>
                        </div>

                        <div class="form-group">
                            <button class="btn btn-primary btn-block" type="submit"
                                    aria-describedby="submitHelp">수정하기</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</body>
</html>
```
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/account_settings_1.jpg"></p>

<br>

## 프로필 수정 처리
- 폼 처리는 간단하다.
    * 기존에 있던 값을 삭제하고 싶을 수도 있기 때문에 비어있는 값을 허용한다.
    * 중복된 값을 고민하지 않아도 된다.
    * 확인할 내용은 입력 값의 길이 정도이다.
<br>

### 목표
- 에러가 있는 경우 폼 다시 보여주기.
- 에러가 없는 경우 저장하고, 프로필 수정 페이지 다시 보여주기. (리다이렉트)
- 리다이렉트 시 수정 완료 메시지 전달.
<br>

### 구현
- PostMapping 추가(프로필 수정 요청이 들어왔을 경우)
    * `@Valid Profile profile, Errors errors`
        - profile을 받아올 때 @ModelAttribute가 생략된 것이며 객체의 바인딩 에러를 받아주는 errors가 오른쪽에 항상 짝으로 있어야 한다.
    * `model.addAttribute(account);`
        - model에는 form으로 들어온 profile과 errors가 이미 들어가 있으므로 account만 넣어주면 된다.
    * `private final AccountService accountService;`
        - 데이터를 변경하는 작업은 accountService에 위임해서 처리하기로 했으므로 주입 받아 온다.
```java
@Controller
@RequiredArgsConstructor
public class SettingsController {

    private static final String SETTINGS_PROFILE_VIEW_NAME = "settings/profile"; // 오타가 자주 나서 바꿈
    private static final String SETTINGS_PROFILE_URL = "/settings/profile";

    private final AccountService accountService;

    @GetMapping(SETTINGS_PROFILE_URL)
    public String profileUpdateForm(@CurrentUser Account account, Model model) {
        model.addAttribute(account);
        model.addAttribute(new Profile(account));

        return SETTINGS_PROFILE_VIEW_NAME;
    }

    @PostMapping("/settings/profile")
    public String updateProfile(@CurrentUser Account account, @Valid Profile profile, Errors errors, Model model) {
        if (errors.hasErrors()) {
            model.addAttribute(account);
            return SETTINGS_PROFILE_VIEW_NAME;
        }

        accountService.updateProfile(account, profile); // account정보를 profile로 업데이트
        return "redirect:" + SETTINGS_PROFILE_URL;
    }
}
```
- AccountService에 `updateProfile()` 추가
```java
@Service
@Transactional
@RequiredArgsConstructor
public class AccountService implements UserDetailsService {

    ....

    public void updateProfile(Account account, Profile profile) {
        account.setBio(profile.getBio());
        account.setUrl(profile.getUrl());
        account.setOccupation(profile.getOccupation());
        account.setLocation(profile.getLocation());
    }
}
```
- 이렇게 하면 될 것 같지만 2가지의 문제가 있다.
    * profile을 바인딩 받을 때 NullPointException이 발생한다.
        - Profile 객체에 기본 생성자가 없기 때문이다.
        - `@ModelAttribute`로 데이터를 받아오려고 할 때 인스턴스를 먼저 만든 다음 setter를 사용해 주입하려고 한다.
        - 이 때 생성자가 1개 밖에 없기 때문에 account를 참조하려 하지만 account는 없다.
        - model에 전달해준 account를 쓸 수 있지 않다. 따라서 기본 생성자를 만들어줘야 한다.
    * DB에 업데이트가 되지 않는다.
        - accountService에서 `completeSignUp()`과 `updateProfile()` 모두 트랜잭션 안에서 처리하지만 객체의 상태가 다르다.
        - `completeSignUp()`은 account의 상태가 persist(영속)다. (accountRepository에서 조회해 왔기 때문) 따라서 DB에 동기화를 한다.
        - `updateProfile()`의 account 상태가 detach(준영속)이다. 세션에 넣어놨던 인증정보에 들어있는 Principal 정보이다.
        - 따라서 아무리 변경해도 변경이력을 감지하지 않고 DB에 반영하지 않는다.
        - accountRepository에 `save()`를 시켜줘야 한다.(merge)
- Profile객체에 기본생성자 추가
```java
@Data
@NoArgsConstructor
public class Profile {

    ...
}
```
- 프로필 수정 항목을 DB에 반영
```java
@Service
@Transactional
@RequiredArgsConstructor
public class AccountService implements UserDetailsService {

    ...

    public void updateProfile(Account account, Profile profile) {
        account.setBio(profile.getBio());
        account.setUrl(profile.getUrl());
        account.setOccupation(profile.getOccupation());
        account.setLocation(profile.getLocation());
        accountRepository.save(account); // save로 DB에 반영(merge) 
    }
}
```
- 이제는 정상적으로 프로필 수정 항목이 변경된다.
- 만약 리다이렉트 시 간단한 데이터를 전달하고 싶다면? 
    * `RedirectAttributes.addFlashAttribute()`
    * Spring MVC에서 제공하는 기능으로 일회용 데이터를 model에 넣고 1번 사용 후 사라진다.
    * [참고 문서](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/mvc/support/RedirectAttributes.html)
```java
@Controller
@RequiredArgsConstructor
public class SettingsController {

    ...

    @PostMapping("/settings/profile")
    public String updateProfile(@CurrentUser Account account, @Valid Profile profile, Errors errors,
                                Model model, RedirectAttributes attributes) {
        if (errors.hasErrors()) {
            model.addAttribute(account);
            return SETTINGS_PROFILE_VIEW_NAME;
        }
        
        accountService.updateProfile(account, profile);
        attributes.addFlashAttribute("message", "프로필을 수정했습니다.");
        return "redirect:" + SETTINGS_PROFILE_URL;
    }
}
```
- profile.html에 메세지를 보여주는 코드 추가
```html
<div th:if="${message}" class="alert alert-info alert-dismissible fade show mt-3" role="alert">
    <span th:text="${message}">메시지</span>
    <button type="button" class="close" data-dismiss="alert" aria-label="Close">
        <span aria-hidden="true">&times;</span>
    </button>
</div>
```
- 실행하면 다음과 같이 프로필을 수정할 때 메세지도 나오고 수정도 성공적으로 완료된다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/account_settings_2.jpg"></p>
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/account_settings_3.jpg"></p>

<br>

## 프로필 수정 테스트
- 지금까지의 테스트는 인증되지 않아도 사용할 수 있었지만 인증된 사용자의 정보를 수정하는 기능을 테스트할 것이다.
- 인증된 사용자가 접근할 수 있는 기능 테스트하기
    * 실제 DB에 저장되어 있는 정보에 대응하는 인증된 Authentication이 필요하다.
    * `@WithMockUser`로는 처리할 수 없다.
- 인증된 사용자를 제공할 커스텀 애노테이션 만들기
    * `@WithAccount`
    * security context를 설정할 수 있는 방법
    * [참고 자료](https://docs.spring.io/spring-security/site/docs/current/reference/html5/#test-method-withsecuritycontext)
<br>

### 테스트 코드 작성
- 커스텀 애노테이션 생성
```java
@Retention(RetentionPolicy.RUNTIME)
@WithSecurityContext(factory = WithAccountSecurityContextFactory.class)
public @interface WithAccount {

    String value();

}
```
- security context를 만들어줄 factory 생성
    * bean으로 등록되기 때문에 다른 bean들을 주입받을 수 있다.
```java
@RequiredArgsConstructor
public class WithAccountSecurityContextFactory implements WithSecurityContextFactory<WithAccount> {

    private final AccountService accountService;

    @Override
    public SecurityContext createSecurityContext(WithAccount withAccount) {
        String nickname = withAccount.value();

        SignUpForm signUpForm = new SignUpForm();
        signUpForm.setNickname(nickname);
        signUpForm.setEmail(nickname + "@email.com");
        signUpForm.setPassword("12345678");
        accountService.processNewAccount(signUpForm);

        // 받아온 value에 해당하는 데이터를 읽어서 SecurityContext에 넣어준다.
        UserDetails principal = accountService.loadUserByUsername(nickname);
        Authentication authentication = new UsernamePasswordAuthenticationToken(principal, principal.getPassword(), principal.getAuthorities());
        SecurityContext context = SecurityContextHolder.createEmptyContext();
        context.setAuthentication(authentication);
        return context;
    }
}
```
- 테스트 코드 작성
    * `@AfterEach`
        - WithAccountSecurityContextFactory에 의해 계정이 생성되므로 테스트 후 반드시 삭제해야 한다.
    * `@WithAccount("hayoung")`
        - 커스텀 애노테이션으로 hayoung이라는 계정 생성
```java
@SpringBootTest
@AutoConfigureMockMvc
class SettingsControllerTest {

    @Autowired MockMvc mockMvc;
    @Autowired AccountRepository accountRepository;

    @AfterEach
    void afterEach() {
        accountRepository.deleteAll();
    }

    @WithAccount("hayoung")
    @DisplayName("프로필 수정 폼")
    @Test
    void updateProfileForm() throws Exception {
        mockMvc.perform(get(SettingsController.SETTINGS_PROFILE_URL))
                .andExpect(status().isOk())
                .andExpect(model().attributeExists("account"))
                .andExpect(model().attributeExists("profile"));
    }

    @WithAccount("hayoung")
    @DisplayName("프로필 수정하기 - 입력값 정상")
    @Test
    void updateProfile() throws Exception {
        String bio = "짧은 소개를 수정하는 경우";

        mockMvc.perform(post(SettingsController.SETTINGS_PROFILE_URL)
                .param("bio", bio)
                .with(csrf()))
                .andExpect(status().is3xxRedirection())
                .andExpect(redirectedUrl(SettingsController.SETTINGS_PROFILE_URL))
                .andExpect(flash().attributeExists("message"));

        Account hayoung = accountRepository.findByNickname("hayoung");
        assertEquals(bio, hayoung.getBio());
    }

    @WithAccount("hayoung")
    @DisplayName("프로필 수정하기 - 입력값 에러")
    @Test
    void updateProfile_error() throws Exception {
        String bio = "35자가 넘도록 길게 소개를 수정하는 경우, 35자가 넘도록 길게 소개를 수정하는 경우, 35자가 넘도록 길게 소개를 수정하는 경우";

        mockMvc.perform(post(SettingsController.SETTINGS_PROFILE_URL)
                .param("bio", bio)
                .with(csrf()))
                .andExpect(status().isOk())
                .andExpect(view().name(SettingsController.SETTINGS_PROFILE_VIEW_NAME))
                .andExpect(model().attributeExists("account"))
                .andExpect(model().attributeExists("profile"))
                .andExpect(model().hasErrors());

        Account hayoung = accountRepository.findByNickname("hayoung");
        assertNull(hayoung.getBio());
    }
}
```
<br>

## 프로필 이미지 변경
- Profile에 프로필 이미지 필드 추가
```java
@Data
@NoArgsConstructor
public class Profile {

    ...

    private String profileImage;

    public Profile(Account account) {
        ...
        this.profileImage = account.getProfileImage();
    }
}

```
- `updateProfile()`에 프로필 이미지 추가
```java
@Service
@Transactional
@RequiredArgsConstructor
public class AccountService implements UserDetailsService {

    ...

    public void updateProfile(Account account, Profile profile) {
        ...
        account.setProfileImage(profile.getProfileImage());
        accountRepository.save(account);
    }
}
```
- 프로필 이미지 뷰 추가
    * type을 hidden으로 설정해서 값을 사용자가 직접 입력하는 것이 아니라 [Cropper.js](https://fengyuanchen.github.io/cropperjs/)를 사용해 이미지 영역을 선택할 수 있도록 한다.
    * Cropper 설치
        - `npm install cropper`
        - `npm install jquery-cropper`
    * [DataURL](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Data_URIs) 사용
        - 이미지를 파일로 db에 저장하지 않고 페이지에 내장된 형태로 사용할 수 있다.
```html
<!DOCTYPE html>
<html lang="en"
      xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments.html :: head"></head>
<body class="bg-light">
<div th:replace="fragments.html :: main-nav"></div>
    <div class="container">
        <div class="row mt-5 justify-content-center">
            <div class="col-2">
                <div th:replace="fragments.html :: settings-menu(currentMenu='profile')"></div>
            </div>
            <div class="col-8">
                <div th:if="${message}" class="alert alert-info alert-dismissible fade show mt-3" role="alert">
                    <span th:text="${message}">메시지</span>
                    <button type="button" class="close" data-dismiss="alert" aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                    </button>
                </div>
                <div class="row">
                    <h2 class="col-sm-12" th:text="${account.nickname}">whiteship</h2>
                </div>
                <div class="row mt-3">
                    <form class="col-sm-6" action="#"
                          th:action="@{/settings/profile}" th:object="${profile}" method="post" novalidate>

                        ...

                        <div class="form-group">
                            <input id="profileImage" type="hidden" th:field="*{profileImage}" class="form-control" />
                        </div>

                        <div class="form-group">
                            <button class="btn btn-primary btn-block" type="submit"
                                    aria-describedby="submitHelp">수정하기</button>
                        </div>
                    </form>
                    <div class="col-sm-6">
                        <div class="card text-center">
                            <div class="card-header">
                                프로필 이미지
                            </div>
                            <div id="current-profile-image" class="mt-3">
                                <svg th:if="${#strings.isEmpty(profile.profileImage)}" class="rounded"
                                     th:data-jdenticon-value="${account.nickname}" width="125" height="125"></svg>
                                <img th:if="${!#strings.isEmpty(profile.profileImage)}" class="rounded"
                                     th:src="${profile.profileImage}"
                                     width="125" height="125" alt="name" th:alt="${account.nickname}"/>
                            </div>
                            <div id="new-profile-image" class="mt-3"></div>
                            <div class="card-body">
                                <div class="custom-file">
                                    <input type="file" class="custom-file-input" id="profile-image-file">
                                    <label class="custom-file-label" for="profile-image-file">프로필 이미지 변경</label>
                                </div>
                                <div id="new-profile-image-control" class="mt-3">
                                    <button class="btn btn-outline-primary btn-block" id="cut-button">자르기</button>
                                    <button class="btn btn-outline-success btn-block" id="confirm-button">확인</button>
                                    <button class="btn btn-outline-warning btn-block" id="reset-button">취소</button>
                                </div>
                                <div id="cropped-new-profile-image" class="mt-3"></div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <link  href="/node_modules/cropper/dist/cropper.min.css" rel="stylesheet">
    <script src="/node_modules/cropper/dist/cropper.min.js"></script>
    <script src="/node_modules/jquery-cropper/dist/jquery-cropper.min.js"></script>
    <script type="application/javascript">
        $(function() {
            cropper = '';
            let $confirmBtn = $("#confirm-button");
            let $resetBtn = $("#reset-button");
            let $cutBtn = $("#cut-button");
            let $newProfileImage = $("#new-profile-image");
            let $currentProfileImage = $("#current-profile-image");
            let $resultImage = $("#cropped-new-profile-image");
            let $profileImage = $("#profileImage");

            $newProfileImage.hide();
            $cutBtn.hide();
            $resetBtn.hide();
            $confirmBtn.hide();

            $("#profile-image-file").change(function(e) {
                if (e.target.files.length === 1) {
                    const reader = new FileReader();
                    reader.onload = e => {
                        if (e.target.result) {
                            let img = document.createElement("img");
                            img.id = 'new-profile';
                            img.src = e.target.result;
                            img.width = 250;

                            $newProfileImage.html(img);
                            $newProfileImage.show();
                            $currentProfileImage.hide();

                            let $newImage = $(img);
                            $newImage.cropper({aspectRatio: 1});
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
                $currentProfileImage.show();
                $newProfileImage.hide();
                $resultImage.hide();
                $resetBtn.hide();
                $cutBtn.hide();
                $confirmBtn.hide();
                $profileImage.val('');
            });

            $cutBtn.click(function () {
                let dataUrl = cropper.getCroppedCanvas().toDataURL();
                let newImage = document.createElement("img");
                newImage.id = "cropped-new-profile-image";
                newImage.src = dataUrl;
                newImage.width = 125;
                $resultImage.html(newImage);
                $resultImage.show();
                $confirmBtn.show();

                $confirmBtn.click(function () {
                    $newProfileImage.html(newImage);
                    $cutBtn.hide();
                    $confirmBtn.hide();
                    $profileImage.val(dataUrl);
                });
            });
        });
    </script>
</body>
</html>
```
- 지금 상태는 프로필 이미지 수정을 해도 수정되지 않는다. 
    * 네비게이션 바의 이미지도 계정이 가지고 있는 데이터로 반영해줘야 한다.
    * account에 있는 프로필 이미지가 없으면 jdenticon으로 보여주고 있으면 account에 있는 프로필 이미지를 그대로 보여주도록 설정
```html
<svg th:if="${#strings.isEmpty(account?.profileImage)}" th:data-jdenticon-value="${#authentication.name}"
    width="24" height="24" class="rounded border bg-light"></svg>
<img th:if="${!#strings.isEmpty(account?.profileImage)}" th:src="${account.profileImage}"
    width="24" height="24" class="rounded border"/>
```
- 실행하면 다음과 같이 프로필 이미지를 잘라서 수정할 수 있고 홈 화면에서도 네이게이션 메뉴에 프로필 이미지가 변경된 것을 확인할 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/account_settings_4.jpg"></p>
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/account_settings_5.jpg"></p>

<br>

## 패스워드 수정

### 목표
- 패스워드 탭 활성화.
- 새 패스워드와 새 패스워드 확인의 값이 일치해야 한다.
- 패스워드 인코딩
- 둘 다 최소 8자에서 최대 50자 사이.
- 사용자 정보를 변경하는 작업.
    * 서비스로 위임해서 트랜잭션 안에서 처리해야 한다.
    * 또는 Detached 상태의 객체를 변경한 다음 Repositoiry의 save를 호출해서 상태 변경 내역을 적용해야 한다.(Merge)
<br>

### 구현
- 패스워드 폼 생성
```java
@Data
public class PasswordForm {

    @Length(min = 8, max = 50)
    private String newPassword;

    @Length(min = 8, max = 50)
    private String newPasswordConfirm;
}
```
- 새로운 비밀번호를 2번 입력받은 두 값이 같은지 확인하는 Validator 생성
    * 다른 bean을 사용할 일이 없으므로 굳이 bean으로 등록할 필요는 없다.
```java
public class PasswordFormValidator implements Validator {

    @Override
    public boolean supports(Class<?> aClass) {
        return PasswordForm.class.isAssignableFrom(aClass); // 어떤 타입의 폼 객체를 검증할 것인가 - PasswordForm 타입에 할당가능한 타입이면 검증하겠다.
    }

    @Override
    public void validate(Object target, Errors errors) {
        PasswordForm passwordForm = (PasswordForm)target; // target 객체를 PasswordForm 타입으로 변경해서 검증
        if (!passwordForm.getNewPassword().equals(passwordForm.getNewPasswordConfirm())) {
            errors.rejectValue("newPassword", "wrong.value", "입력한 새 패스워드가 일치하지 않습니다.");
        }
    }
}
```
- 패스워드 수정 맵핑
    * `@InitBinder("passwordForm")`
        - [회원가입 폼 서브밋 검증](https://github.com/qlalzl9/TIL/blob/a3359d29c14b45d0d57404d640d7597fa395166f/Spring_SpringBoot/devWebservice_Based_on_Spring_and_JPA/signUp.md#%ED%9A%8C%EC%9B%90%EA%B0%80%EC%9E%85-%ED%8F%BC-%EC%84%9C%EB%B8%8C%EB%B0%8B-%EA%B2%80%EC%A6%9D)에서와 같이 커스텀 검증을 수행한다.
    * `@GetMapping(SETTINGS_PASSWORD_URL)`
        - model에 account와 new PasswordForm()를 넘겨주고
        - settings/profile 뷰로 return
    * `@PostMapping(SETTINGS_PASSWORD_URL)`
        - 에러가 있으면 model에 다시 account를 넣고 settings/password 뷰로 return
        - 에러가 없으면 accountService의 `updatePassword()`로 패스워드 변경 후 메세지를 띄우고 /settings/password로 redirect
```java
@Controller
@RequiredArgsConstructor
public class SettingsController {

    @InitBinder("passwordForm")
    public void initBinder(WebDataBinder webDataBinder) {
        webDataBinder.addValidators(new PasswordFormValidator());
    }

    ...

    static final String SETTINGS_PASSWORD_VIEW_NAME = "settings/password";
    static final String SETTINGS_PASSWORD_URL = "/settings/password";

    private final AccountService accountService;

    ...

    @GetMapping(SETTINGS_PASSWORD_URL)
    public String updatePasswordForm(@CurrentUser Account account, Model model) {
        model.addAttribute(account);
        model.addAttribute(new PasswordForm());
        return SETTINGS_PASSWORD_VIEW_NAME;
    }

    @PostMapping(SETTINGS_PASSWORD_URL)
    public String updatePassword(@CurrentUser Account account, @Valid PasswordForm passwordForm, Errors errors,
                                 Model model, RedirectAttributes attributes) {
        if (errors.hasErrors()) {
            model.addAttribute(account);
            return SETTINGS_PASSWORD_VIEW_NAME;
        }

        accountService.updatePassword(account, passwordForm.getNewPassword());
        attributes.addFlashAttribute("message", "패스워드를 변경했습니다.");
        return "redirect:" + SETTINGS_PASSWORD_URL;
    }
}
```
- accountService에 updatePassword() 기능 추가
    * 객체의 상태가 detached 상태이므로 변경이력을 추적하지않는다.
    * 따라서 merge해줘야 한다.
```java
@Service
@Transactional
@RequiredArgsConstructor
public class AccountService implements UserDetailsService {

    ...

    public void updatePassword(Account account, String newPassword) {
        account.setPassword(passwordEncoder.encode(newPassword));
        accountRepository.save(account);
    }
}
```
- 패스워드 변경 뷰 생성
```html
<!DOCTYPE html>
<html lang="en"
      xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments.html :: head"></head>
<body class="bg-light">
    <div th:replace="fragments.html :: main-nav"></div>
    <div class="container">
        <div class="row mt-5 justify-content-center">
            <div class="col-2">
                <div th:replace="fragments.html :: settings-menu(currentMenu='profile')"></div>
            </div>
            <div class="col-8">
                <div th:if="${message}" class="alert alert-info alert-dismissible fade show mt-3" role="alert">
                    <span th:text="${message}">메시지</span>
                    <button type="button" class="close" data-dismiss="alert" aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                    </button>
                </div>
                <div class="row">
                    <h2 class="col-sm-12" >패스워드 변경</h2>
                </div>
                <div class="row mt-3">
                    <form class="needs-validation col-12" action="#"
                          th:action="@{/settings/password}" th:object="${passwordForm}" method="post" novalidate>
                        <div class="form-group">
                            <label for="newPassword">새 패스워드</label>
                            <input id="newPassword" type="password" th:field="*{newPassword}" class="form-control"
                                   aria-describedby="newPasswordHelp" required min="8" max="50">
                            <small id="newPasswordHelp" class="form-text text-muted">
                                새 패스워드를 입력하세요.(8자 이상 50자 이내)
                            </small>
                            <small class="invalid-feedback">패스워드를 입력하세요.</small>
                            <small class="form-text text-danger" th:if="${#fields.hasErrors('newPassword')}" th:errors="*{newPassword}">New Password Error</small>
                        </div>

                        <div class="form-group">
                            <label for="newPasswordConfirm">새 패스워드 확인</label>
                            <input id="newPasswordConfirm" type="password" th:field="*{newPasswordConfirm}" class="form-control"
                                   aria-describedby="newPasswordConfirmHelp" required min="8" max="50">
                            <small id="newPasswordConfirmHelp" class="form-text text-muted">
                                새 패스워드를 다시 한번 입력하세요.(8자 이상 50자 이내)
                            </small>
                            <small class="invalid-feedback">새 패스워드를 다시 입력하세요.</small>
                            <small class="form-text text-danger" th:if="${#fields.hasErrors('newPasswordConfirm')}" th:errors="*{newPasswordConfirm}">New Password Confirm Error</small>
                        </div>

                        <div class="form-group">
                            <button class="btn btn-outline-primary" type="submit" aria-describedby="submitHelp">패스워드 변경하기</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
<script th:replace="fragments.html :: form-validation"></script>
</body>
</html>
```
- 실행 후 패스워드를 수정하면 다음과 같이 성공적으로 변경되는 것을 확인할 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/account_settings_6.jpg"></p>

<br>

## 패스워드 수정 테스트
```java
@SpringBootTest
@AutoConfigureMockMvc
class SettingsControllerTest {

    @Autowired MockMvc mockMvc;
    @Autowired AccountRepository accountRepository;
    @Autowired PasswordEncoder passwordEncoder;

    ...

    @WithAccount("hayoung")
    @DisplayName("패스워드 수정 폼")
    @Test
    void updatePassword_form() throws Exception {
        mockMvc.perform(get(SettingsController.SETTINGS_PASSWORD_URL))
                .andExpect(status().isOk())
                .andExpect(model().attributeExists("account"))
                .andExpect(model().attributeExists("passwordForm"));
    }

    @WithAccount("hayoung")
    @DisplayName("패스워드 수정 - 입력값 정상")
    @Test
    void updatePassword_success() throws Exception {
        mockMvc.perform(post(SettingsController.SETTINGS_PASSWORD_URL)
                .param("newPassword", "12345678")
                .param("newPasswordConfirm", "12345678")
                .with(csrf()))
                .andExpect(status().is3xxRedirection())
                .andExpect(redirectedUrl(SettingsController.SETTINGS_PASSWORD_URL))
                .andExpect(flash().attributeExists("message"));

        Account hayoung = accountRepository.findByNickname("hayoung");
        assertTrue(passwordEncoder.matches("12345678", hayoung.getPassword()));
    }

    @WithAccount("hayoung")
    @DisplayName("패스워드 수정 - 입력값 에러 - 패스워드 불일치")
    @Test
    void updatePassword_fail() throws Exception {
        mockMvc.perform(post(SettingsController.SETTINGS_PASSWORD_URL)
                .param("newPassword", "12345678")
                .param("newPasswordConfirm", "11111111")
                .with(csrf()))
                .andExpect(status().isOk())
                .andExpect(view().name(SettingsController.SETTINGS_PASSWORD_VIEW_NAME))
                .andExpect(model().hasErrors())
                .andExpect(model().attributeExists("passwordForm"))
                .andExpect(model().attributeExists("account"));
    }
}
```
<br>

## 알림 설정

### 목표
- 특정 웹 서비스 이벤트(스터디 생성, 참가 신청 결과, 참여중인 스터디)에 대한 정보를 이메일로 받을지, 웹 알림 메시지로 받을지 선택하는 기능. 
    * 물론 둘 다 받을 수도 있다.
<br>

### 구현
- 알림을 받을지 설정할 폼 객체 생성
    * profile의 경우와 마찬가지로 기본 생성자가 없으면 바인딩 받을 때 NullPointException이 발생하므로 `@NoArgsConstructor` 추가한다.
        - [참고 링크](https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/devWebservice_Based_on_Spring_and_JPA/account_settings.md#%ED%94%84%EB%A1%9C%ED%95%84-%EC%88%98%EC%A0%95-%EC%B2%98%EB%A6%AC)
```java
@Data
@NoArgsConstructor
public class Notifications {

    private boolean studyCreatedByEmail;

    private boolean studyCreatedByWeb;

    private boolean studyEnrollmentResultByEmail;

    private boolean studyEnrollmentResultByWeb;

    private boolean studyUpdatedByEmail;

    private boolean studyUpdatedByWeb;

    public Notifications(Account account) {
        this.studyCreatedByEmail = account.isStudyCreatedByEmail();
        this.studyCreatedByWeb = account.isStudyCreatedByWeb();
        this.studyEnrollmentResultByEmail = account.isStudyEnrollmentResultByEmail();
        this.studyEnrollmentResultByWeb = account.isStudyUpdatedByWeb();
        this.studyUpdatedByEmail = account.isStudyUpdatedByEmail();
        this.studyUpdatedByWeb = account.isStudyUpdatedByWeb();
    }
}
```
- 알림 설정 메서드 `updateNotifications()` 생성
```java
@Service
@Transactional
@RequiredArgsConstructor
public class AccountService implements UserDetailsService {

    ...

    public void updateNotifications(Account account, Notifications notifications) {
        account.setStudyCreatedByWeb(notifications.isStudyCreatedByWeb());
        account.setStudyCreatedByEmail(notifications.isStudyCreatedByEmail());
        account.setStudyUpdatedByWeb(notifications.isStudyUpdatedByWeb());
        account.setStudyUpdatedByEmail(notifications.isStudyUpdatedByEmail());
        account.setStudyEnrollmentResultByEmail(notifications.isStudyEnrollmentResultByEmail());
        account.setStudyEnrollmentResultByWeb(notifications.isStudyEnrollmentResultByWeb());
        accountRepository.save(account);
    }
}
```
- 알림 설정 맵핑
```java
@Controller
@RequiredArgsConstructor
public class SettingsController {

    ...

    static final String SETTINGS_NOTIFICATIONS_VIEW_NAME = "settings/notifications";
    static final String SETTINGS_NOTIFICATIONS_URL = "/settings/notifications";

    private final AccountService accountService;

    ...

    @GetMapping(SETTINGS_NOTIFICATIONS_URL)
    public String updateNotificationsForm(@CurrentUser Account account, Model model) {
        model.addAttribute(account);
        model.addAttribute(new Notifications(account));
        return SETTINGS_NOTIFICATIONS_VIEW_NAME;
    }

    @PostMapping(SETTINGS_NOTIFICATIONS_URL)
    public String updateNotifications(@CurrentUser Account account, @Valid Notifications notifications, Errors errors,
                                      Model model, RedirectAttributes attributes) {
        if (errors.hasErrors()) {
            model.addAttribute(account);
            return SETTINGS_NOTIFICATIONS_VIEW_NAME;
        }

        accountService.updateNotifications(account, notifications);
        attributes.addFlashAttribute("message", "알림 설정을 변경했습니다.");
        return "redirect:" + SETTINGS_NOTIFICATIONS_URL;
    }
}
```
- 알림 설정 뷰 생성
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments.html :: head"></head>
<body class="bg-light">
    <nav th:replace="fragments.html :: main-nav"></nav>
    <div class="container">
        <div class="row mt-5 justify-content-center">
            <div class="col-2">
                <div th:replace="fragments.html :: settings-menu(currentMenu='notifications')"></div>
            </div>
            <div class="col-8">
                <div th:if="${message}" class="alert alert-info alert-dismissible fade show mt-3" role="alert">
                    <span th:text="${message}">완료</span>
                    <button type="button" class="close" data-dismiss="alert" aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                    </button>
                </div>
                <div class="row">
                    <h2 class="col-12">알림 설정</h2>
                </div>
                <div class="row mt-3" th:fragment="profile-form">
                    <form class="col-12" action="#" th:action="@{/settings/notifications}" th:object="${notifications}" method="post" novalidate>
                        <div class="alert alert-light" role="alert">
                            <strong><a href="#" th:href="@{/settings/locations}">주요 활동 지역</a>에
                                <a href="#" th:href="@{/settings/keywords}">관심있는 주제</a>의 스터디가 만들어졌을 때</strong> 알림을 받을 방법을 설정하세요.
                        </div>
                        <div class="form-group">
                            <div class="custom-control custom-switch custom-control-inline">
                                <input type="checkbox" th:field="*{studyCreatedByEmail}" class="custom-control-input" id="studyCreatedByEmail">
                                <label class="custom-control-label" for="studyCreatedByEmail">이메일로 받기</label>
                            </div>
                            <div class="custom-control custom-switch custom-control-inline">
                                <input type="checkbox" th:field="*{studyCreatedByWeb}" class="custom-control-input" id="studyCreatedByWeb">
                                <label class="custom-control-label" for="studyCreatedByWeb">웹으로 받기</label>
                            </div>
                        </div>
                        <div class="alert alert-light" role="alert">
                            <strong>스터디 모임 참가 신청</strong> 결과 알림을 받을 방법을 설정하세요.
                        </div>
                        <div class="form-group">
                            <div class="custom-control custom-switch custom-control-inline">
                                <input type="checkbox" th:field="*{studyEnrollmentResultByEmail}" class="custom-control-input" id="studyEnrollmentResultByEmil">
                                <label class="custom-control-label" for="studyEnrollmentResultByEmil">이메일로 받기</label>
                            </div>
                            <div class="custom-control custom-switch custom-control-inline">
                                <input type="checkbox" th:field="*{studyEnrollmentResultByWeb}" class="custom-control-input" id="studyEnrollmentResultByWeb">
                                <label class="custom-control-label" for="studyEnrollmentResultByWeb">웹으로 받기</label>
                            </div>
                        </div>
                        <div class="alert alert-light" role="alert">
                            <strong>참여중인 스터디</strong>에 대한 알림을 받을 방법을 설정하세요.
                        </div>
                        <div class="form-group">
                            <div class="custom-control custom-switch custom-control-inline">
                                <input type="checkbox" th:field="*{studyUpdatedByEmail}" class="custom-control-input" id="studyWatchByEmail">
                                <label class="custom-control-label" for="studyWatchByEmail">이메일로 받기</label>
                            </div>
                            <div class="custom-control custom-switch custom-control-inline">
                                <input type="checkbox" th:field="*{studyUpdatedByWeb}" class="custom-control-input" id="studyWatchByWeb">
                                <label class="custom-control-label" for="studyWatchByWeb">웹으로 받기</label>
                            </div>
                        </div>
                        <div class="form-group">
                            <button class="btn btn-outline-primary" type="submit" aria-describedby="submitHelp">저장하기</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>

        <div th:replace="fragments.html :: footer"></div>
    </div>
</body>
</html>
```
- 실행하면 다음과 같이 알림 설정을 기능을 사용할 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/account_settings_7.jpg"></p>

<br>

## ModelMapper 적용
- [ModelMapper](http://modelmapper.org/)
    * 객체의 프로퍼티를 다른 객체의 프로퍼티로 맵핑해주는 유틸리티
<br>

### 구현
- ModelMapper 의존성 추가
```xml
<dependency>
	<groupId>org.modelmapper</groupId>
	<artifactId>modelmapper</artifactId>
	<version>2.3.8</version>
</dependency>
```
- Tokenizer 설정 및 Bean 등록
    * ModelMapper를 매번 만들어서 사용할 필요가 없기 때문에 Bean으로 등록해서 사용한다.
    * 데이터를 맵핑할 때 nested한 객체들도 지원하기 때문에 Tokenizer 설정을 추가로 해줘야 한다.
        - 기본 설정은 모든 Tokenizer와 naming pattern이 다 적용된다.
        - Notifications의 경우 camelCase로 되어 있어 nested한 프로퍼티 이름과 비슷하기 때문에 ModelMapper가 이해를 못할 수도 있다.
```java
@Configuration
public class AppConfig {

    ...

    @Bean
    public ModelMapper modelMapper() {
        ModelMapper modelMapper = new ModelMapper();
        modelMapper.getConfiguration()
                .setDestinationNameTokenizer(NameTokenizers.UNDERSCORE)
                .setSourceNameTokenizer(NameTokenizers.UNDERSCORE); // UNDERSCORE만 구별하도록 설정
        return modelMapper();
    }
}
```
- AccountService에 주입받아서 사용
    * ModelMapper에는 `map()` 있는데 source에 있는 데이터를 destination으로 복사해주는 역할을 한다.
        - map(source, destination)
```java
@Service
@Transactional
@RequiredArgsConstructor
public class AccountService implements UserDetailsService {

    private final AccountRepository accountRepository;
    private final JavaMailSender javaMailSender;
    private final PasswordEncoder passwordEncoder;
    private final ModelMapper modelMapper; // ModelMapper 주입

    ...

    public void updateProfile(Account account, Profile profile) {
        modelMapper.map(profile, account); // account에 있는 데이터가 profile에 있는 데이터로 바뀐다.
        accountRepository.save(account); 
    }

    ...

    public void updateNotifications(Account account, Notifications notifications) {
        modelMapper.map(notifications, account); // account에 있는 데이터가 notifications에 있는 데이터로 바뀐다.
        accountRepository.save(account);
    }
}
```
- Profile과 Notifications에 ModelMapper 적용
    * Bean이 아니라 new를 호출해서 만드는 객체이기 때문에 ModelMapper를 주입받지 못한다.
    * ModelMapper를 직접 만들어 사용할 수도 있지만 Controller에서 처리한다.
        - 안에서 만들지 않기 때문에 기본 생성자는 알아서 생긴다. (`@NoArgsConstructor` 제거)
    * 따라서 Profile과 Notifications에 account를 받아서 세팅하는 코드를 지운다.
```java
@Data
public class Profile {

    @Length(max = 35)
    private String bio;

    @Length(max = 50)
    private String url;

    @Length(max = 50)
    private String occupation;

    @Length(max = 50)
    private String location;

    private String profileImage;
}
```
```java
@Data
public class Notifications {

    private boolean studyCreatedByEmail;

    private boolean studyCreatedByWeb;

    private boolean studyEnrollmentResultByEmail;

    private boolean studyEnrollmentResultByWeb;

    private boolean studyUpdatedByEmail;

    private boolean studyUpdatedByWeb;
}
```
```java
@Controller
@RequiredArgsConstructor
public class SettingsController {

    ...

    private final ModelMapper modelMapper;

    ...

    @GetMapping(SETTINGS_PROFILE_URL)
    public String updateProfileForm(@CurrentUser Account account, Model model) {
        model.addAttribute(account);
        model.addAttribute(modelMapper.map(account, Profile.class)); // Profile 타입의 인스턴스가 만들어지고 account에 들어있는 데이터로 채워준다.
        return SETTINGS_PROFILE_VIEW_NAME;
    }

    ...

    @GetMapping(SETTINGS_NOTIFICATIONS_URL)
    public String updateNotificationsForm(@CurrentUser Account account, Model model) {
        model.addAttribute(account);
        model.addAttribute(modelMapper.map(account,Notifications.class)); // Notifications 타입의 인스턴스가 만들어지고 account에 들어있는 데이터로 채워준다.
        return SETTINGS_NOTIFICATIONS_VIEW_NAME;
    }

    ...
}
```
<br>

## 닉네임 수정

### 목표
- 닉네임은 특정 패턴("^[ㄱ-ㅎ가-힣a-z0-9_-]{3,20}$")의 문자열만 지원
- 중복 닉네임 확인.
<br>

### 구현
- NicknameForm 객체 생성
```java
@Data
public class NicknameForm {

    @NotBlank
    @Length(min = 3, max = 20)
    @Pattern(regexp = "^[ㄱ-ㅎ가-힣a-z0-9_-]{3,20}$")
    private String nickname;
}
```
- NicknameForm 객체를 validation할 validator 생성
```java
@Component
@RequiredArgsConstructor
public class NicknameValidator implements Validator {

    private final AccountRepository accountRepository;

    @Override
    public boolean supports(Class<?> clazz) {
        return NicknameForm.class.isAssignableFrom(clazz);
    }

    @Override
    public void validate(Object target, Errors errors) {
        NicknameForm nicknameForm = (NicknameForm) target;
        Account byNickname = accountRepository.findByNickname(nicknameForm.getNickname());
        if (byNickname != null) {
            errors.rejectValue("nickname", "wrong.value", "입력하신 닉네임을 사용할 수 없습니다.");
        }
    }
}
```
- 닉네임 수정 맵핑
```java
@Controller
@RequiredArgsConstructor
public class SettingsController {

    ...
    static final String SETTINGS_ACCOUNT_VIEW_NAME = "settings/account";
    static final String SETTINGS_ACCOUNT_URL = "/" + SETTINGS_ACCOUNT_VIEW_NAME;

    ...
    private final NicknameValidator nicknameValidator;

    ...

    @InitBinder("nicknameForm") // 커스텀 validator 사용
    public void nicknameFormFormInitBinder(WebDataBinder webDataBinder) {
        webDataBinder.addValidators(nicknameValidator);
    }

    ...

    @GetMapping(SETTINGS_ACCOUNT_URL)
    public String updateAccountForm(@CurrentUser Account account, Model model) {
        model.addAttribute(account);
        model.addAttribute(modelMapper.map(account, NicknameForm.class));
        return SETTINGS_ACCOUNT_VIEW_NAME;
    }

    @PostMapping(SETTINGS_ACCOUNT_URL)
    public String updateAccount(@CurrentUser Account account, @Valid NicknameForm nicknameForm, Errors errors,
                                Model model, RedirectAttributes attributes) {
        if (errors.hasErrors()) {
            model.addAttribute(account);
            return SETTINGS_ACCOUNT_VIEW_NAME;
        }

        accountService.updateNickname(account, nicknameForm.getNickname());
        attributes.addFlashAttribute("message", "닉네임을 수정했습니다.");
        return "redirect:" + SETTINGS_ACCOUNT_URL;
    }
}
```
- AccountService에 닉네임 수정 메서드 `updateNickname()` 생성
```java
@Service
@Transactional
@RequiredArgsConstructor
public class AccountService implements UserDetailsService {

    ...

    public void updateNickname(Account account, String nickname) {
        account.setNickname(nickname);
        accountRepository.save(account);
        login(account);
    }
}
```
- 닉네임 수정 뷰 생성
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments.html :: head"></head>
<body class="bg-light">
    <nav th:replace="fragments.html :: main-nav"></nav>
    <div class="container">
        <div class="row mt-5 justify-content-center">
            <div class="col-2">
                <div th:replace="fragments.html :: settings-menu(currentMenu='account')"></div>
            </div>
            <div class="col-8">
                <div th:if="${message}" class="alert alert-info alert-dismissible fade show mt-3" role="alert">
                    <span th:text="${message}">완료</span>
                    <button type="button" class="close" data-dismiss="alert" aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                    </button>
                </div>
                <div class="row">
                    <h2 class="col-12">닉네임 변경</h2>
                </div>
                <div class="row">
                    <form class="needs-validation col-12" th:object="${nicknameForm}" th:action="@{/settings/account}" method="post" novalidate>
                        <div class="alert alert-warning" role="alert">
                            닉네임을 변경하면 프로필 페이지 링크도 바뀝니다!
                        </div>
                        <div class="form-group">
                            <input id="nickname" type="text" th:field="*{nickname}" class="form-control" aria-describedby="nicknameHelp" required>
                            <small id="nicknameHelp" class="form-text text-muted">
                                공백없이 문자와 숫자로만 3자 이상 20자 이내로 입력하세요. 가입후에 변경할 수 있습니다.
                            </small>
                            <small class="invalid-feedback">닉네임을 입력하세요.</small>
                            <small class="form-text text-danger" th:if="${#fields.hasErrors('nickname')}" th:errors="*{nickname}">nickname Error</small>
                        </div>
                        <div class="form-group">
                            <button class="btn btn-outline-primary" type="submit" aria-describedby="submitHelp">변경하기</button>
                        </div>
                    </form>
                </div>
                <div class="dropdown-divider"></div>
                <div class="row">
                    <div class="col-sm-12">
                        <h2 class="text-danger">계정 삭제</h2>
                        <div class="alert alert-danger" role="alert">
                            이 계정은 삭제할 수 없습니다.
                        </div>
                        <button class="btn btn-outline-danger disabled">계정 삭제</button>
                    </div>
                </div>
            </div>
        </div>

        <div th:replace="fragments.html :: footer"></div>
    </div>
    <script th:replace="fragments.html :: form-validation"></script>
</body>
</html>
```
- 실행해서 닉네임을 수정하면 다음과 같이 성공적으로 변경되는 것을 확인할 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/account_settings_8.jpg"></p>

<br>

### 테스트 코드 작성
```java
@SpringBootTest
@AutoConfigureMockMvc
class SettingsControllerTest {

    ...

    @WithAccount("hayoung")
    @DisplayName("닉네임 수정 폼")
    @Test
    void updateAccountForm() throws Exception {
        mockMvc.perform(get(SettingsController.SETTINGS_ACCOUNT_URL))
                .andExpect(status().isOk())
                .andExpect(model().attributeExists("account"))
                .andExpect(model().attributeExists("nicknameForm"));
    }

    @WithAccount("hayoung")
    @DisplayName("닉네임 수정하기 - 입력값 정상")
    @Test
    void updateAccount_success() throws Exception {
        String newNickname = "kimhayoung";
        mockMvc.perform(post(SettingsController.SETTINGS_ACCOUNT_URL)
                .param("nickname", newNickname)
                .with(csrf()))
                .andExpect(status().is3xxRedirection())
                .andExpect(redirectedUrl(SettingsController.SETTINGS_ACCOUNT_URL))
                .andExpect(flash().attributeExists("message"));

        assertNotNull(accountRepository.findByNickname("kimhayoung"));
    }

    @WithAccount("hayoung")
    @DisplayName("닉네임 수정하기 - 입력값 에러")
    @Test
    void updateAccount_failure() throws Exception {
        String newNickname = "¯\\_(ツ)_/¯";
        mockMvc.perform(post(SettingsController.SETTINGS_ACCOUNT_URL)
                .param("nickname", newNickname)
                .with(csrf()))
                .andExpect(status().isOk())
                .andExpect(view().name(SettingsController.SETTINGS_ACCOUNT_VIEW_NAME))
                .andExpect(model().hasErrors())
                .andExpect(model().attributeExists("account"))
                .andExpect(model().attributeExists("nicknameForm"));
    }

    ...
}
```
<br>

## 패스워드를 잃어버린 경우 - 패스워드 없이 로그인 하기(이메일 링크)

### 목표
- 패스워드를 잊은 경우에는 “로그인 할 수 있는 링크”를 이메일로 전송한다.
- 이메일로 전송된 링크를 클릭하면 로그인한다.
- 링크를 너무 자주 보내 악의적인 이용을 막기 위해 시간을 1시간으로 설정
<br>

### 구현
- 맵핑
    * `@GetMapping("/email-login")`
        - 이메일을 입력할 수 있는 폼을 보여주고, 링크 전송 버튼을 제공한다.
    * `@PostMapping("/email-login")`
        - 입력받은 이메일에 해당하는 계정을 찾아보고, 있는 계정이면 로그인 가능한 링크를 이메일로 전송한다.
        - 이메일 전송 후, 안내 메시지를 보여준다.
    * `@GetMapping("/login-by-email")`
        - 토큰과 이메일을 확인한 뒤 해당 계정으로 로그인한다.
```java
@Controller
@RequiredArgsConstructor
public class AccountController {

    ...

    @GetMapping("/email-login")
    public String emailLoginForm() {
        return "account/email-login";
    }

    @PostMapping("/email-login")
    public String sendEmailLoginLink(String email, Model model, RedirectAttributes attributes) {
        Account account = accountRepository.findByEmail(email);
        if (account == null) {
            model.addAttribute("error", "유효한 이메일 주소가 아닙니다.");
            return "account/email-login";
        }

        if (!account.canSendConfirmEmail()) {
            model.addAttribute("error", "이메일 로그인은 1시간 뒤에 사용할 수 있습니다.");
            return "account/email-login";
        }

        accountService.sendLoginLink(account);
        attributes.addFlashAttribute("message", "이메일 인증 메일을 발송했습니다.");
        return "redirect:/email-login";
    }

    @GetMapping("/login-by-email")
    public String loginByEmail(String token, String email, Model model) {
        Account account = accountRepository.findByEmail(email);
        String view = "account/logged-in-by-email";
        if (account == null || !account.isValidToken(token)) {
            model.addAttribute("error", "로그인할 수 없습니다.");
            return view;
        }

        accountService.login(account);
        return view;
    }
}
```
- AccountService에 로그인 링크를 보내는 `sendLoginLink()` 생성
```java
@Service
@Transactional
@RequiredArgsConstructor
public class AccountService implements UserDetailsService {

    ...

    public void sendLoginLink(Account account) {
        account.generateEmailCheckToken();
        SimpleMailMessage mailMessage = new SimpleMailMessage();
        mailMessage.setTo(account.getEmail());
        mailMessage.setSubject("스터디 올래, 로그인 링크");
        mailMessage.setText("/login-by-email?token=" + account.getEmailCheckToken() +
                "&email=" + account.getEmail());
        javaMailSender.send(mailMessage);
    }
}
```
- 패스워드 없이 로그인 관련 뷰 추가
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments.html :: head"></head>
<body class="bg-light">
    <div th:replace="fragments.html :: main-nav"></div>
    <div class="container">
        <div class="py-5 text-center">
            <p class="lead">스터디올래</p>
            <h2>패스워드 없이 로그인하기</h2>
        </div>
        <div class="row justify-content-center">
            <div th:if="${error}" class="alert alert-danger alert-dismissible fade show mt-3" role="alert">
                <span th:text="${error}">완료</span>
                <button type="button" class="close" data-dismiss="alert" aria-label="Close">
                    <span aria-hidden="true">&times;</span>
                </button>
            </div>

            <div th:if="${message}" class="alert alert-info alert-dismissible fade show mt-3" role="alert">
                <span th:text="${message}">완료</span>
                <button type="button" class="close" data-dismiss="alert" aria-label="Close">
                    <span aria-hidden="true">&times;</span>
                </button>
            </div>

            <form class="needs-validation col-sm-6" action="#" th:action="@{/email-login}" method="post" novalidate>
                <div class="form-group">
                    <label for="email">가입 할 때 사용한 이메일</label>
                    <input id="email" type="email" name="email" class="form-control"
                           placeholder="example@email.com" aria-describedby="emailHelp" required>
                    <small id="emailHelp" class="form-text text-muted">
                        가입할 때 사용한 이메일을 입력하세요.
                    </small>
                    <small class="invalid-feedback">이메일을 입력하세요.</small>
                </div>

                <div class="form-group">
                    <button class="btn btn-success btn-block" type="submit"
                            aria-describedby="submitHelp">로그인 링크 보내기</button>
                    <small id="submitHelp" class="form-text text-muted">
                        스터디올래에 처음 오신거라면 <a href="#" th:href="@{/sign-up}">계정을 먼저 만드세요.</a>
                    </small>
                </div>
            </form>
        </div>

        <div th:replace="fragments.html :: footer"></div>
    </div>
    <script th:replace="fragments.html :: form-validation"></script>
</body>
</html>
```
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments.html :: head"></head>
<body class="bg-light">
    <nav th:replace="fragments.html :: main-nav"></nav>

    <div class="container">
        <div class="py-5 text-center" th:if="${error}">
            <p class="lead">스터디올래 이메일 로그인</p>
            <div  class="alert alert-danger" role="alert">
                이메일이 정확하지 않거나 가입하지 않은 이메일 입니다.
            </div>
            <p class="lead" th:text="${email}">your@email.com</p>
        </div>

        <div class="py-5 text-center" th:if="${error == null}">
            <p class="lead">스터디올래 이메일 로그인</p>

            <h2>이메일을 확인하세요. 로그인 링크를 보냈습니다.</h2>
            <p class="lead" th:text="${email}">your@email.com</p>
            <small class="text-info">이메일을 확인해야 로그인 할 수 있습니다.</small>
        </div>
    </div>
</body>
</html>
```
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments.html :: head"></head>
<body class="bg-light">
    <nav th:replace="fragments.html :: main-nav"></nav>

    <div class="container">
        <div class="py-5 text-center" th:if="${error}">
            <p class="lead">스터디올래 이메일 로그인</p>
            <div  class="alert alert-danger" role="alert" th:text="${error}">
                로그인 할 수 없습니다.
            </div>
        </div>

        <div class="py-5 text-center" th:if="${error == null}">
            <p class="lead">스터디올래 이메일 로그인</p>
            <h2>이메일로 로그인 했습니다. <a th:href="@{/settings/password}">패스워드를 변경</a>하세요.</h2>
        </div>
    </div>
</body>
</html>
```
- 실행해서 패스워드 없이 로그인을 하면 다음과 같이 성공적으로 동작한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/account_settings_9.jpg"></p>
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/account_settings_10.jpg"></p>

<br>
