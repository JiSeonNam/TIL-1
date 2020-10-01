# 회원 도메인 개발
- 구현 기능
    * 회원 등록
    * 회원 목록 조회
- 순서
    * 회원 엔티티 코드 다시 보기
    * 회원 Repository 개발
    * 회원 Service개발
    * 회원 기능 테스트
<br>

## 회원 Repository 개발
- repository 패키지 생성 및 MemberRepository 생성
    * `@PersistenceContext`
        * 스프링이 JPA의 EntityManager를 주입해준다.
        * 만약 EntityManagerFactory를 주입받고 싶으면 `@PersistenceUnit`어노테이션을 사용하면 된다.
```java
@Repository
public class MemberRepository {
    @PersistenceContext
    private EntityManager em;

    public void save(Member member) {
        em.persist(member);
    }

    public Member findOne(Long id) {
        return em.find(Member.class, id);
    }

    // 전체 조회는 JPQL을 작성해야 한다.
    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }

    // 만약 이름으로 회원으로 검색하게 되면 파라미터를 바인딩해서 조회한다.
    public List<Member> findByName(String name) {
        return em.createQuery("select m from Member m where m.name = :name", Member.class)
                .setParameter("name", name)
                .getResultList();
    }
}
```
<br>

## 회원 Service 개발
- service 패키지 생성 및 MemberService 생성
    * `@Transactional`
        - JPA의 모든 로직이나 데이터 변경은 트랜잭션 안에서 실행되야 한다.
        - javax가 아닌 스프링에서 제공하는 `@Transactional`을 써야 쓸 수 있는 옵션이 많다.(권장)
        - 클래스에 조회가 많으므로 `readOnly = true`로 성능을 최적화하고 쓰기인 join()에서는 `@Transactional`으로 따로 표시해서 우선권을 준다.
            * 기본값 : `readOnly = false`
    * `@RequiredArgsConstructor`
        - 필드나 Setter를 통한 주입보다는 생성자를 통한 주입이 권장된다.
            * 변경 불가능한 안전한 객체 생성이 가능하다.
            * 생성자가 하나이면 `@Autowired`를 생략할 수 있다.
            * final 키워드를 추가하면 컴파일 시점에 memberRepository를 설정하지 않는 오류를 체크할 수 있다.
        - lombok을 사용하여 `@RequiredArgsConstructor`만 써주면 스프링 부트에서 자동으로 생성자를 만들어준다. 
```java
@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class MemberService {

    private final MemberRepository memberRepository;


    /**
     * 회원 가입
     */
    @Transactional
    public Long join(Member member) {
        validateDuplicateMember(member); // 중복 회원 검증
        //save를 통해 persist()하면 영속성 컨텍스트에 member객체를 올리고 그 때 영속성 컨텍스트는 key인 id값은 항상 보장된다.
        memberRepository.save(member);
        return member.getId();
    }

    private void validateDuplicateMember(Member member) {
        List<Member> findMembers = memberRepository.findByName(member.getName());
        if(!findMembers.isEmpty()) {
            throw new IllegalStateException(("이미 존재하는 회원입니다."));
        }
    }

    /**
     * 회원 전체 조회
     */
    public List<Member> findMembers() {
        return memberRepository.findAll();
    }

    // 하나만 조회
    public Member findOne(Long memberId) {
        return memberRepository.findOne(memberId);
    }
}
```
- 참고로 스프링 데이터 JPA를 사용하면 EntityManager도 주입이 가능하기 때문에 memberRepository를 다음 코드와 같이 변경할 수 있다.
```java
@Repository
@RequiredArgsConstructor
public class MemberRepository {

    private EntityManager em;

    ...
}
```
<br>

## 회원 기능 테스트
- 테스트 요구사항
    * 회원가입을 성공해야 한다.
    * 회원가입 할 때 같은 이름이 있으면 예외가 발생해야 한다.
- MemberServiceTest 생성
    * 테스트를 보면 DB에 INSERT쿼리가 나가지 않은 것을 볼 수 있다.
    * 스프링에서 `@Transactional`은 트랜잭션을 commit하지 않고 rollback하기 때문이다.
        - `@Rollback(false)`로 설정하고 테스트를 실행하면 INSERT 쿼리가 나간 것을 확인할 수 있다.
    - 만약 rollback을 하지만 DB에 쿼리가 나가는 것을 보고싶으면 EntityManager를 주입받고 검증단계에서 flush()하면 INSERT 쿼리가 나간 뒤에 rollback된다.
```java
@SpringBootTest
@Transactional
class MemberServiceTest {

    @Autowired
    MemberService memberService;
    @Autowired
    MemberRepository memberRepository;
    @Autowired
    EntityManager em;

    @Test
    void 회원가입() throws Exception {
        //given
        Member member = new Member();
        member.setName("kim");

        //when
        Long savedId = memberService.join(member);

        //then
        assertEquals(member, memberRepository.findOne(savedId));
    }

    @Test
    void 중복_회원_예외() throws Exception {
        //given
        Member member1 = new Member();
        member1.setName("kim");

        Member member2 = new Member();
        member2.setName("kim");

        //when
        memberService.join(member1);
        memberService.join(member2); // 예외 발생해야 한다.

        //then
        IllegalStateException thrown = assertThrows(IllegalStateException.class, () -> memberService.join(member2));
        assertEquals("이미 존재하는 회원입니다.", thrown.getMessage());
    }
}
```
- 외부 DB(H2)를 사용한 테스트는 병렬로 여러 개 돌리거나 DB를 외부에 설치해야하는 단점들이 있다.
- 테스트를 완전히 격리된 환경, 자바를 띄울 때 자바안에 DB를 새로 만들어서 끼우는 방법을 스프링부트가 제공한다.
    * 메모리 DB
<br>

### 메모리 DB
- 패키지 구조에서 운영로직은 main에서 우선권을 가지고 test는 test 패키지에서 우선권을 가진다.
    * application.yml을 test 패키지에 따로 만들면 test를 실행할 때는 test에 있는 application.yml이 설정대로 실행된다.
- 따라서 h2에서 제공하는 메모리 DB의 주소를 url에 수정하면 메모리 모드로 DB가 동작한다.
```yml
spring:
#  datasource:
#    url: jdbc:h2:mem:test
#    username: sa
#    password:
#    driver-class-name: org.h2.Driver
#
#  jpa:
#    hibernate:
#      ddl-auto: create
#    properties:
#      hibernate:
#        show_sql: true
#        format_sql: true

logging:
  level:
    org.hibernate.SQL: debug
```