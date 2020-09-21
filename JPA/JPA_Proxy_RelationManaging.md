# 프록시와 연관관계 관리

## 프록시
- 프록시의 개념에 앞서 Member 엔티티를 조회할 때 Team도 함께 조회해야 할까?
    * 비즈니스 로직에 따라 다르다. 
    * Team이 필요한 경우가 있고 필요없는 경우가 있다.
    * 굳이 항상 Team을 가져올 필요가 없기 때문에 JPA에서는 지연로딩과 프록시라는 개념을 사용한다.
<br>

### 프록시 기초
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_Proxy_RelationManaging_1.jpg"></p>

- JPA에서는 `em.find()` 말고도 `em.getReference()`라는 메소드도 제공 된다.
    * 이름 그대로 참조를 가져온다.
- `em.find()`는 DB를 통해서 실제 엔티티 객체를 조회하는 메소드이고
- `em.getReference()`는 DB의 조회를 미루는 가짜(프록시) 엔티티 객체를 조회하는 메소드이다.
    * DB에 쿼리가 날라가지 않았는데 조회가 된다.
- 하이버네이트가 내부 라이브러리를 사용해 프록시라고 하는 가짜 엔티티를 준다.
```java
Member member = new Member();
member.setUsername("hello");

em.persist(member);

em.flush();
em.clear();

//em.find()로 멤버를 조회하면 DB에 쿼리가 바로 나간다.
//Member findMember = em.find(Member.class, member.getId());

//em.getReference()로 멤버를 조회하면 실제로 필요한 시점에 데이터베이스에 쿼리가 나간다.
Member findMember = em.getReference(Member.class, member.getId());

System.out.println("findMember = " + findMember.getClass()); // 객체를 확인하면 Member객체가 아니라 HibernateProxy 객체인 것을 볼 수 있다.
System.out.println("findMember.id = " + findMember.getId()); // findMember를 찾을때 id값을 넣었으므로 바로 가져올 수 있다.  
System.out.println("findMember.username = " + findMember.getUsername()); // username은 DB에 있으므로 DB에 쿼리를 날린다.(프록시 객체 초기화 참고)

tx.commit();
```
<br>

### 프록시의 특징
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_Proxy_RelationManaging_2.jpg"></p>

- 실제 클래스를 상속 받아서 만들어지기 때문에 실제 클래스와 겉모양이 같다.
    * 하이버네이트가 내부적으로 라이브러리를 사용해 만든다.
- 사용하는 입장에서 진짜 객체인지 프록시 객체인지 구분하지 않고 사용하면 된다.(이론상)
- 프록시 객체는 실제 객체의 참조(target)를 보관한다.
- 프록시 객체를 호출하면 프록시 객체는 실제 객체의 메소드를 호출해준다.
<br>

### 프록시 객체의 초기화
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_Proxy_RelationManaging_3.jpg"></p>

```java
Member member = em.getReference(Member.class, member.getId());
member.getName();
```
- `em.getReference()`로 프록시 객체를 가져온 다음에, `getName()`메소드를 호출하면
- target에 값이 없으면 JPA가 영속성 컨텍스트에 진짜 객체를 가져오라고 요청하고
- 영속성 컨텍스트는 DB를 조회해서 실제 Entity 객체를 생성해준다.
- 프록시 객체가 가지고 있는 target에 실제 Entity 객체를 연결해준다.
- 그래서 `getName()` 했을 때 target에 연결되어 있는 진짜 `getName()` 통해서 반환된다.
- target에 한번 초기화되면 프록시 객체의 초기화 동작은 없어도 된다.
<br>

### 프록시 정리
- 프록시 객체는 처음 사용할 때 한 번만 초기화된다.
- 프록시 객체를 초기화 할 때, 프록시 객체가 실제로 엔티티로 바뀌는 것은 아니다.
    * 초기화 되면 프록시 객체를 통해서 실제 엔티티에 접근 가능한 것이다.
    * 교체의 의미보다는 채워지는 의미
- 프록시 객체는 원본 엔티티를 상속 받는다. 따라서 타입 체크시 주의해야 한다.
    * 프록시 객체와 원본 객체가 타입이 다르다.
    * == 비교 대신 instance of를 사용해야 한다.
```java
Member member1 = new Member();
member.setUsername("member1");
em.persist(member1);

Member member2 = new Member();
member.setUsername("member2");
em.persist(member2);

Member member3 = new Member();
member.setUsername("member3");
em.persist(member3);

em.flush();
em.clear();

Member m1 = em.find(Member.class, member1.getId());
Member m2 = em.find(Member.class, member2.getId());
Member m3 = em.getReference(Member.class, member2.getId());

System.out.println("m1 == m2 : " + (m1.getClass() == m2.getClass())); // true
System.out.println("m1 == m2 : " + (m1.getClass() == m3.getClass())); // false
System.out.println("m1 == m2 : " + (m3 instanceof Member); // true

tx.commit();
```
- 영속성 컨텍스트에 찾는 엔티티가 이미 있으면, `em.getReference()`를 호출해도 실제 엔티티를 반환한다. 반대도 똑같다.
    * JPA는 하나의 영속성 컨텍스트에서 조회하는 같은 엔티티의 동일성을 보장한다.
    * 만약 둘 다 `getReference()`로 가져오면 둘 다 프록시 객체이다.
    * 만약 `getReference()`로 먼저 가져오고, `find()`로 실제 객체를 조회하면 둘 다 프록시 객체가 반환된다.
        -  엔티티의 동일성을 보장 하기 위해서 같도록 맞춰버린다.
```java
Member member1 = new Member();
member.setUsername("member1");
em.persist(member1);

em.flush();
em.clear();

Member m1 = em.find(Member.class, member1.getId());
System.out.println("m1 = " + m1.getClass()); // Member 클래스

// 이미 영속성 컨텍스트에 올라가 있기 때문에 실제 엔티티를 반환한다. 
Member reference = em.getReference(Member.class, member1.getId());
System.out.println("reference = " + reference.getClass()); // Member 클래스

System.out.println("m1 == reference : " + (find == reference)); // true
```
- 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태일 때
    * 초기화 문제가 발생한다.
    * 실무에서 많이 발생하는 문제이다.
```java
Member member = new Member();
member.setUsername("member1");
em.persist(member1);

em.flush();
em.clear();

Member reference = em.getReference(Member.class, member1.getId());

// 준영속 상태로 만들기
em.detach(reference); // em.close, em.clear도 동일


System.out.println("refMemer = " + reference.getUsername());

tx.commit();
```
<br>

### 프록시 확인 관련 메소드
- 프록시 인스턴스의 초기화 여부 확인
    * `PersistenceUnitUtil.isLoaded(Object entity);`
```java
boolean isLoaded = emf.getPersistenceUnitUtil().isLoaded(referenceMember);
```
- 프록시 클래스 확인
    * `entity.getClass().getName()` 출력
- 프록시 강제 초기화
    * `org.hibernate.Hibernate.initialize(entity);`
```java
Hibernate.initialize(referenceMamber);
```
<br>

## 즉시로딩과 지연로딩

### 지연로딩 LAZY를 사용해서 프록시로 조회
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_Proxy_RelationManaging_4.jpg"></p>

- 그림과 같이 지연로딩을 사용하면 로딩되는 시점에 Lazy 로딩 설정이 되어있는 Team 엔티티는 프록시 객체로 가져온다.
    * Member를 가져올 때는 Member만 가져오고 Team은 프록시로 가져온다.
    * Member를 가져올 때 대부분 Team이 필요없을 때 주로 사용한다.
- 실제 객체를 사용하는 시점에(Team을 사용하는 시점에) 초기화되고 DB에 쿼리가 나간다.
```java
@Entity
public class Member extends BaseEntity {

    @Id @GeneratedValue
	private Long id ;

	@Column (name = "USERNAME")
	private String name;

	@ManyToOne (fetch = FetchType.LAZY) // 지연로딩
	@JoinColumn (name = "TEAM_ID")
	private Team team;

    // ...
}
```
```java
Team team = new Team();
team.setName("teamA");
em.persist(team);

Member member = new Member();
member.setUsername("member1");
em.persist(member1);

em.flush();
em.clear();

Member m = em.find(Member.class, member.getId());

System.out.println("m = " + m.getTeam().getClass()); // 프록시 객체가 출력된다.

tx.commit();
```
<br>

### 즉시로딩 EAGER를 사용해 함께 조회
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_Proxy_RelationManaging_5.jpg"></p>

- Member와 Team을 자주 함께 사용해야 할 때 사용한다.
- Member와 Team을 JOIN해서 한번에 가져온다.
    * 둘 다 프록시가 아니라 실제 객체를 가져온다.
- 
```java
@Entity
public class Member extends BaseEntity {

    @Id @GeneratedValue
	private Long id ;

	@Column (name = "USERNAME")
	private String name;

	@ManyToOne (fetch = FetchType.EAGER) // 즉시로딩
	@JoinColumn (name = "TEAM_ID")
	private Team team;

    // ...
}
```
<br>

### 프록시와 즉시로딩 주의점
- 실무에서 가급적이면 지연로딩만 사용해야 한다.
- 즉시로딩을 사용하면 예상치 못한 SQL이 발생한다.
    * 실무에서 테이블이 많을 경우에 JOIN을 많이 해야하고 쿼리도 엄청나게 나올 수 있다.
- 즉시로딩은 JPQL에서 N+1 문제를 일으킨다.
    * `em.find()`는 PK를 정해놓고 DB에서 가져오기 때문에 JPA 내부에서 최적화를 할 수 있다.
    * 그러나 JPQL에선 입력 받은 query string이 그대로 SQL로 변환된다.
    * JPA 내부에서 최적화를 하지 않기 때문에 변환된 SQL에서 필요한 테이블에 대한 쿼리를 날려서 가져온다.
    * N+1 문제는 쿼리를 1개 날렸는데, 그것 때문에 추가 쿼리가 N개 나간다는 의미이다.
- 그렇다면 실무에서 지연로딩을 사용해서 항상 쿼리를 2번씩 날려서 조회를 해야할까?
    * 이런 경우를 위해 JPQL의 fetch join을 이용하면 해당 시점에 한번에 쿼리로 가져와서 사용할 수 있다.
    * 어노테이션이나 배치 사이즈 설정으로 해결하는 방법도 있다.
- `@ManyToOne`, `@OneToOne`과 같이 `@XXXToOne` 어노테이션들은 기본이 즉시로딩(EAGER) 이다.
    * 반드시 지연(LAZY)로 변경해서 사용하는 것이 좋다.
- `@OneToMany`와 `@ManyToMany`는 기본이 지연로딩(LAZY)이다.
<br>

### 지연로딩 활용
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_Proxy_RelationManaging_5.jpg"></p>

- Member와 Team을 자주 함께 사용한다 -> 즉시로딩
- Member와 Order는 가끔 사용한다 -> 지연로딩
- Order와 Product는 자주 함께 사용한다 -> 즉시로딩
- 이론적인 내용으로 실무에서는 다 지연로딩으로 사용하면 된다.
    * 즉시로딩 사용하지 말자!
    * JPQL fetch join이나, 엔티티 그래프 기능으로 해결하자.
    * 즉시로딩은 상상하지 못한 쿼리가 나간다.
<br>


## 영속성 전이 : CASCADE
- 영속성 전이는 즉시로딩, 지연로딩이나 연관관계 설정과 전혀 관계가 없다.
- 특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶을 때 사용한다.
    * ex) 부모 엔티티를 저장할 때 자식 엔티티도 함께 저장하고 싶은 상황
<br>

### 코드로 이해하기
- 다대일 관계인 Child와 Parent를 만들고, 양방향 연관관계 맵핑
```java
@Entity
public class Parent {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "parent")
    private List<Child> children = new ArrayList<>();

    //...getter and setter...
}
```
```java
@Entity
public class Child {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "parent_id")
    private Parent parent;

    // 연관관계 편의 메소드 추가
    public void addChild(Child child) {
        this.children.add(child);
        child.setParent(this);
    }

    //...getter and setter...
}
```
-  `em.persist()`을 3번 호출해야 한다.
```java
Child child1 = new Child();
Child child2 = new Child();

Parent parent = new Parent();
parent.addChild(child1);
parent.addChild(child2);

em.persist(parent);
em.persist(child1);
em.persist(child2);

tx.commit();
```
- 이 중복을 없애기 위해 CASCADE 기능을 사용할 수 있다.
    * Parent가 Child를 관리한다.
    * Parent를 persist할 때 Child가 자동으로 persist된다.
    * 코드를 짤 때 Parent 중심으로 짤 수 있다.
```java
@Entity
public class Parent {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL) //cascade 추가
    private List<Child> children = new ArrayList<>();

    public void addChild(Child child) {
        this.children.add(child);
        child.setParent(this);
    }
}
```
- 다음 코드를 실행하면 정상적으로 parent만 persist해도 child 2개도 persist되어 쿼리가 3개 나간다.
```java
Child child1 = new Child();
Child child2 = new Child();

Parent parent = new Parent();
parent.addChild(child1);
parent.addChild(child2);

// parent만 persist해도 child 2개도 persist 된다.
em.persist(parent);

tx.commit();
```
<br>

### CASCADE 주의 사항
- 영속성 전이는 연관관계를 맵핑하는 것과 아무 관련이 없다.
- 엔티티를 영속화할 때 연관된 엔티티도 함께 영속화하는 편리함을 제공할 뿐이다.
- 일대다의 경우 항상 써야할까? 아니다.
    * 하나의 부모가 자식들을 관리할 때 의미가 있다.
        - 게시판(부모)에서 첨부파일의 경로(자식들)를 관리할 때
    * 다른 엔티티들과 연관 관계가 있으면 사용하면 안된다.
    * 소유자가 하나일 때, 즉 단일 엔티티에 완전히 종속적일 때 사용하는 것이 좋다.
<br>

### CASCADE의 종류
- **ALL** : 모두 적용(모든 라이프 사이클을 맞춰야 할 때, 삭제가 위험할 때)
- **PERSIST** - 영속(저장할 때만 사용)
- **REMOVE** - 삭제
- MERGE
- REFRESH
- DETACH
<br>

## 고아 객체
- 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제하는 기능이다.
- orpahnRemoval 옵션으로 설정한다.
- 다음 코드에서 컬렉션에서 빠진 엔티티는 자동으로 삭제된다.
```java
@Entity
public class Parent {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orpahnRemoval = true ) // 고아 객체 기능 사용
    private List<Child> children = new ArrayList<>();

    public void addChild(Child child) {
        this.children.add(child);
        child.setParent(this);
    }
}
```
- 자식 엔티티를 컬렉션에서 제거하면 자동으로 DELETE 쿼리가 나간다.
```java
Child child1 = new Child();
Child child2 = new Child();

Parent parent = new Parent();
parent.addChild(child1);
parent.addChild(child2);

em.persist(parent);

em.flush();
em.clear();

Parent findParent = em.find(Parent.class, parent.getId());
findParent.getChildren().remove(0); // 자식 엔티티를 컬렉션에서 제거

tx.commit();
```
<br>

### 고아 객체 주의점
- 참조가 제거된 엔티티는 다른 곳에서 참조하지 않는 고아 객체로 보고 삭제하는 기능이다.
- 참조하는 곳이 하나일 때 사용해야한다.
- 특정 엔티티가 개인 소유할 때 사용해야 한다.
- `@OneToOne`, `@OneToMany`에서만 사용 가능하다.
- 개념적으로 부모를 제거하면 자식은 고아가 된다.
    * 고아 객체 제거 기능을 활성화하면, 부모를 제거할 때 자식도 함께 제거 된다.
    * `CascadeType.REMOVE`처럼 동작한다.
    * 따라서 매우 조심히 사용해야 한다.
<br>

### 영속성 전이와 고아 객체를 같이 사용했을 때의 생명주기
- 스스로 생명주기를 관리하는 엔티티는 `em.persist()`로 영속화하고 `em.remove()`로 제거할 수 있다.
- 두 옵션을 모두 활성화 하면 부모 엔티티를 통해서 자식의 생명주기를 관리할 수 있다.
    * 부모에 의해 생성되고 제거되기 때문이다.
- 도메인 주도 설계(DDD)의 Aggregate Root개념을 구현할 때 유용하다.