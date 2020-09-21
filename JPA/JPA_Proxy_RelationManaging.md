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
