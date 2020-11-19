# 관심 주제와 지역 정보
- 주요 기능
    * 관심 주제 등록
    * 관심 주제 삭제
    * 지역 정보 데이터 초기화
    * 지역 정보 등록 
    * 지역 정보 삭제
<br>

## 관심 주제 도메인
- 관심 주제(tag)는 엔티티로 만든다.
    * tag로 특정 데이터를 검색도 하고 다른 곳(study)에서 참조도 할 것이기 때문
- 객체 관점에서의 관계
    * Account -> Tag
    * ManyToMany
    * Account에서 Tag를 참조(단방향)
- DB 관점에서의 관계
    * Account <- Account_Tag -> Tag
    * 조인 (join) 테이블을 사용해서 다대다 관계를 표현.
    * Account_Tag에서 Account의 PK 참조.
    * Account_Tag에서 Tag의 PK 참조.
<br>

### 구현
- Tag 엔티티 생성
```java
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
@Builder @AllArgsConstructor @NoArgsConstructor
public class Tag {

    @Id
    @GeneratedValue
    private Long id;

    @Column(unique = true, nullable = false)
    private String title;
}
```
- Account 엔티티에 연관관계 맵핑
```java
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
@Builder @AllArgsConstructor @NoArgsConstructor
public class Account {

    ...

    @ManyToMany
    private Set<Tag> tags;

    ...
}
```
- DB 관련 설정
```properties
# 개발할 때에만 create-drop 또는 update를 사용하고 운영 환경에서는 validate를 사용합니다.
spring.jpa.hibernate.ddl-auto=create-drop

# 개발시 SQL 로깅을 하여 어떤 값으로 어떤 SQL이 실행되는지 확인합니다.
spring.jpa.properties.hibernate.format_sql=true
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```
- [Tagify](https://github.com/yairEO/tagify)를 사용해 tag기능 구현
    * `npm install @yaireo/tagify`
    * [예제](https://yaireo.github.io/tagify/)
<br>

## 관심 주제 등록 뷰
- Tag 관련 맵핑 추가
```java
@Controller
@RequiredArgsConstructor
public class SettingsController {

    ...
    static final String SETTINGS_TAGS_VIEW_NAME = "settings/tags";
    static final String SETTINGS_TAGS_URL = "/" + SETTINGS_TAGS_VIEW_NAME;

    ...

    @GetMapping(SETTINGS_TAGS_URL)
    public String updateTags(@CurrentUser Account account, Model model) {
        model.addAttribute(account);
        return SETTINGS_TAGS_VIEW_NAME;
    }
}
```
- Tag 뷰 생성
    * [Tagify](https://github.com/yairEO/tagify)를 사용해 tag기능 구현
    * npm install @yaireo/tagify
    * [예제](https://yaireo.github.io/tagify/)
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments.html :: head"></head>
<body class="bg-light">
    <div th:replace="fragments.html :: main-nav"></div>
    <div class="container">
        <div class="row mt-5 justify-content-center">
            <div class="col-2">
                <div th:replace="fragments.html :: settings-menu(currentMenu='tags')"></div>
            </div>
            <div class="col-8">
                <div class="row">
                    <h2 class="col-12">관심있는 스터디 주제</h2>
                </div>
                <div class="row">
                    <div class="col-12">
                        <div class="alert alert-info" role="alert">
                            참여하고 싶은 스터디 주제를 입력해 주세요.<br>해당 주제의 스터디가 생기면 알림을 받을 수 있습니다.<br>태그를 입력하고 콤마(,)
                            또는 엔터를 입력하세요.
                        </div>
                        <input id="tags" type="text" name="tags" class="tagify-outside" aria-describedby="tagHelp"/>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <script src="/node_modules/@yaireo/tagify/dist/tagify.min.js"></script>
    <script type="application/javascript" th:inline="javascript">
        $(function() {
            var csrfToken = /*[[${_csrf.token}]]*/ null;
            var csrfHeader = /*[[${_csrf.headerName}]]*/ null;
            $(document).ajaxSend(function (e, xhr, options) {
                xhr.setRequestHeader(csrfHeader, csrfToken);
            });
        });
    </script>
    <script type="application/javascript">
        $(function () {
            function tagRequest(url, tagTitle) {
                $.ajax({
                    dataType: "json",
                    autocomplete: {
                        enabled: true,
                        rightKey: true,
                    },
                    contentType: "application/json; charset=utf-8",
                    method: "POST",
                    url: "/settings/tags" + url,
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
</body>
</html>
```
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/tag_zone_1.jpg"></p>

<br>
