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
    * `npm install @yaireo/tagify`
    * [예제](https://yaireo.github.io/tagify/)
    * 모든 POST 요청 뿐 아니라 Ajax요청도 csrf토큰을 전송해야 한다.
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

## 관심 주제 등록 기능
- 요청 본문에 들어오는 데이터를 받아줄 Form 생성
```java
@Data
public class TagForm {

    private String tagTitle;
}
```
- tagRepository 생성
```java
@Transactional(readOnly = true)
public interface TagRepository extends JpaRepository<Tag, Long>{

    Tag findByTitle(String title);
}
```
- PostMapping
```java
@Controller
@RequiredArgsConstructor
public class SettingsController {

    ...

    @PostMapping("/settings/tags/add")
    @ResponseBody // Ajax 요청이므로 응답이 ResponseBody
    public ResponseEntity addTag(@CurrentUser Account account, @RequestBody TagForm tagForm) {
        String title = tagForm.getTagTitle();

        Tag tag = tagRepository.findByTitle(title); // title에 해당하는 태그가 있는지 없는지 조회해서 없으면 새로 DB에 저장 후 account에 추가
        if (tag == null) {
            tag = tagRepository.save(Tag.builder().title(tagForm.getTagTitle()).build());
        }

        accountService.addTag(account, tag);
        return ResponseEntity.ok().build();
    }
}
```
- accountService에 `addTag()` 추가
```java
@Service
@Transactional
@RequiredArgsConstructor
public class AccountService implements UserDetailsService {

    ...

    public void addTag(Account account, Tag tag) {
        Optional<Account> byId = accountRepository.findById(account.getId()); // account가 detached 상태이므로 읽어와야 한다.
        byId.ifPresent(a -> a.getTags().add(tag)); // 있으면 태그 추가

    }
}
```
- 실행해서 관심 주제를 추가하면 다음과 같이 성공적으로 추가된다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/tag_zone_2.jpg"></p>

<br>

## 관심 주제 조회
- 현재는 태그를 입력할 수 있는 Form을 보여줄 때 account 정보만 넘긴다.
- 실제로 뷰에는 등록했던 태그 정보들을 조회할 수 있어야 한다.
<br>

### 구현
- 태그 등록화면에서 등록했던 태그 정보들을 조회
```java
@Controller
@RequiredArgsConstructor
public class SettingsController {

    ...

    @GetMapping(SETTINGS_TAGS_URL)
    public String updateTags(@CurrentUser Account account, Model model) {
        model.addAttribute(account);
        Set<Tag> tags = accountService.getTags(account);
        // Tag 엔티티 타입이 아니라 문자열 형태로 전송
        model.addAttribute("tags", tags.stream().map(Tag::getTitle).collect(Collectors.toList()));

        return SETTINGS_TAGS_VIEW_NAME;
    }

    ...
}
```
- accountService의 `getTags()` 메서드 생성
```java
@Service
@Transactional
@RequiredArgsConstructor
public class AccountService implements UserDetailsService {

    ...

    public Set<Tag> getTags(Account account) {
        Optional<Account> byId = accountRepository.findById(account.getId());
        return byId.orElseThrow().getTags(); // 없으면 에러를 던지고 있으면 태그 정보를 리턴
    }
}
```
- 뷰에서 thymeleaf가 제공하는 유틸리티를 사용해서 Join한다.
    * [참고 - 그 밖의 다양한 유틸리티](https://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf.html#expression-utility-objects)
```html
<input id="tags" type="text" name="tags" th:value="${#strings.listJoin(tags, ',')}" class="tagify-outside" aria-describedby="tagHelp"/>
```
<br>

## 관심 주제 삭제
- 화면에서 이미 삭제 요청은 하고 있으므로 Ajax 요청을 처리하는 핸들러만 구현하면 된다.
- DB에 없는 태그라면, Bad Request로 응답하고
- DB에 있는 태그라면  Account에서 삭제 (DB에서 태그 정보를 삭제하는 것이 아니라 연관 관계만 삭제)
<br>

### 구현
- 핸들러 작성
```java
@Controller
@RequiredArgsConstructor
public class SettingsController {

    ...

    @PostMapping(SETTINGS_TAGS_URL + "/remove")
    @ResponseBody
    public ResponseEntity removeTag(@CurrentUser Account account, @RequestBody TagForm tagForm) {
        String title = tagForm.getTagTitle();

        Tag tag = tagRepository.findByTitle(title);
        if (tag == null) { // 삭제의 경우 없으면 badRequest 응답을 보낸다.
            return ResponseEntity.badRequest().build();
        }

        accountService.removeTag(account, tag); // 정상적인 경우 태그 삭제
        return ResponseEntity.ok().build();
    }
}
```
- accountService에 `removeTag()` 메서드 구현
```java
@Service
@Transactional
@RequiredArgsConstructor
public class AccountService implements UserDetailsService {

    ...

    public void removeTag(Account account, Tag tag) {
        Optional<Account> byId = accountRepository.findById(account.getId());
        byId.ifPresent(a -> a.getTags().remove(tag));
    }
}
```
<br>

## 관심 주제 자동완성
- 태그를 새롭게 등록할 수도 있지만 기존의 있던 태그 목록 중에 선택할 수 있도록 편의 기능 구현
- Tagify의 whitelist를 사용한다.
- [참고 링크](https://yaireo.github.io/tagify/#section-manual-suggestions)
<br>

### 구현
- 컨트롤러에서 뷰를 보여줄 때 태그 목록을 whitelist로 제공
    * list를 json 문자열로 변환하려면 ObjectMapper를 사용하면 된다.
        - Spring boot는 기본적으로 fasterxml이 제공하는 ObjectMapper가 의존성으로 들어와 있고 Bean으로 등록되어 있다.
```java
@Controller
@RequiredArgsConstructor
public class SettingsController {

    ...
    private final ObjectMapper objectMapper;

    ...

    @GetMapping(SETTINGS_TAGS_URL)
    public String updateTags(@CurrentUser Account account, Model model)  throws JsonProcessingException {
        model.addAttribute(account);
        Set<Tag> tags = accountService.getTags(account);
        model.addAttribute("tags", tags.stream().map(Tag::getTitle).collect(Collectors.toList()));

        // 태그들을 모두 가져와서 stream으로 맵핑(타입이 Tag지만 String(태그 이름) 타입으로 변환) 그 다음 list로 변환
        List<String> allTags = tagRepository.findAll().stream().map(Tag::getTitle).collect(Collectors.toList());
        model.addAttribute("whitelist", objectMapper.writeValueAsString(allTags)); // list<String> to JSON

        return SETTINGS_TAGS_VIEW_NAME;
    }

    ...
}
```
- 뷰에서 참조
    * html에서 값을 받고 숨겨둔 다음 id로 js에서 값을 참조한다.
```html
<div id="whitelist" th:text="${whitelist}" hidden></div>

...

var tagify = new Tagify(tagInput, {
                pattern: /^.{0,20}$/,
                whitelist: JSON.parse(document.querySelector("#whitelist").textContent),
                dropdown : {
                    enabled: 1, // suggest tags after a single character input
                } // map tags
            });
```
- 실행해서 확인하면 다음과 같이 이미 등록한 태그들은 다음에, 또는 다른 유저가 등록할 경우 whitelist로 자동완성을 제공한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/tag_zone_3.jpg"></p>

<br>

## 관심 주제 테스트

### Refactoring
- accountService에서 Account 객체 생성 시 builder를 사용하지 않고 modelMapper 사용
```java
@Service
@Transactional
@RequiredArgsConstructor
public class AccountService implements UserDetailsService {

    ...

    // Account 생성
    private Account saveNewAccount(@Valid SignUpForm signUpForm) {
        signUpForm.setPassword(passwordEncoder.encode(signUpForm.getPassword()));
        Account account = modelMapper.map(signUpForm, Account.class);
        account.generateEmailCheckToken();
        
        return accountRepository.save(account);
    }

    ...
}
```
- Account 엔티티에서 tags를 비어있는 값으로 설정
```java
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
@Builder @AllArgsConstructor @NoArgsConstructor
public class Account {

    ...

    @ManyToMany
    private Set<Tag> tags = new HashSet<>();
```
<br>

### 테스트 코드 작성
```java
@Transactional
@SpringBootTest
@AutoConfigureMockMvc
class SettingsControllerTest {

    ...
    @Autowired ObjectMapper objectMapper;
    @Autowired TagRepository tagRepository;
    @Autowired AccountService accountService;

    ...

    @WithAccount("hayoung")
    @DisplayName("태그 수정 폼")
    @Test
    void updateTagsForm() throws Exception {
        mockMvc.perform(get(SettingsController.SETTINGS_TAGS_URL))
                .andExpect(view().name(SettingsController.SETTINGS_TAGS_VIEW_NAME))
                .andExpect(model().attributeExists("account"))
                .andExpect(model().attributeExists("whitelist"))
                .andExpect(model().attributeExists("tags"));
    }

    @WithAccount("hayoung")
    @DisplayName("계정에 태그 추가")
    @Test
    void addTag() throws Exception {
        TagForm tagForm = new TagForm();
        tagForm.setTagTitle("newTag");

        mockMvc.perform(post(SettingsController.SETTINGS_TAGS_URL + "/add")
                .contentType(MediaType.APPLICATION_JSON) // 데이터가 요청안에 파라미터가 아닌 본문으로 들어온다. (JSON 타입)
                .content(objectMapper.writeValueAsString(tagForm)) // objectMapper로 TagForm 객체를 JSON으로 변환해서 본문에 넣어준다.
                .with(csrf())) // POST 요청이므로 csrf 토큰도 같이 보낸다.
                .andExpect(status().isOk());

        // 태그 저장 여부 확인
        Tag newTag = tagRepository.findByTitle("newTag");
        assertNotNull(newTag);
        Account hayoung = accountRepository.findByNickname("hayoung");
        assertTrue(hayoung.getTags().contains(newTag));
    }

    @WithAccount("hayoung")
    @DisplayName("계정에 태그 삭제")
    @Test
    void removeTag() throws Exception {
        // 계정, 태그 만들어서 넣기
        Account hayoung = accountRepository.findByNickname("hayoung");
        Tag newTag = tagRepository.save(Tag.builder().title("newTag").build());
        accountService.addTag(hayoung, newTag);
        
        // 태그 저장 여부 확인
        assertTrue(hayoung.getTags().contains(newTag));

        TagForm tagForm = new TagForm();
        tagForm.setTagTitle("newTag");

        mockMvc.perform(post(SettingsController.SETTINGS_TAGS_URL + "/remove")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(tagForm))
                .with(csrf()))
                .andExpect(status().isOk());
        
        // 계정이 태그를 가지고 있는지 확인
        assertFalse(hayoung.getTags().contains(newTag));
    }
}
```
<br>
