# 영속성 관리 - 내부 동작 방식

## 영속성 컨텍스트
- JPA를 이해하려면 영속성 컨텍스트를 이해해야한다.
- JPA에서 가장 중요한 것
    * 객체와 관계형 DB 맵핑하기
    * 영속성 컨텍스트
<br>

### 엔티티 매니저 팩토리와 엔티티 매니저
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_PersistenceContext_1.jpg"></p>
- 사용자의 요청이 올 때마다 EntityManagerFactory를 통해서 EntityManager를 생성한다.
- EntityMagager는 내부적으로 DB 커넥션을 사용해서 DB를 사용한다.
<br>

### 영속성 컨텍스트
- JPA를 이해하는데 가장 중요한 용어로 "엔티티를 영구 저장하는 환경"이라는 뜻이다.
- `EntityManager.persist(entity);`
    * 앞 내용에서는 `persist()`로 DB에 객체(entity)를 저장하는 것이라고 배웠지만
    * 실제로는 DB에 저장한다는 것이 아니라, 영속성 컨텍스트를 통해서 엔티티를 영속화 한다는 뜻이다.
    * 정확히는 `persist()`는 DB에 저장하는 것이 아니라 entity를 영속성 컨텍스트라는 곳에 저장한다.
- 영속성 컨텍스트는 눈에 보이지 않는 논리적인 개념이다.
- 엔티티 매니저를 통해서 영속성 컨텍스트에 접근한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_PersistenceContext_2.jpg"></p>

- 스프링에서 EntityManager를 주입 받아서 쓰면, 같은 트랜잭션의 범위에 있는 EntityManager는 동일 영속성 컨텍스트에 접근한다.
<br>

### 엔티티의 생명주기
- 비영속(new/transient)
    * 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태.
- 영속(managed)
    * 영속성 컨텍스트에 관리되는 상태.
    * 엔티티 매니저안에는 영속성 컨텍스트가 있어서 객체를 생성한 다음에 persist()로 객체를 넣으면 영속 상태가 된다.
    * 이 때 DB에 저장되지 않고 트랜잭션의 커밋 시점에 영속성 컨텍스트에 있는 정보들이 DB에 쿼리로 날라간다.
- 준영속(detached)
    * 엔티티가 영속성 컨텍스트에서 분리된 상태.
- 삭제(removed)
    * 객체를 삭제한 상태.
```java
public class JpaMain {
    public static void main(String[] args) {

        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");

        EntityManager em = emf.createEntityManager();

        EntityTransaction tx = em.getTransaction();
        tx.begin();

        try {
            // 비영속
            Member member = new Member();
            member.setId("member1");
            member.setName("회원1")

            // 영속
            em.persist(member);

            // 준영속 상태
            em.detach(member);

            // 삭제
            em.remove(member);
            }

            tx.commit();
        } catch (Exception e) {
            tx.rollback();
        } finally {
            em.close();
        }

        emf.close();
    }
}
```
<br>

### 영속성 컨텍스트의 이점
- 1차 캐시
- 동일성 보장
- 트랜잭션을 지원하는 쓰기 지연
- Dirty Checking(변경 감지)
- 지연 로딩(Lazy Loading)
<br>

### 1차 캐시
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_PersistenceContext_3.jpg"></p>

- 영속성 컨텍스트는 내부에 1차 캐시를 가지고 있다.
- 엔티티가 영속성 컨텍스트에 저장(persist)되면 1차 캐시에 Map이 저장된다.
    * key : DB PK로 맵핑한 @Id 필드값
    * value : 엔티티 객체 자체
- 1차 캐시는 조회할 때 이점이 있다.
    * find()로 조회를 하면 먼저 DB가 아니라 1차 캐시에서 조회를 한다. 
    * 1차 캐시에 있으면 캐시에 있는 값을 바로 조회한다.
    * 1차 캐시에 없으면 DB에서 조회해서 조회한 것을 1차 캐시에 저장하고 반환한다. (이후에 다시 조회하면 1차 캐시에서 반환한다.)
- 엔티티 매니저는 DB 트랜잭션 단위로 만들고 트랜잭션이 끝나면 종료되기 때문에 1차 캐시도 종료된다.
    * 애플리케이션 전체에서 공유하는 캐시는 2차 캐시라고 한다.
    * 1차 캐시는 DB 트랜잭션 안에서만 효과가 있기 때문에 큰 성능 이점은 아니다.
        - 비즈니스 로직이 굉장히 복잡하면 좋다.
- 다음 코드를 실행하면 조회를 했는데도 DB의 SELECT 쿼리를 안나왔다.
    * `persist()`로 저장할 때 1차 캐시에 저장되고 조회할 때도 DB보다 1차 캐시에서 먼저 조회해서 가져오기 때문이다.
    * 저장할 때 1차 캐시에 올려놓는다.
```java
public class JpaMain {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");

        EntityManager em = emf.createEntityManager();

        EntityTransaction tx = em.getTransaction();
        tx.begin();

        try {
            Member member = new Member();
            member.setId(101L);
            member.setName("HelloJPA");

            em.persist(member);

            Member findMember = em.find(Member.class, 101L);

            System.out.println("findMember.id = " + findMember.getId());
            System.out.println("findMember.name = " + findMember.getName());

            tx.commit();
        } catch (Exception e) {
            tx.rollback();
        } finally {
            em.close();
        }

        emf.close();
    }
}
```
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_PersistenceContext_4.jpg"></p>

- 다음 코드와 같이 101번에 엔티티가 저장된 상태에서 2번 조회하면 쿼리가 한 번 나온다.
    * 새로 실행하면 엔티티 매니저가 새로 생성되고 처음 조회 했을때는 1차 캐시에 없기 때문에 DB에서 가져오고 영속성 컨텍스트에 올려놓는다.
        - JPA는 엔티티를 조회하면 무조건 영속성 컨텍스트에 올린다.
    * 2번째 조회 때는 영속성 컨텍스트에 있는 1차 캐시부터 찾아서 반환한다. 
```java
public class JpaMain {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");

        EntityManager em = emf.createEntityManager();

        EntityTransaction tx = em.getTransaction();
        tx.begin();

        try {
            Member findMember1 = em.find(Member.class, 101L);
            Member findMember2 = em.find(Member.class, 101L);

            tx.commit();
        } catch (Exception e) {
            tx.rollback();
        } finally {
            em.close();
        }

        emf.close();
    }
}
```
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_PersistenceContext_5.jpg"></p>
<br>

### 영속 엔티티의 동일성 보장
- JPA는 자바 컬렉션처럼 영속 엔티티의 동일성을 보장해준다. (같은 트랜잭션 안에서)
    * == 비교해도 true가 나온다.
- 1차 캐시가 있기 때문에 가능하다.
    * 2번 조회해도 같은 객체로 래퍼런스가 같다.
```java
Member findMember1 = em.find(Member.class, "member1");
Member findMember2 = em.find(Member.class, "member1");

System.out.println("result = " + (findMember1 == findMember2)); // 동일성 비교 true
```
- 1차 캐시로 반복 가능한 읽기(REPEATABLE READ) 등급의 트랜잭션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공한다.
<br>

### 엔티티 등록 - 트랜잭션을 지원하는 쓰기 지연
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_PersistenceContext_6.jpg"></p>
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_PersistenceContext_7.jpg"></p>

- 영속성 컨텍스트 안에는 1차 캐시도 있지만 쓰기 지연 SQL 저장소라는 것도 있다.
- `em.persist(memberA);`과 `em.persist(memberB);`를 수행했을 때의 과정
    * memberA가 `persist()`에 의해 1차 캐시에 들어가면 동시에 JPA가 엔티티를 분석해서 INSERT 쿼리를 생성해서 쓰기 지연 SQL 저장소에 쌓아둔다.
    * memberB도 `persist()`에 의해 1차 캐시에 들어가면 또 INSERT 쿼리를 생성해서 쓰기 지연 SQL 저장소에 쌓아둔다.
    * 트랜잭션을 커밋하는 시점(`transaction.commit();`)에 쓰기 지연 SQL 저장소에 있던 쿼리들이 flush해서 DB로 날라간다
- persistence.xml에 `hibernate.jdbc.batch_size`라는 옵션을 통해서 사이즈 조절도 할 수 있다.
```java
EntityManager em = emf.createEntityManager();
EntityTransaction transaction = em.getTransaction();
// 엔티티 매니저는 데이터 변경시 트랜잭션을 시작해야 한다.
transaction.begin(); // 트랜잭션 시작

Member member1 = new Member(150L, "A");
Member member2 = new Member(160L, "B");

em.persist(member1);
em.persist(member2);
// 이때까지 INSERT SQL을 데이터베이스에 보내지 않는다.

// 커밋하는 순간 데이터베이스에 INSERT SQL을 보낸다.
transaction.commit();
```
<br>

### 엔티티 수정 - Dirty Checking(변경 감지)
- 엔티티 수정을 하고 나서 `persist()`나 `update()`로 다시 저장할 필요가 없다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_PersistenceContext_8.jpg"></p>

- DB 트랜잭션을 커밋하면 `flush()`가 호출되고 엔티티와 스냅샷을 비교한다.
    * 1차 캐시에는 PK인 @Id가 있고, Entity, 스냅샷이 있다.
    * 스냅샷은 값을 최초로 읽어온 시점의 최초 데이터를 저장한 것이다.
    * 엔티티와 스냅샷을 비교해서 변경사항이 있으면 UPDATE 쿼리를 사용해서 DB에 반영해서 커밋한다.
- [Dirty Checking 추가 정보](https://jojoldu.tistory.com/415)
<br>

### 엔티티 삭제
- 삭제 또한 위의 메커니즘과 같이 트랜잭션 commit 시점에 쿼리가 나간다.
```java
Member memberA = em.find(Member.class, "memberA");

em.remove(memberA); // 엔티티 삭제
```
<br>

### flush
- flush는 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영하는 것을 말한다.
- 영속성 컨텍스트의 변경사항과 DB를 맞추는 작업이라고 생각하면 된다.
    * 영속성 컨텍스트의 쿼리들을 DB에 날려주는 것.
- DB 트랜잭션이 `trascation.commit`하는 순간 flush가 자동으로 발생한다.
- flush가 발생하면 생기는 일
    * Dirty Checking(변경 감지)
    * 수정된 엔티티를 쓰기 지연 SQL 저장소에 등록
    * 쓰기 지연 SQL 저장소의 쿼리를 DB에 전송(등록, 수정, 삭제 쿼리)
    * flush가 발생한다고 트랜잭션이 발생하는 것은 아니고 flush로 보내고 commit이 일어난다.
- flush해도 1차 캐시가 지워지지는 않는다.
    * DB에 반영이 되는 과정이라고 보면 된다.
<br>

#### 영속성 컨텍스트를 flush하는 방법
- em.flush()로 직접 호출
```java
Member member = new Member(200L, "A");
em.persist(member);

// flush를 강제로 직접 호출
// DB에 쿼리가 즉시 나가고 그 다음에 트랜잭션이 commit된다.
em.flush();

System.out.println("Before Commit");

transaction.commit();
```
- 트랜잭션 commit - flush 자동 호출
- JPQL 쿼리 실행 - flush 자동 호출
```java
em.persist(memberA);
em.persist(memberB);
em.persist(memberB);
// 이 때 까지 DB에 안날라간다.

// 중간에 JPQL 실행
// 바로 모든 member를 조회하면 member A, B, C가 DB에서 조회가 되지 않는다.
// 그래서 이런 문제를 방지하기 위해 기본적으로 JPQL 쿼리를 실행할 때 무조건 flush 호출한다.
query = em.createQuery("select m from Member m", Member.class);
List<Member> members = query.getResultList();
```
<br>

#### flush 모드 옵션
- `em.setFlushMode(FlushModeType.COMMIT);`로 세팅하는데 직접 쓸 일은 거의 없다.
- `FlushModeType.AUTO`
    * 기본값
    * 커밋이나 쿼리를 실행할 때 flush
- `FlushModeType.COMMIT`
    * 커밋을 할 때만 flush
    * 쿼리를 실행할 때는 flush를 하지 않는다.
<br>

#### flush 정리
- flush는 영속성 컨텍스트를 비우는 것이 아니다.
- flush는 영속성 컨텍스트의 변경 내용을 데이터베이스에 동기화 하는 것이다.
- flush가 동작할 수 있는 이유는 데이터베이스 트랜잭션이라는 개념이 있기 때문이다.
- 어떻게 하든 트랜잭션 커밋되는 직전에만 동기화 해주면 되기 때문에, flush 매커니즘의 동작이 가능한 것이다.
- JPA는 기본적으로 데이터를 맞추거나 동시성에 관련된 것들은 데이터베이스 트랜잭션에 위임해서 사용한다. (참고)
<br>

### 준영속 상태
- `em.persist()`을 통해 영속성 컨텍스트에 저장해도 영속 상태이지만 `em.find()`조회를 할 때, 영속성 컨텍스트 1차 캐시에 없어서 DB에서 가져와서 1차 캐시에 저장한 상태도 영속 상태다.
- 준영속 상태는 영속 상태의 엔티티가 영속성 컨텍스트에서 분리(detached)되는 것을 말한다.
    * 이렇게 되면 영속성 컨텍스트가 제공하는 기능을 사용하지 못한다.
<br>

#### 준영속 상태로 만드는 방법
- 특정 엔티티를 준영속 상태로 전환
    * `em.detach(entity);`
```java
// em.find()로 가져올 때 영속성 컨텍스트에 올리기 때문에 영속 상태가 된다.
Member member = em.find(Member.class, 150L);
// 그래서 값을 변경하면 Dirty Checking해서 commit 시점에 쿼리를 날려준다.
member.setname("AAAAA");

// 영속성 컨텍스트에서 분리시켜 더이상 JPA에서 관리하지 않는다.
em.detach(member);

// 따라서 트랜잭션을 커밋할 때 아무일도 일어나지 않는다.
tx.commit();

//실행해도 SELECT 쿼리만 나가고 UPDATE 쿼리는 안나간다.
```
- 엔티티 매니저 안에 있는 영속성 컨텍스트를 초기화 
    * `em.clear();`
```java
// 영속성 컨텍스트에 1번 올라간다.
Member member = em.find(Member.class, 150L);
member.setname("AAAAA");

// 영속성 컨텍스트 초기화
em.clear();

// 다시 조회하면 영속성 컨텍스트에 없으므로 다시 올린다.
Member member = em.find(Member.class, 150L);

tx.commit();

// 따라서 SELECT 쿼리가 나온다. 
```
- 영속성 컨텍스트 종료
    * `em.close()`
    * 더이상 관리가 되지않는다. 
    * 데이터를 변경해도 변경되지 않는다.
<br>
