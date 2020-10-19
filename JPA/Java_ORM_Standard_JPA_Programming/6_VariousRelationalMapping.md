# 다양한 연관관계 맵핑

## 연관관계 맵핑시 고려사항 3가지

### 다중성
- 다대일(N:1) - `@ManyToOne`
- 일대다(1:N) - `@OneToMany`
- 일대일(1:1) - `@OneToOne`
- 다대다(N:M) - `@ManyToMany`
    * 사실 다대다는 실무에서 쓰면 안된다.
- JPA가 4가지 어노테이션을 지원해준다.
    * 4가지 어노테이션들은 DB와 맵핑하기 위해 존재하기 때문에 DB 관점에서의 다중성을 기준으로 고민하면 된다.
- 다중성을 고민하다가 풀리지 않으면 대칭성을 고려하면 된다. 
    * 다중성의 관계들은 대칭성을 다 가지고 있다. 
    * 다대일 <-> 일대다, 일대일 <-> 일대일, 다대다 <-> 다대다
    * ex) Member와 Team, Team과 Member 둘 다 고려하면 쉬워진다.
<br>

### 단방향, 양방향
- 테이블
    * 외래키 하나로 양쪽 JOIN 가능하다.
    * 방향이라는 개념이 없다.
- 객체
    * 참조용 필드가 있는 쪽으로만 참조할 수 있다. 
    * 한 쪽만 참조하면 단방향이고 양쪽이 서로 참조하면 양방향이다.
        * 사실 양방향은 마주보는 단방향이 2개인 것이다
<br>

### 연관관계 주인
- 테이블은 외래키 하나로 두 테이블이 연관관계를 맺는다.
- 객체의 양방향 관계는 A -> B, B -> A 처럼 참조가 2군데에서 일어난다.
- 따라서 객체에서 둘 중 테이블의 외래키를 관리할 곳을 정해야 한다.
- 연관관계의 주인은 외래키를 관리하는 참조이다.
- 주인의 반대편에서는 외래키에 영향을 주지 않고, 단순 조회만 할 수 있다.
<br>

## 다대일[N:1]
- JPA에서 가장 많이 사용하고, 꼭 알야야 되는 다중성이다.
- DB 설계상 다 쪽에 외래키가 있어야 한다. 그렇지 않으면 잘못된 설계이다.
- 다대일 관계에서 다인 Member에서 단순하게 Team으로 단방향 참조를 하고 싶다고 할 때, 테이블에 외래키 있는 쪽인 Member객체에서 Team객체로 연관관계 맵핑을 하면 된다.
<br>

### 다대일 관계 단방향 맵핑
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_VariousRelationalMapping_1.jpg"></p>

- JPA에서 지원하는 `@ManyToOne` 어노테이션을 사용해서 다대일 관계를 맵핑한다.
- `@JoinColumn`은 외래키를 맵핑할 때 사용한다.
    * name속성은 맵핑할 외래키 이름이다.
```java
public class Member {
    // ...
  
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
  
    // ...
}
```
<br>

#### 다대일 단방향 정리
- 가장 많이 사용하는 연관관계이다.
- 다대일의 반대는 일대다
<br>

### 다대일 관계 양방향 맵핑
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_VariousRelationalMapping_2.jpg"></p>

- 다대일 관계에서 단방향 맵핑을 진행하고, 양방향 맵핑을 진행할 때 반대쪽에서 일대다 단방향 맵핑을 해주면 된다.
    * 객체에서 컬렉션만 추가해주면 된다.
- 반대에서 단방향 맵핑을 한다고 해서 DB 테이블에 영향을 전혀 주지 않는다.
- 다대일관계의 다 쪽에서 이미 연관관계의 주인이 되어서 외래키를 관리하고 있다.
- 반드시 mappedBy로 연관관계 주인을 설정해야 한다.
```java
public class Team {
    //...

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();
    
    //...
```
<br>

#### 다대일 양방향 정리
- 외래키가 있는 쪽이 연관관계의 주인이다.
- 양쪽을 서로 참조하도록 개발하면 된다.
<br>

## 일대다[1:N]
- 일대다 관계에서는 일이 연관관계의 주인이다.
    * 일대다 중 일 쪽에서 외래키를 관리하겠다는 의미이다.
- 표준 스펙에서 지원은 하지만 실무에서는 권장하지 않는다.
<br>

### 일대다 단방향 맵핑
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_VariousRelationalMapping_3.jpg"></p>

- Team은 Member를 참조하는데 Member는 Team을 참조하지 않아도 된다는 설계가 나올 순 있다.
    * 객체 입장에서는 충분히 나올 수 있는 설계이다.
- 그러나 DB입장에서는 무조건 일대다의 다쪽에 외래키가 들어가야한다.
- Team에서 members가 바뀌면, DB의 Member 테이블 즉, 다른 테이블의 업데이트 쿼리가 나가게 된다.
```java
@Entity
public class Team {
    // ...
    
    @OneToMany
    @JoinColumn(name = "TEAM_ID") // 일대다 중 일 쪽에서 연관관계 주인이 되어 외래키 관리
    private List<Member> members = new ArrayList<>();
    
    // ...
}
```
```java
// member 저장
Member member = new Member();
member.setUsername("member1");
em.persist(member);


// team 저장
Team team = new Team();
team.setName("teamA");
team.getMembers().add(member);
em.persist(team);


tx.commit();
```
- 실행하면 다음과 같이 member와 team INSERT 쿼리문이 나가고 트랜잭션 커밋 시점에 MEMBER 테이블을 UPDATE 쿼리가 한번 더 나간다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_VariousRelationalMapping_4.jpg"></p>

<br>

#### 일대다 단방향 정리
- 일대다 단방향은 일대다(1:N)의 일(1)이 연관관계의 주인이다.
- 테이블 일대다 관계는 항상 다(N) 쪽에 외래키가 있다.
- 객체와 테이블의 패러다임 차이 때문에 객체의 반대편 테이블의 외래키를 관리하는 특이한 구조이다.
- @JoinColumn을 꼭 사용해야 한다. 
    * 그렇지 않으면 자동으로 조인 테이블 방식을 사용한다.(중간에 테이블을 하나 추가한다.)
- 일대다 단방향 맵핑의 단점
    * 엔티티가 관리하는 외래키가 다른 테이블에 있다.
    * 연관관계 관리를 위해 추가로 UPDATE SQL이 실행된다.
- 따라서 다대일 단방향 관계로 맵핑하고, 필요할 경우 양방향 맵핑을 사용하는 것이 좋다.
- 객체 입장에서보면, 반대방향으로 참조할 필요가 없는데 관계를 하나 만드는 것이지만, DB의 입장으로 설계의 방향을 조금 더 맞춰서 운영상 유지보수하기 쉬운 쪽으로 선택할 수 있다.
<br>

### 일대다 양방향 맵핑
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_VariousRelationalMapping_5.jpg"></p>

- 스펙 상 가능한 것은 아니지만 억지로 하려면 가능하다.
- 아래와 같이 @ManyToOne과 @JoinColumn을 사용해서 연관관계를 맵핑하면, 다대일 단방향 맵핑이 되어버린다. 근데 반대쪽 Team에서 이미 일대다 단방향 맵핑이 설정되어있다. 이런 상황에서는 두 엔티티에서 모두 테이블의 FK 키를 관리 하게 되는 상황이 벌어진다.
- 그런 다음 insertable, updatable 옵션을 false로 설정해 읽기 전용 필드으로 만들어 버린다.
    * 맵핑도 되고 값도 쓰지만 최종적으로 DB에 UPDATE를 하지 않는다.(읽기 전용)
```java
@Entity
public class Member {
    // ...
    
    @ManyToOne
    @JoinColumn(name = "team_id", insertable = false, updatable = false)
    private Team team;
  
    // ...
}
```
<br>

#### 일대다 양방향 정리
- JPA 스펙 상 공식적으로 존재하진 않는다.
- `@JoinColumn(insertable=false, updatable=false)` 
    * 읽기 전용으로 만들 수 있다.
- 생각보다 맵핑하고 읽기 전용으로 쓰는 전략이 필요한 경우가 있다.
- **최대한 다대일 양방향을 사용하자**
- 설계라는 것은 테이블도 많고 누구나 사용할 수 있도록 단순해야 한다.
<br>

## 일대일[1:1]
- 일대일 관계는 반대도 일대일이다.
- 주 테이블이나 대상 테이블 중에 외래키를 선택 가능하다.
    * 여기서 주로 많이 Access하는 테이블을 주 테이블이라 한다.
    * 둘 중에 한 곳에 넣으면 된다.
    * 주 테이블에 외래키
    * 대상 테이블에 외래키
- 외래키에 데이터베이스 유니크(UNI) 제약조건이 추가되야 일대일 관계가 된다.
<br>

### 일대일 - 주 테이블에 외래키 단방향 맵핑
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_VariousRelationalMapping_6.jpg"></p>

- Member가 하나의 Locker를 가지고 반대로 Locker도 하나의 Member만 가질 수 있다고 비즈니스 룰을 가정할 때, 일대일 연관관계가 된다.
- MEMBER에 LOCKER_ID를 외래키로 가지고 UNI 제약조건을 걸어도 되고, 반대로 LOCKER에 MEMBER_ID를 외래키로 가지고 UNI 제약조건을 걸어도 된다.
```java
@Entity
public class Member {
    // ...
        
    @OneToOne
    @JoinColumn(name = "locker_id")
    private Locker locker;
 
    // ...
}
```
```java
@Entity
public class Locker {
    
    @Id @GeneratedValue
    private Loing id;

    private String name;
}
```
<br>

#### 일대일 - 주 테이블에 외래키 단방향 맵핑 정리
- 다대일(N:1) 단방향 관계 맵핑과 유사하다.
<br>

### 일대일 - 주 테이블에 외래키 양방향 맵핑
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_VariousRelationalMapping_7.jpg"></p>

- 주 테이블에 외래키 단방향 맵핑에서 Locker 클래스에 member를 추가하면 된다.
    * 읽기 전용이 된다.
    * mappedBy가 다대일과 똑같이 적용된다.
```java
@Entity
public class Member {
    // ...
        
    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;
 
    // ...
}
```
```java
@Entity
public class Locker {
    
    @Id @GeneratedValue
    private Loing id;

    private String name;

    // 양방향으로 설정
    @OneToOne(mappedBy = "locker") // member에 있는 locker
    private Member member;
}
```
<br>

#### 일대일 - 주 테이블에 외래키 양방향 맵핑 정리
- 다대일 양방향 맵핑처럼 외래키가 있는 곳이 연관관계의 주인이다.
- 반대편은 mappedBy를 적용해야 한다.
<br>

### 일대일 - 대상 테이블에 외래키 단방향 맵핑
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_VariousRelationalMapping_8.jpg"></p>

- 대일관계에서 대상 테이블에 외래키를 저장하는 단방향 관계는 JPA에서 지원도 방법도 없다.
    * Member에 있는 locker로 LOCKER테이블의 MEMBER_ID(FK)를 관리할 수 없다.
<br>

#### 일대일 - 대상 테이블에 외래키 단방향 맵핑 정리
- 단방향 관계는 JPA에서 지원하지 않는다.
- 양방향 관계는 지원한다.
<br>

### 일대일 - 대상 테이블에 외래키 단방향 맵핑
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_VariousRelationalMapping_9.jpg"></p>

- 일대일 주 테이블에 외래키 양방향 맵핑을 반대로 뒤집었다고 생각하면 된다.
- Locker의 member를 연관관계의 주인으로 잡고 LOCKER의 MEMBER_ID와 맵핑을 하면 된다.
    * Member를 읽기 전용으로 만들어야 한다.
<br>


#### 일대일 - 대상 테이블에 외래키 단방향 맵핑 정리
- 일대일 주 테이블에 외래키 양방향과 맵핑 방법이 같다.
<br>

### 일대일[1:1] 정리
- 주 테이블에 외래키
    * 주 객체가 대상 객체의 참조를 가지는 것 처럼 주 테이블에 외래키를 두고 대상 테이블을 찾는다.
    * 객체지향 개발자들이 선호한다.
    * JPA 맵핑이 편리하다.
    * 장점 : 주 테이블만 조회해도 대상 테이블에 데이터가 있는지 확인 가능하다.
    * 단점 : 값이 없으면 외래키에 null을 허용한다.
- 대상 테이블 외래키
    * 대상 테이블에 외래키가 존재한다.
    * 전통적인 데이터베이스 개발자들이 선호한다.
    * 장점 : 주 테이블과 대상 테이블을 일대일에서 일대다 관계로 변경할 때 테이블 구조 유지가 된다.
    * 단점 : 프록시 기능의 한계로 지연 로딩으로 설정해도 항상 즉시 로딩된다. 
        - JPA 입장에서 일대일 관계의 주 테이블(Member)에 외래키를 저장하는 상황에서는, Member를 로딩할 때 LOCKER_ID가 있는지 없는지만 판단하면 된다.
        - 있으면 프록시 객체를 넣어주고, 없으면 null을 넣으면 된다.
        - 그러나 문제는 대상 테이블에 외래키를 저장한다면, JPA가 Member의 locker를 조회하는 상황에서 DB의 MEMBER 테이블만 조회해서는 알 수 없다.
        - 어차피 LOCKER 테이블을 찾아서 MEMBER_ID에 있는지 없는지 확인해야(쿼리를 날려야) 알 수 있다
        - 어차피 쿼리가 나갈거면 프록시를 만들 필요가 없다.
        - 따라서 하이버네이트 구현체 같은 경우는 지연 로딩으로 설정해도 항상 즉시 로딩 된다.
<br>

## 다대다[N:M]
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_VariousRelationalMapping_10.jpg"></p>

- 실무에서는 사용하지 않는게 좋다.
- 관계형 데이터베이스는 정규화된 테이블 2개로 다대다 관계를 표현할 수 없다.
- 연결 테이블(조인 테이블)을 추가해서 일대다, 다대일 관계로 풀어내야 한다.
- 객체는 컬렉션을 사용해서 객체 2개로 다대다 관계가 가능하다.
    * ORM 입장에서는 객체는 되고 테이블은 안되는 것을 지원해줘야 한다.
    * 따라서, 아래의 그림에서와 같이 객체는 Member와 Product 서로의 list를 가질 수 있기 때문에
    * 그림의 테이블처럼 일대다, 다대일 맵핑을 연결 테이블을 넣어서 맵핑해준다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_VariousRelationalMapping_11.jpg"></p>

- `@ManyToMany` 어노테이션을 사용하고  `@JoinTable`로 연결 테이블을 지정해줄 수 있다.
<br>

### 다대다 단방향 맵핑
- 아래 코드와 같이 다대다 관계의 단방향 설정 후 실행하면 
    * MEMBER_PRODUCT라는 연결 테이블이 생기고 외래 키 제약조건도 두 가지가 설정 된다.
```java
@Entity
public class Member {
    //...
    
    @ManyToMany
    @JoinTable(name = "MEMBER_PRODUCT") // 연결테이블
    private List<Product> products = new ArrayList<>();
    
    //...
}
```
```java
@Entity
public class Product {

    @Id @GenerateValue
    private Long id;

    private String name;

    // ...getter and setter...
}
```
<br>

### 다대다 양방향 맵핑
- 단방향을 양방향으로 만들고 싶다면 반대에도 `@ManyToMany` 설정을 하고 mappedBy를 설정해야 한다.
```java
@Entity
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToMany(mappedBy = "products")
    private List<Member> members = new ArrayList<>();
}
```
<br>

### 다대다 맵핑의 한계
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_VariousRelationalMapping_12.jpg"></p>

- 편리해 보이지만 실무에서 사용할 수 없다.
- 연결 테이블이 단순히 연결만 하고 끝나지 않는다.
- 주문시간, 수량 같은 데이터가 들어올 수 있다. 
- 그러나 연결 테이블에 맵핑 정보만 넣는 것이 가능하고 추가 정보를 넣는게 불가능하다.
- 연결 테이블이 숨겨져 있기 떄문에 예상치 못한 쿼리들이 나와 매우 어렵다.
<br>

### 다대다 한계 극복
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_VariousRelationalMapping_13.jpg"></p>

- `@ManyToMany`를 `@OneToMany`와 `@ManyToOne`으로 바꾸고
- 연결 테이블용 엔티티를 추가한다. (연결 테이블을 엔티티로 승격)
<br>

#### 실습
- MemberProduct 엔티티를 만든다.
    * Member, Product 엔티티와 각각 다대일 관계로 맵핑
```java
@Entity
public class MemberProduct {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;

    @ManyToOne
    @JoinColumn(name = "PRODUCT_ID")
    private Product product;

    // ...getter and setter
}
```
- Member 엔티티를 MemberProducct 엔티티와 일대다 맵핑으로 변경
```java
@Entity
public class Member {
    //...
        
    @OneToMany(mappedBy = "member")
    private List<MemberProduct> memberProducts = new ArrayList<>();

    //...
}
```
- Product 엔티티를 MemberProducct 엔티티와 일대다 맵핑으로 변경
```java
@Entity
public class Product {
    //...

    @OneToMany(mappedBy = "product")
    private List<MemberProduct> members = new ArrayList<>();
    
    //...
}
```
- 위와 같이 코드를 작성하면 중간 테이블 엔티티로 승격하는 것이며 다대다를 일대다, 다대일로 풀어내는 것이다.
- 이렇게 하면 원하는 추가 정보를 MemberProduct에 추가할 수 있다.
    * 추가되면 Orders라던지 의미있는 엔티티로 변경해서 쓴다.
```java
@Entity
public class MemberProduct {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;

    @ManyToOne
    @JoinColumn(name = "PRODUCT_ID")
    private Product product;

    // 원하는 추가 정보들
    private int count;
    private int price;
    private LocalDateTime orderDateTime;

    // ...getter and setter
}
```
- 위의 다대다 맵핑의 한계 첨부 그림에서는 MemberProduct의 MEMBER_ID, PRODUCT_ID를 묶어서 PK로 썼지만, 실제로는 `@GeneratedValue`로 자동생성되는 id를 사용하는 것을 권장한다.
    * 왠만하면 PK는 의미없는 값을 써야한다.
- Id가 두 개의 테이블에 종속되지 않고 더 유연하게 사용할 수 있다.