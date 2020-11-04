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
