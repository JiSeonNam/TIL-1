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
    * 엔티티가 화면 종속적인 기능이 계속 생겨 엔티티가 지저분해진다.
        - 유지보수가 어렵다.
        - 엔티티는 핵심 비즈니스 로직만 가지고 있어야 한다.
<br>

## 회원 목록 조회
- MemberController에 회원 목록 조회 맵핑
    * 목록을 조회해서 뿌릴 때 예제가 단순해서 엔티티를 사용했지만 실무적으로 복잡해지면 엔티티 보다는 DTO로 변환해서 화면에 꼭 필요한 데이터만 가지고 출력하는 것이 좋다.(권장)
    * **API를 만들 때에는 이유불문 절대 엔티티를 넘겨서는 안된다.**
        - API라는 것은 스펙이기 때문에 엔티티에 로직을 추가할 경우 API 스펙이 변해버린다.
```java
@Controller
@RequiredArgsConstructor
public class MemberController {

    ...

    @GetMapping("/members")
    public String list(Model model) {
        List<Member> members = memberService.findMembers();
        model.addAttribute("members", members);
        return "members/memberList";
    }
}
```
- memberList.html 생성
```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments/header :: header" />
<body>

<div class="container">
    <div th:replace="fragments/bodyHeader :: bodyHeader" />
    <div>
        <table class="table table-striped">
            <thead>
            <tr>
                <th>#</th>
                <th>이름</th>
                <th>도시</th>
                <th>주소</th>
                <th>우편번호</th>
            </tr>
            </thead>
            <tbody>
            <!-- thymeleaf에서는 html 문법을 그대로 가져다 쓸 수 있다. -->
            <tr th:each="member : ${members}">
                <td th:text="${member.id}"></td>
                <td th:text="${member.name}"></td>
                <!-- thymeleaf에서 ?문법은 address가 null이면 뒤 코드를 더이상 진행하지 않는다. -->
                <td th:text="${member.address?.city}"></td>
                <td th:text="${member.address?.street}"></td>
                <td th:text="${member.address?.zipcode}"></td> </tr>
            </tbody>
        </table>
    </div>

    <div th:replace="fragments/footer :: footer" />

</div> <!-- /container -->

</body>
</html>
```
- 실행해서 회원 등록 후 회원 목록을 조회하면 다음과 같이 목록이 조회되는 것을 확인할 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/web_layer_development_3.jpg"></p>

<br>

## 상품 등록
- 상품 등록 폼 생성 : BookForm
    * 예제의 단순화를 위해 Book만 한다.
```java
@Getter @Setter
public class BookForm {

    private Long id;

    private String name;
    private int price;
    private int stockQuantity;

    private String author;
    private String isbn;

}
```
- 상품 등록 컨트롤러 생성 : ItemController
```java
@Controller
@RequiredArgsConstructor
public class ItemController {

    private final ItemService itemService;

    @GetMapping("items/new")
    public String createForm(Model model) {
        model.addAttribute("form", new BookForm());
        return "items/createItemForm";
    }

    @PostMapping("items/new")
    public String create(BookForm form) {
        // setter 보다는 따로 create 메소드를 작성하는 것이 좋긴 하다.
        Book book = new Book();
        book.setName(form.getName());
        book.setPrice(form.getPrice());
        book.setStockQuantity(form.getStockQuantity());
        book.setAuthor(form.getAuthor());
        book.setIsbn(form.getIsbn());

        itemService.saveItem(book);

        return "redirect:/items";
    }
}
```
- 상품 등록 뷰 생성 : items/createItemForm.html
    * 상품 등록 폼에서 데이터를 입력하고 Submit 버튼을 클릭하면 /items/new를 POST 방식으로 요청한다.
    * 상품 저장이 끝나면 상품 목록 화면(redirect:/items)으로 redirect한다.
```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments/header :: header" />
<body>

<div class="container">
    <div th:replace="fragments/bodyHeader :: bodyHeader"/>

    <form th:action="@{/items/new}" th:object="${form}" method="post"> <div class="form-group">
        <label th:for="name">상품명</label>
        <input type="text" th:field="*{name}" class="form-control" placeholder="이름을 입력하세요">
    </div>
        <div class="form-group">
            <label th:for="price">가격</label>
            <input type="number" th:field="*{price}" class="form-control" placeholder="가격을 입력하세요">
        </div>
        <div class="form-group">
            <label th:for="stockQuantity">수량</label>
            <input type="number" th:field="*{stockQuantity}" class="form-control" placeholder="수량을 입력하세요">
        </div>
        <div class="form-group">
            <label th:for="author">저자</label>
            <input type="text" th:field="*{author}" class="form-control" placeholder="저자를 입력하세요">
        </div>
        <div class="form-group">
            <label th:for="isbn">ISBN</label>
            <input type="text" th:field="*{isbn}" class="form-control" placeholder="ISBN을 입력하세요">
        </div>
        <button type="submit" class="btn btn-primary">Submit</button> </form>
    <br/>
    <div th:replace="fragments/footer :: footer" />
</div> <!-- /container -->

</body>
</html>
```
- 실행 후 다음과 같은 상품 등록 화면에서 상품 등록을 하면 DB에 잘 저장되는 것을 볼 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/web_layer_development_4.jpg"></p>

<br>

## 상품 목록 조회
- 상품 목록 컨트롤러에 상품 목록 조회 맵핑
```java
@Controller
@RequiredArgsConstructor
public class ItemController {

    ...

    @GetMapping("/items")
    public String list(Model model) {
        List<Item> items = itemService.findItems();
        model.addAttribute("items", items);
        return "items/itemList";
    }

}
```
- 상품 목록 뷰 생성 : items/itemList.html
    * model에 담아둔 상품 목록인 items를 꺼내서 상품 정보를 출력한다.
```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments/header :: header" />
<body>

<div class="container">
    <div th:replace="fragments/bodyHeader :: bodyHeader"/>

    <div>
        <table class="table table-striped">
            <thead>
            <tr>
                <th>#</th>
                <th>상품명</th>
                <th>가격</th>
                <th>재고수량</th>
                <th></th>
            </tr>
            </thead>
            <tbody>
            <tr th:each="item : ${items}">
                <td th:text="${item.id}"></td>
                <td th:text="${item.name}"></td>
                <td th:text="${item.price}"></td>
                <td th:text="${item.stockQuantity}"></td>
                <td>
                    <a href="#" th:href="@{/items/{id}/edit (id=${item.id})}" class="btn btn-primary" role="button">수정</a>
                </td>
            </tr>
            </tbody>
        </table>
    </div>

    <div th:replace="fragments/footer :: footer"/>

</div> <!-- /container -->

</body>
</html>
```
- 실행해서 상품을 등록한 뒤 상품 목록을 조회하면 다음과 같이 결과가 나오는 것을 볼 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/web_layer_development_5.jpg"></p>

<br>

## 상품 수정
- 조회에 비해 수정은 복잡하다.
- JPA에서는 merge(병합)와 변경 감지 2가지 방법이 있다.
- 보통 merge를 많이 쓰지만 JPA에서 권장하는 방법은 변경 감지 방법이다.
- ItemController에 상품 수정 코드 추가 : `updateItemForm()`, `updateItem()`
    * `updateItemForm()`
        - `@PathVariable`
            * itemId는 변경될 수 있으므로 사용한다.
            * [@PathVariable](https://github.com/qlalzl9/TIL/blob/ff09a96af100ff09157f5b634e8c6c77c71739a8/Spring_SpringBoot/SpringMVCUtilization.md#pathvariable)
        - 수정 버튼을 선택하면 /items/{itemId}/edit url을 GET 방식으로 요청한다.
        - `itemService.findOne(itemId);`
            * 수정할 상품을 조회
        - 조회 결과를 모델 객체에 담아서 뷰(items/updateItemForm)에 전달한다.
    * `updateItem()`
        - 상품 수정 폼에서 정보를 수정하고 Submit 버튼을 클릭하면 /items/{itemId}/edit url을 POST 방식으로 요청한다.
        - 컨트롤러에 파라미터로 넘어온 item 엔티티 인스턴스는 준영속 상태로, 영속성 컨텍스트의 지원을 받을 수 없고 데이터를 수정해도 변경 감지 기능은 동작하지 않는다.
        - `@ModelAttribute`
            * 생략가능하다.
            * [@ModelAttribute](https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/SpringMVCUtilization.md#%ED%95%B8%EB%93%A4%EB%9F%AC-%EB%A9%94%EC%86%8C%EB%93%9C-5-modelattribute)
```java
@Controller
@RequiredArgsConstructor
public class ItemController {

    @GetMapping("items/{itemId}/edit")
    public String updateItemForm(@PathVariable("itemId") Long itemId, Model model) {
        // 반환 타입이 Item이지만 예제 단순화를 위해 Book으로 캐스팅해서 사용
        Book item = (Book) itemService.findOne(itemId);

        BookForm form = new BookForm();
        form.setId(item.getId());
        form.setName(item.getName());
        form.setPrice(item.getPrice());
        form.setStockQuantity(item.getStockQuantity());
        form.setAuthor(item.getAuthor());
        form.setIsbn(item.getIsbn());

        model.addAttribute("form", form);
        return "items/updateItemForm";
    }

    @PostMapping("/items/{itemId}/edit")
    public String updateItem(@ModelAttribute("form") BookForm form) {
        Book book = new Book();
        book.setId(form.getId());
        book.setName(form.getName());
        book.setPrice(form.getPrice());
        book.setStockQuantity(form.getStockQuantity());
        book.setAuthor(form.getAuthor());
        book.setIsbn(form.getIsbn());

        itemService.saveItem(book);
        return "redirect:items";

    }
}
```
- 상품 수정 폼 화면 생성 : updateItemForm.html
```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments/header :: header" />
<body>

<div class="container">
    <div th:replace="fragments/bodyHeader :: bodyHeader"/>

    <form th:object="${form}" method="post">
        <!-- id -->
        <input type="hidden" th:field="*{id}" />
        <div class="form-group">
            <label th:for="name">상품명</label>
            <input type="text" th:field="*{name}" class="form-control" placeholder="이름을 입력하세요" />
        </div>
        <div class="form-group">
            <label th:for="price">가격</label>
            <input type="number" th:field="*{price}" class="form-control" placeholder="가격을 입력하세요" />
        </div>
        <div class="form-group">
            <label th:for="stockQuantity">수량</label>
            <input type="number" th:field="*{stockQuantity}" class="form-control" placeholder="수량을 입력하세요" />
        </div>
        <div class="form-group">
            <label th:for="author">저자</label>
            <input type="text" th:field="*{author}" class="form-control" placeholder="저자를 입력하세요" />
        </div>
        <div class="form-group">
            <label th:for="isbn">ISBN</label>
            <input type="text" th:field="*{isbn}" class="form-control" placeholder="ISBN을 입력하세요" />
        </div>
        <button type="submit" class="btn btn-primary">Submit</button>
    </form>

    <div th:replace="fragments/footer :: footer" />

</div> <!-- /container -->

</body>
</html>
```
- 실행해서 상품을 등록하고 수정하면 다음과 같이 수정 완료되어 상품 목록이 조회되는 것을 볼 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/web_layer_development_6.jpg"></p>

<br>

## 변경 감지와 병합(merge)
- 매우 중요하다.
- 준영속 엔티티
    * 영속성 컨텍스트가 더이상 관리하지 않는 엔티티를 말한다.
    * 예제에서는 `itemService.saveItem(book)`에서 수정을 시도하는 `Book`객체이다.
        - Book 객체는 이미 DB에 한번 저장되어서 식별자가 존재한다.
        - 이렇게 임의로 만들어낸 엔티티도 기존 식별자를 가지고 있으면 준영속 엔티티로 볼 수 있다.
- 준영속 엔티티를 수정하는 2가지 방법
    * 변경 감지 기능 사용(dirty checking)
    * 병합 사용(merge)
<br>

### 변경 감지 기능 사용
- 영속성 컨텍스트에서 엔티티를 다시 조회한 후에 데이터를 수정하는 방법
- 트랜잭션 안에서 엔티티를 다시 조회하고 변경하면 트랜잭션 커밋시점에 변경 감지(Dirty Checking)이 동작하여 DB에 UPDATE SQL이 실행된다.
- 다음 코드에서 findItem은 영속 상태이기 때문에 save, persist, merge 같은 것들을 할 필요가 없다.
- 보통 업데이트는 setter를 이용한 단발성 업데이트 보다 `change(price, name, stockQuantity)`, `addStock()`과 같은 의미있는 메서드를 생성해서 변경하는 것이 좋다.
    * 이렇게 해야 변경 지점이 엔티티로 간다.
    * setter를 사용하면 조금만 복잡해져도 어디서 변경되는지 알기 어렵다.(유지 보수 어려움)
```java
@Transactional
    public void updateItem(Long itemId, String name, int price, int stockQuantity) {
        Item findItem = itemRepository.findOne(itemId);
        findItem.setName(name);
        findItem.setPrice(price);
        findItem.setStockQuantity(stockQuantity);
    }
```
<br>

### 병합 사용
- 예제의 상품 수정에서 쓴 방법이 merge를 사용한 방법이다.
    * `itemService.saveItem(book);`하면 itemRepository에 `save()`로 item을 넘기고 id가 있으므로 `em.merge()`가 실행된다.
    * merge에 넘어온 item의 파라미터 값으로 모든 데이터값을 바꾼다.

```java
@PostMapping("/items/{itemId}/edit")
    public String updateItem(@ModelAttribute("form") BookForm form) {
        Book book = new Book();
        book.setId(form.getId());
        book.setName(form.getName());
        book.setPrice(form.getPrice());
        book.setStockQuantity(form.getStockQuantity());
        book.setAuthor(form.getAuthor());
        book.setIsbn(form.getIsbn());

        itemService.saveItem(book);
        return "redirect:/items";

    }
```
<br>

### 병합
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/web_layer_development_7.jpg"></p>

- 병합 동작 방식
    1. `merge()`를 실행한다. 
    2. 파라미터로 넘어온 준영속 엔티티의 식별자 값으로 1차 캐시에서 엔티티를 조회한다. 
        - 만약 1차 캐시에 엔티티가 없으면 DB에서 엔티티를 조회하고, 1차 캐시에 저장한다. 
    3. 조회한 영속 엔티티(mergeMember)에 member엔티티의 값을 채워넣는다. 
        - member엔티티의 모든 값을 mergeMember에 밀어넣는다. 
        - 이 때 mergeMember의 “회원1”이라는 이름이 “회원명변경”으로 바뀐다.
    4. 영속 상태인 mergeMember를 반환한다.
- 병합 동작 방식을 간단히 말하자면
    * 준영속 엔티티의 식별자 값으로 영속 엔티티를 조회한다. 
    * 영속 엔티티의 값을 준영속 엔티티의 값으로 모두 교체한다.(병합한다.) 
    * 트랜잭션 커밋 시점에 변경 감지 기능이 동작해서 DB에 UPDATE SQL이 실행된다.
- 병합 주의점
    * 변경 감지 기능을 사용하면 원하는 속성만 선택해서 변경할수있지만, 병합을 사용하면 모든 속성이 변경된다. 
    * 병합 시 값이 없으면 null로 업데이트할 위험도 있다.
        - 병합은 모든 필드를 교체한다.
<br>

### 가장 좋은 해결 방법
- **엔티티를 변경할 때는 항상 변경 감지를 사용하자**
- 컨트롤러에서 어설프게 엔티티를 생성하지 말아야 한다.
- 트랜잭션이 있는 서비스 계층에 식별자(id)와 변경할 데이터를 명확하게 전달한다.
    * 파라미터
    * Dto
- 트랜잭션이 있는 서비스 계층에서 영속 상태의 엔티티를 조회하고, 엔티티의 데이터를 직접 변경한다.
- 트랜잭션 커밋 시점에 변경 감지가 실행된다.
```java
@PostMapping("/items/{itemId}/edit")
public String updateItem(@ModelAttribute("form") BookForm form) {
    // Book book = new Book();
    // book.setId(form.getId());
    // book.setName(form.getName());
    // book.setPrice(form.getPrice());
    // book.setStockQuantity(form.getStockQuantity());
    // book.setAuthor(form.getAuthor());
    // book.setIsbn(form.getIsbn());
    // itemService.saveItem(book);
    // 이렇게 어설프게 엔티티를 생성하지 말아야 한다.

    // 이렇게 식별자와 변경 데이터를 명확하게 전달하는 것이 유지보수성이 좋고 깔끔하다.
    itemService.updateItem(itemId, form.getName(), form.getPrice(), form.getStockQuantity());

    return "redirect:/items";

}
```
- 만약 업데이트할 데이터가 많으면 service계층에 Dto를 생성해서 해결하는 것이 좋다.
<br>

## 상품 주문
- 상품 주문 컨트롤러 생성 : OrderController
    * Member와 Item을 모두 선택할 수 있어야 한다.
    * `createForm()`
        - 상품 주문을 선택하면 /order를 GET 방식으로 호출한다.
        - 주문 화면에는 주문할 Member정보와 Item 정보가 필요하므로 model 객체에 담아서 뷰에 넘겨준다.
    * `order()`
        - 주문할 회원과 상품, 수량을 선택해서 Submit 버튼을 누르면 /order URL을 POST 방식으로 호출한다.
        - 회원 id, 상품 id, 수량 정보를 받아서 orderService에 주문을 요청한다.
        - 주문이 끝나면 상품 주문 내역이 있는 /orders로 redirect한다.
        - [@RequestParam](https://github.com/qlalzl9/TIL/blob/ff09a96af100ff09157f5b634e8c6c77c71739a8/Spring_SpringBoot/SpringMVCUtilization.md#%ED%95%B8%EB%93%A4%EB%9F%AC-%EB%A9%94%EC%86%8C%EB%93%9C-3-%EC%9A%94%EC%B2%AD-%EB%A7%A4%EA%B0%9C%EB%B3%80%EC%88%98)
```java
@Controller
@RequiredArgsConstructor
public class OrderController {

    private final OrderService orderService;
    private final MemberService memberService;
    private final ItemService itemService;

    @GetMapping("/order")
    public String createForm(Model model) {

        List<Member> members = memberService.findMembers();
        List<Item> items = itemService.findItems();

        model.addAttribute("members", members);
        model.addAttribute("items", items);

        return "order/orderForm";
    }

    @PostMapping("/order")
    public String order(@RequestParam("memberId") Long memberId,
                        @RequestParam("itemId") Long itemId,
                        @RequestParam("count") int count) {
        orderService.order(memberId, itemId, count);
        return "redirect:/orders";
    }
}
```
- 상품 주문 폼 생성 : orderForm.html
```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments/header :: header"/>
<body>

<div class="container">
    <div th:replace="fragments/bodyHeader :: bodyHeader"/>

    <form role="form" action="/order" method="post">

        <div class="form-group">
            <label for="member">주문회원</label>
            <select name="memberId" id="member" class="form-control">
                <option value="">회원선택</option>
                <option th:each="member : ${members}"
                        th:value="${member.id}"
                        th:text="${member.name}"/>
            </select>
        </div>

        <div class="form-group">
            <label for="item">상품명</label>
            <select name="itemId" id="item" class="form-control">
                <option value="">상품선택</option>
                <option th:each="item : ${items}"
                        th:value="${item.id}"
                        th:text="${item.name}"/>
            </select>
        </div>

        <div class="form-group">
            <label for="count">주문수량</label>
            <input type="number" name="count" class="form-control" id="count"
                   placeholder="주문 수량을 입력하세요">
        </div>
        <button type="submit" class="btn btn-primary">Submit</button>
    </form>
    <br/>

    <div th:replace="fragments/footer :: footer"/>

</div> <!-- /container -->

</body>
</html>
```
<br>
