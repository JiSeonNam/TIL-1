# 회원가입과 로그인

## 계정 관리 기능 미리보기
- 회원 가입
- 이메일 인증
- 로그인
- 로그아웃
- 프로필 추가 정보 입력
- 프로필 이미지 등록
- 알림 설정
- 패스워드 수정
- 패스워드를 잊어버린 경우
- 관심 주제(태그) 등록
- 주요 활동 지역 등록
<br>

## 계정 도메인
- domain 패키지에 Account 도메인 생성
    * `@EqualsAndHashCode(of = "id")`
        - 연관관계가 복잡해지면 `equals()`, `hashcode()`에서 서로 다른 연관관계를 계속해서 순환 참조하느라 무한 루프가 발생하고 결국에는 StackOverFlow가 발생할 수 있다.
        - 따라서 id만 사용 
    * email과 nickname은 중복 되면 안된다.(unique)
    * `@Lob @Basic(fetch = FetchType.EAGER)`
        - 프로필 이미지는 VARCHAR(255)보다 커질 가능성이 있고
        - 유저를 로딩할 때 종종 같이 쓰이므로 바로바로 가져올 수 있도록 즉시 로딩 사용
```java
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
@Builder @AllArgsConstructor @NoArgsConstructor
public class Account {

    @Id @GeneratedValue
    private Long id;

    @Column(unique = true)
    private String email;

    @Column(unique = true)
    private String nickname;

    private String password;

    private boolean emailVerified; // 이메일 인증 절차 유무를 판단할 flag

    private String emailCheckToken; // 이메일을 검증할 때 사용할 토큰 값

    private LocalDateTime joinedAt; // 인증을 거친 사용자들의 가입 날짜

    private String bio; // 간단한 자기소개

    private String url; // 웹사이트 url

    private String occupation; // 직업

    private String location; // 거주 지역

    @Lob @Basic(fetch = FetchType.EAGER)
    private String profileImage; // 프로필 이미지

    private boolean studyCreatedByEmail; // 스터디가 생성되면 이메일로 받을 것인지

    private boolean studyCreatedByWeb; // 스터디가 생성되면 웹으로 받을 것인지

    private boolean studyEnrollmentResultByEmail; // 스터디 가입 신청 결과를 이메일로 받을 것인지

    private boolean studyEnrollmentResultByWeb; // 스터디 가입 신청 결과를 웹으로 받을 것인지

    private boolean studyUpdatedByEmail; // 스터디 변경 정보를 이메일로 받을 것인지

    private boolean studyUpdatedByWeb; // 스터디 변경 정보를 웹으로 받을 것인지

}
```
<br>