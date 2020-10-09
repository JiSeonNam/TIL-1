# API 개발 기본
- API tool Postman 설치
    * [Postman](https://www.postman.com/downloads/)
- 참고로 화면과 api의 패키지를 분리시키는 것이 좋다.
    * 공통으로 예외처리를 할 때 패키지 등의 구성 단위로 처리를 하는 경우 많다.
    * 화면과 api는 공통 처리를 할 요소들이 다르다.
    * 화면은 문제가 생기면 공통 에러 html이 나가야 하지만 api는 공통 에러용 JSON API 스펙이 나가야 한다.
<br>

## 회원 등록 API
- api 패키지에 MemberApiController 생성
- 2가지 방법이 있다.
<br>

### 엔티티 사용 방법
- `@RequestBody`
    * 응답 데이터(XML, JSON 등)를 객체로 맵핑해서 넣어준다.
```java
@RestController
@RequiredArgsConstructor
public class MemberApiController {

    private final MemberService memberService;

    @PostMapping("/api/v1/members")
    public CreateMemberResponse saveMemberV1(@RequestBody @Valid Member member) {
        Long id = memberService.join(member);
        return new CreateMemberResponse(id);
    }

    @Data
    static class CreateMemberResponse {
        private Long id;

        public CreateMemberResponse(Long id) {
            this.id = id;
        }
    }
}
```
- 실행 후 Postman으로 확인하면 잘 저장되고 id가 1로 넘어 온 것을 볼 수 있다.
    * DB 쿼리 또한 INSERT로 잘 들어간 것을 볼 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/API_dev_basic_1.jpg"></p>

- Postman에서 name을 빼고 넣어도 저장이 된다.
    * Member 엔티티에서 name에 제약조건을 넣지 않았기 때문이다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/API_dev_basic_2.jpg"></p>

- 따라서 Member 엔티티의 name에 `@NotEmpty`를 추가하고 실행한 후 Postman에서 name을 빼고 보내면 스프링 부트가 기본으로 설정해 놓은 에러가 발생한다.
    * MemberApiController의 `@Valid` 애노테이션이 javax.validation 검증을 하기 때문이다.
    * 이름을 넣어야만 정상적으로 실행된다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/API_dev_basic_3.jpg"></p>

- 그러나 이 방법에는 여러가지 문제가 있다.
    * 프레젠테이션 계층을 위한 검증 로직이 엔티티에 들어가 있으면 문제가 있다.
        - 어떤 API는 `@NotEmpty`가 필요하지만 어떤 API에서는 필요없을 경우 
    * 만약 엔티티의 name을 username으로 바꾼다면? API 스펙이 바뀌어 버린다. 
- **따라서 결론적으로 API 스펙을 위한 Dto를 만들어야 한다.**
- API를 만들 때는 항상 엔티티를 파라미터로 받지 않는 것이 좋다.
    * 엔티티를 외부에 노출하면 안된다.
<br>

### Dto 사용 방법
```java
@RestController
@RequiredArgsConstructor
public class MemberApiController {

    private final MemberService memberService;

    @PostMapping("/api/v2/members")
    public CreateMemberResponse saveMemberV2(@RequestBody @Valid CreateMemberRequest request) {
        Member member = new Member();
        member.setName(request.getName());

        Long id = memberService.join(member);
        return new CreateMemberResponse(id);
    }

    @Data
    static class CreateMemberRequest {
        private String name;
    }
}
```
- 실행 후 Postman으로 확인하면 잘 저장되고 id도 잘 넘어온 것을 볼 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/API_dev_basic_4.jpg"></p>

- 첫 번째 방법의 유일한 방법은 엔티티를 쓰기 때문에 클래스를 추가로 만들지 않아도 된다는 것이다.
- Dto를 쓴다면 엔티티와 API 스펙이 명확하게 분리되어 엔티티가 변경되어도 API 스펙이 바뀌지 않는다.
    * 만약 Member엔티티의 name을 username으로 바꾼다면 컴파일 오류가 나서 알려준다.
    * `setName`을 `setUsername`으로만 바꿔주면 스펙 변화없이 끝이다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/API_dev_basic_5.jpg"></p>

- **API를 만들 때는 엔티티를 절대 사용하지 말고 Dto를 생성해서 사용하자!**
<br>
