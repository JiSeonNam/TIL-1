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

    private String bio;

    private String url;

    private String occupation;

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
