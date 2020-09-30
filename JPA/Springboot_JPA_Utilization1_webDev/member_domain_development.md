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