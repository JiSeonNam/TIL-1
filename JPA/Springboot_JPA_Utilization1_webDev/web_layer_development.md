# 웹 계층 개발

## 홈 화면과 레이아웃
- HomeController 생성
    * `@Slf4j`
        - 롬복을 사용하면 Slf4j를 사용해서 로거를 사용할 수 있다.
```java
@Controller
@Slf4j
public class HomeController {

    @RequestMapping("/")
    public String home() {
        log.info("home controller");
        return "home"; // home.html로 연결
    }
}
```
- home.html 파일 작성
    * header, bodyHeader, footer를 import해서 사용한다.
```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments/header :: header">
    <title>Hello</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>

<body>

<div class="container">

    <div th:replace="fragments/bodyHeader :: bodyHeader" />

    <div class="jumbotron">
        <h1>HELLO SHOP</h1>
        <p class="lead">회원 기능</p>
        <p>
            <a class="btn btn-lg btn-secondary" href="/members/new">회원 가입</a>
            <a class="btn btn-lg btn-secondary" href="/members">회원 목록</a>
        </p>
        <p class="lead">상품 기능</p>
        <p>
            <a class="btn btn-lg btn-dark" href="/items/new">상품 등록</a>
            <a class="btn btn-lg btn-dark" href="/items">상품 목록</a>
        </p>
        <p class="lead">주문 기능</p>
        <p>
            <a class="btn btn-lg btn-info" href="/order">상품 주문</a>
            <a class="btn btn-lg btn-info" href="/orders">주문 내역</a>
        </p>
    </div>

    <div th:replace="fragments/footer :: footer" />

</div> <!-- /container -->

</body>
</html>
```
- templates 패키지에 fragments패키지를 생성하고 header, bodyHeader, footer를 각각 생성
    * 예제에서는 단순화를 위해 Include-style layout을 쓰지만 Hierarchical-style layout을 사용하면 파일마다 header, footer의 코드중복을 없앨 수 있다.
    * [참고링크](https://www.thymeleaf.org/doc/articles/layouts.html)
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head th:fragment="header">
    <!-- Required meta tags -->
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink- to-fit=no">

    <!-- Bootstrap CSS -->
    <link rel="stylesheet" href="/css/bootstrap.min.css" integrity="sha384-JcKb8q3iqJ61gNV9KGb8thSsNjpSL0n8PARn9HuZOnIxN0hoP+VmmDGMN5t9UJ0Z" crossorigin="anonymous">

    <!-- Custom styles for this template -->
    <link href="/css/jumbotron-narrow.css" rel="stylesheet">

    <title>Hello, world!</title>
</head>
```
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<div class="header" th:fragment="bodyHeader">
    <ul class="nav nav-pills pull-right">
        <li><a href="/">Home</a></li>
    </ul>
    <a href="/"><h3 class="text-muted">HELLO SHOP</h3></a>
</div>
```
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<div class="footer" th:fragment="footer">
    <p>&copy; Hello Shop V2</p>
</div>
```
- 부트스트랩을 다운로드 받아 static 패키지에 넣고 추가로 jumbotron-narrow.css 파일도 작성한다.
```css
/* Space out content a bit */
body {
    padding-top: 20px;
    padding-bottom: 20px;
}

/* Everything but the jumbotron gets side spacing for mobile first views */
.header,
.marketing,
.footer {
    padding-left: 15px;
    padding-right: 15px;
}

/* Custom page header */
.header {
    border-bottom: 1px solid #e5e5e5;
}

/* Make the masthead heading the same height as the navigation */
.header h3 {
    margin-top: 0;
    margin-bottom: 0;
    line-height: 40px;
    padding-bottom: 19px;
}

/* Custom page footer */
.footer {
    padding-top: 19px;
    color: #777;
    border-top: 1px solid #e5e5e5;
}

/* Customize container */
@media (min-width: 768px) {
    .container {
        max-width: 730px;
    }
}
.container-narrow > hr {
    margin: 30px 0;
}

/* Main marketing message and sign up button */
.jumbotron {
    text-align: center;
    border-bottom: 1px solid #e5e5e5;
}
.jumbotron .btn {
    font-size: 21px;
    padding: 14px 24px;
}

/* Supporting marketing content */
.marketing {
    margin: 40px 0;
}
.marketing p + h4 {
    margin-top: 28px;
}

/* Responsive: Portrait tablets and up */
@media screen and (min-width: 768px) {
    /* Remove the padding we set earlier */
    .header,
    .marketing,
    .footer {
    padding-left: 0;
    padding-right: 0;
    }
    /* Space out the masthead */
    .header {
    margin-bottom: 30px;
    }
    /* Remove the bottom border on the jumbotron for visual effect */
    .jumbotron {
    border-bottom: 0;
    }
}
```
- 애플리케이션을 실행하면 홈 화면이 나오는 것을 확인할 수 있다.
<br>

## 회원 등록
- 폼 객체를 사용해 화면 계층과 서비스 계층을 명확하게 분리한다.
- 회원 등록 폼 객체 생성 : MemberForm
    * `@NotEmpty`
        - 값이 필수로 있어야 하는 경우 애노테이션을 사용해서 Validation 해준다.
        - 값이 비어있으면 오류가 난다.
```java
@Getter @Setter
public class MemberForm {

    @NotEmpty(message = "회원 이름은 필수 입니다.")
    private String name;

    private String city;
    private String street;
    private String zipcode;
}
```
- 회원 등록 컨트롤러 생성 : MemberController
```java
@Controller
@RequiredArgsConstructor
public class MemberController {

    private final MemberService memberService;

    @GetMapping("/members/new")
    public String createForm(Model model) {
        model.addAttribute("memberForm", new MemberForm());
        return "members/createMemberForm";
    }

}
```
- 회원 등록 폼 화면 생성 : createMemberForm.html
```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments/header :: header"/>
<style>
    .fieldError {
        border-color: #bd2130;
    }
</style>
<body>

<div class="container">
    <div th:replace="fragments/bodyHeader :: bodyHeader"/>
    <form role="form" action="/members/new" th:object="${memberForm}" method="post">
        <div class="form-group">
            <label th:for="name">이름</label>
            <input type="text" th:field="*{name}" class="form-control" placeholder="이름을 입력하세요"
                   th:class="${#fields.hasErrors('name')}? 'form-control fieldError' : 'form-control'">
            <p th:if="${#fields.hasErrors('name')}" th:errors="*{name}">Incorrect date</p>
        </div>
        <div class="form-group">
            <label th:for="city">도시</label>
            <input type="text" th:field="*{city}" class="form-control" placeholder="도시를 입력하세요">
        </div>
        <div class="form-group">
            <label th:for="street">거리</label>
            <input type="text" th:field="*{street}" class="form-control" placeholder="거리를 입력하세요">
        </div>
        <div class="form-group">
            <label th:for="zipcode">우편번호</label>
            <input type="text" th:field="*{zipcode}" class="form-control" placeholder="우편번호를 입력하세요">
        </div>
        <button type="submit" class="btn btn-primary">Submit</button>
    </form>
    <br/>
    <div th:replace="fragments/footer :: footer"/>
</div> <!-- /container -->

</body>
</html>
```
- MemberController에 PostMapping 추가
    * url 경로는 같지만 GetMapping은 Form 화면을 열어보고 PostMapping은 데이터를 실제 등록하는 역할을 한다.
    * `@Valid`
        - MemberForm이 javax validation 기능을 쓰는지 인지해서 `@NotEmpty`등을 Validation해준다.
    * `return "redirect://";`
        - 저장이 되면 재로딩되면 좋지 않기 때문에 보통 redirect를 많이 쓴다.
    * `BindingResult`
        - 만약 이름이 없으면 스프링 부트가 기본적으로 제공하는 Whitelabel Error Page가 나온다.
        - 에러를 직접 다루고 싶은 경우 BindingResult를 사용한다.
        - 원래는 오류가 튕겨버리지만 BindingResult를 사용하면 오류가 담겨서 실행된다.
    * `if(result.hasErrors()) { return "members/createMemberForm"; }`
        - Spring과 Thymeleaf는 integration이 잘 되어 있다.
        - 따라서 Spring이 BindingResult를 화면까지 끌고 가서 어떤 에러가 있는지 화면에 뿌릴 수 있다.
        - 의존성에서 thymeleaf도 있지만 thymeleaf-spring도 있다.
```java
@Controller
@RequiredArgsConstructor
public class MemberController {

    private final MemberService memberService;

    @GetMapping("/members/new")
    public String createForm(Model model) {
        model.addAttribute("memberForm", new MemberForm());
        return "members/createMemberForm";
    }

    @PostMapping("/members/new")
    public String create(@Valid MemberForm form, BindingResult result) {

        if(result.hasErrors()) {
            return "members/createMemberForm";
        }

        Address address = new Address(form.getCity(), form.getStreet(), form.getZipcode());

        Member member = new Member();
        member.setName(form.getName());
        member.setAddress(address);

        memberService.join(member);

        return "redirect:/";
    }

}
```
- 실행 후 정상적으로 값을 입력하면 다음과 같이 INSERT쿼리가 나가고 DB에도 데이터가 저장된 것을 볼 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/web_layer_development_1.jpg"></p>

- 실행 후 이름을 입력하지 않으면 BindingResult에 의해 다음과 같이 members/createMemberForm으로 돌아가면서 에러를 알린다.
    * BindingResult를 사용하지 않았다면 스프링 부트의 기본 에러페이지가 나왔을 것이다.
    * 빨간색으로 에러 표시가 되면서 MemberForm에서 `@NotEmpty`의 message가 출력된다. 
    * 자동으로 되는 것이 아니라 createMemberForm.html에서 thymeleaf문법으로 이름 input 태그에서 설정했기 때문에 랜더링이 된 것이다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/web_layer_development_2.jpg"></p>

- 참고로 Member 엔티티를 직접 쓰지 않고 Form을 만들어서 쓰는 이유는 다음과 같다.
    * Controller에서 넘어올 때의 Validation과 실제 도메인이 원하는 Validation이 다를 수 있다.
    * 엔티티가 지저분해진다.
<br>
