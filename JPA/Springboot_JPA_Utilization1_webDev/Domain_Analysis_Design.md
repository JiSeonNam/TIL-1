# 도메인 분석 설계

## 요구사항 분석
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/Domain_Analysis_Design_1.jpg"></p>

- 회원 기능
    * 회원 등록
    * 회원 조회
- 상품 기능
    * 상품 등록
    * 상품 수정
    * 상품 조회
- 주문기능
    * 상품 주문
    * 주문 내역 조회
    * 주문 취소
- 기타 요구사항
    * 상품은 재고 관리가 필요하다.
    * 상품의 종류는 도서, 음반, 영화가 있다.
    * 상품을 카테고리로 구분할 수 있다.
    * 상품 주문 시 배송 정보를 입력할 수 있다.
<br>

## 도메인 모델과 테이블 설계
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/Domain_Analysis_Design_2.jpg"></p>

- 회원은 여러 상품을 주문할 수 있기 때문에 1:N 관계다.
    * 회원 : 주문 = 1 : N
- 회원이 한 번 주문할 때 여러 상품을 선택할 수 있고 상품도 여러 주문에 담길 수 있기 때문에 N:M관계이다.
    * 주문 상품이라는 테이블을 만들어 각각 1:N, N:1 관계로 풀어낸다.
- 상품은 도서, 음반, 영화로 구분되어 상품이라는 공통 속성을 사용하므로 상속 구조로 표현했다.
<br>

### 회원 엔티티 분석
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/Domain_Analysis_Design_3.jpg"></p>

- 회원(Member)
    * 이름과 임베디드 타입인 주소(Address), 주문리스트(orders)를 가진다.
- 주문(Order)
    * 한 번 주문할 때 여러 상품을 주문할 수 있으므로 주문상품(OrderItem)은 1:N 관계다.
    * 상품을 주문한 회원과 배송 정보, 주문 날짜, 주문 상태를 가지고 있다.
- 주문상품(OrderItem)
    * N:M 관계를 1:N, N:1 관계를 풀어내는 역할 뿐만 아니라
    * 주문한 상품 정보와 주문 금액(orderPrice), 주문 수량(count) 정보를 가지고 있다.
- 배송(Delivery)
    * 주문과 배송지 주소, 주문 상태를 가지고 있다.
    * 주문 시 하나의 배송 정보를 생성하며 주문과 배송은 1:1 관계다.
- 상품(Item)   
    * 상품 이름, 가격, 재고수량, 어느 카테고리에 맵핑되어 있는지 카테고리 정보를 가지고 있다.
    * 상속 관계로 Album, Book, Movie를 가진다.
- 카테고리(Category)
    * 카테고리와 아이템은 N:M 관계다.
- 참고
    * 회원이 주문을 하기 때문에, 회원이 주문리스트를 가지는 것은 얼핏보면 잘 설계한것 같지만, 객체 세상은 실제 세계와는 다르다. 
    * 실무에서는 회원이 주문을 참조하지않고, 주문이 회원을 참조하는것으로 충분하다. 
    * 실습에서는 1:N, N:1의 양방향 연관관계를 설명하기 위해서 추가했다.
    * 카테고리와 아이템 관계 또한 실무에서는 N:M 관계를 쓰면 안된다.
<br>

### 회원 테이블 분석
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/Domain_Analysis_Design_4.jpg"></p>

- MEMBER
    * 거의 1:1 맵핑이다.
    * Address 임베디드 타입 정보가 MEMBER 테이블에 그대로 들어갔다.(DELIVERY 테이블도 마찬가지)
- ITEM
    * 앨범, 도서, 영화 타입을 통합해서 하나의 테이블로 만들었다. (싱글 테이블 전략을 썼다.)
    * DTYPE 컬럼으로 타입을 구분한다.
- ORDERS
    * DB에 order by가 예약어로 잡고 있는 경우가 많아 관례상 ORDERS로 사용
- ORDER_ITEM
    * 어느 주문인지, 어느 아이템인지 확인할 FK, ORDER_ID, ITEM_ID가 있다.
- CATEGORY
    * 객체에서는 카테고리와 아이템이 서로의 객체를 가져도 된다.
    * 그러나 RDB에서는 일반적으로 불가능하다. 따라서 중간에 맵핑 테이블 CATEGORY_ITEM을 둬서 1:N, N:1 관계로 풀어낸다.
<br>

### 연관관계 맵핑 분석
- 회원과 주문
    * 회원 입장에서 1:N, 주문입장에서 N:1의 양방향 연관관계이다.
    * 따라서 연관관계의 주인을 정해야 하는데, 외래키가 있는 주문을 연관관계의 주인으로 정하는 것이 좋다.
        - 테이블에서 1:N 관계에서 항상 N 쪽에 외래키가 있다.
    * Order.member를 ORDERS.MEMBER_ID(FK)와 맵핑한다.
- 주문상품과 주문
    * N:1 양방향 관계이다.
    * 외래키가 주문상품에 있으므로 주문상품이 연관관계의 주인이다.
    * OrderItem.order를 ORDER_ITEM.ORFER_ID(FK)와 맵핑한다.
- 주문상품과 상품
    * N:1 단방향 관계이다.
    * OrderItem.item을 ORDER_ITEM_ID(FK)와 맵핑한다.
- 주문과 배송
    * 1:1 단방향 관계이다.
    * Order.delivery를 ORDERS.DELIVERY_ID(FK)와 맵핑한다.
<br>

## 엔티티 클래스 개발
- Member 엔티티 생성
```java
@Entity
@Getter @Setter
public class Member {

    @Id @GeneratedValue
    @Column(name = "member_id") // PK 이름 지정
    private Long id;

    private String name;

    @Embedded // 임베디드 타입 표시
    private Address address;

    @OneToMany(mappedBy = "member") // Member 입장에서 1:N 맵핑, 연관관계 주인 반대편이다.
    private List<Order> orders = new ArrayList<>();
}
```
- Address 클래스 생성
    * 임베디드 타입
    * 값 타입은 기본적으로 변경되면 안된다.(변경 불가능하게 설계해야 한다.)
        - `@Setter`대신 생성자를 둬서 생성할 때만 값이 세팅되게 하는 것이 좋다. 
    * reflection이나 프록시 기술을 쓸 수 있도록 기본 생성자를 만들어 준다.
        - public으로 하면 많이 호출되어 JPA에서는 Protected까지 허용해준다.(안전)
```java
@Embeddable // 임베디드 타입 표시
@Getter
public class Address {

    private String city;
    private String street;
    private String zipcode;
}
```
- Order 엔티티 생성
    * Member 엔티티와 N:1 맵핑
    * OrderItem 엔티티와 1:N 맵핑
    * 1:1 관계의 경우 FK를 엔티티에 둬도 된다.
        - 여기서는 Order에 FK를 놓고 연관관계 주인을 Order로 정한다.
```java
@Entity
@Table(name = "orders")
@Getter @Setter
public class Order {

    @Id @GeneratedValue
    @Column(name = "order_id")
    private Long id;

    @ManyToOne(fetch = LAZY)
    @JoinColumn(name = "member_id") // 맵핑을 member_id로 한다.
    private Member member;

    @OneToMany(mappedBy = "order")
    private List<OrderItem> orderItems = new ArrayList<>();

    @OneToOne(fetch = LAZY)
    @JoinColumn(name = "delivery_id")
    private Delivery delivery;

    private LocalDateTime orderDate;

    private OrderStatus status; // 주문상태 [ORDER, CANCEL]
}
```
- OrderStatus Enum 생성
```java
public enum OrderStatus {
    ORDER, CANCEL
}
```
- OrderItem 엔티티 생성
```java
@Entity
public class OrderItem {

    @Id @GeneratedValue
    @Column(name = "order_item_id")
    private Long id;

    @ManyToOne(fetch = LAZY)
    @JoinColumn(name = "item_id")
    private Item item;

    @ManyToOne(fetch = LAZY)
    @JoinColumn(name = "order_id")
    private Order order;

    private int orderPirce; // 주문 가격
    private int count; // 주문 수량
}
```
- Item 엔티티 생성
    * `@Inheritance(strategy = InheritanceType.SINGLE_TABLE)`
        - 상속 관계 맵핑이기 때문에 상속 관계 전략을 부모 클래스에 설정
        - SINGLE_TABLE 전략 선택
    * `@DiscriminatorColumn(name = "dtype")`
        - 타입을 구분하기 위한 dtype 컬럼
```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "dtype")
@Getter @Setter
public abstract class Item {

    @Id @GeneratedValue
    @Column(name = "item_id")
    private Long id;

    private String name;
    private int price;
    private int stockQuantity;

    @ManyToMany(mappedBy = "items")
    private List<Category> categories = new ArrayList<>();

}
```
- Item의 자손인 Book, Album, Movie 엔티티 생성
    * `@DiscriminatorValue("B")`
        - 타입 구분을 위한 애노테이션, 기본값은 클래스명
```java
@Entity
@DiscriminatorValue("B")
@Getter @Setter
public class Book extends Item {

    private String author;
    private String isbn;
}
```
```java
@Entity
@DiscriminatorValue("A")
@Getter @Setter
public class Album extends Item {

    private String artist;
    private String etc;
}
```
```java
@Entity
@DiscriminatorValue("M")
@Getter @Setter
public class Movie extends Item {

    private String director;
    private String isbn;
}
```
- Delivery 엔티티 생성
    * `@Enumerated(EnumType.STRING)`
        -  ORDINAL로 설정하면 Enum 순서로 숫자가 맵핑되는데, Enum 중간에 필드가 하나 추가 되어 순서가 꼬이게 되면 매우 위험하다.
        - 반드시 EnumType.STRING으로 설정해야 한다.
```java
@Entity
@Getter @Setter
public class Delivery {

    @Id @GeneratedValue
    @Column(name = "delivery_id")
    private Long id;

    @OneToOne(mappedBy = "delivery", fetch = LAZY)
    private Order order;

    @Embedded // 내장 타입이기 때문에 @Embedded 어노테이션 사용
    private Address address;

    @Enumerated(EnumType.STRING)
    private DeliveryStatus status; // READY, COMP
}
```
- DeliveryStatus enum 생성
```java
public enum DeliveryStatus {
    READY, COMP
}
```
- Category 엔티티 생성
```java
@Entity
@Getter @Setter
public class Category {

    @Id @GeneratedValue
    @Column(name = "category_id")
    private Long id;

    private String name;

    @ManyToMany
    @JoinTable(name = "category_item",
            joinColumns = @JoinColumn(name = "category_id"),
            inverseJoinColumns = @JoinColumn(name = "item_id"))
    private List<Item> items = new ArrayList<>();

    @ManyToOne(fetch = LAZY)
    @JoinColumn(name = "parent_id")
    private Category parent;

    @OneToMany(mappedBy = "parent")
    private List<Category> child = new ArrayList<>();

}
```
<br>

## 엔티티 설계시 주의점

### 엔티티에는 가급적 Setter를 사용하지 말자
- Setter가 열려있으면 변경 포인트가 너무 많아진다.
- 유지보수가 어렵다.
- 리펙토링으로 Setter를 제거하는 것이 좋다.
<br>

### 모든 연관관계는 지연로딩으로 설정해야 한다.
- 즉시로딩(EAGER)은 예측이 어렵고, 어떤 SQL이 실행될지 추적하기 어렵다.
    * JPQL을 실행할 때 N+1 문제가 자주 발생한다.
- 실무에서는 모든 연관관계를 지연로딩(LAZY)으로 설정해야 한다.
- 연관된 엔티티를 함께 DB에서 조회해야 하면, fetch join 또는 엔티티 그래프 기능을 사용한다.
- `@OneToOne`, `@ManyToOne` 관계는 기본이 즉시로딩이므로 직접 지연로딩으로 설정해야 한다.
<br>

### 컬렉션은 필드에서 초기화 하자.
- 컬렉션은 필드에서 바로 초기화하는 것이 안전하다. 
- null 문제에서 안전하다. 
- 하이버네이트는 엔티티를 영속화(persist)할 때 컬랙션을 감싸서 하이버네이트가 제공하는 내장 컬렉션으로 변경한다. 
    * 만약 `getOrders()`처럼 임의의 메서드에서 컬력션을 잘못 생성하면 하이버네이트 내부 메커니즘에 문제가 발생할 수 있다. 
    * 따라서 필드 레벨에서 생성하는 것이 가장 안전하고, 코드도 간결하다.
    * 컬렉션을 생성하면 변경하지 않는 것이 안전하다.
```java
Member member = new Member(); 
System.out.println(member.getOrders().getClass()); 
em.persist(team); 
System.out.println(member.getOrders().getClass());

//출력 결과 
class java.util.ArrayList 
class org.hibernate.collection.internal.PersistentBag // 하이버네이트가 추적할 수 있는 내장 컬렉션으로 변경한다.
```
<br>

### 테이블, 컬럼명 생성 전략
- 스프링 부트에서 하이버네이트 기본 맵핑 전략을 변경해서 실제 테이블 필드명은 다르다.
- 하이버네이트 기본 맵핑 전략 : 엔티티의 필드명을 그대로 테이블 명으로 사용한다.
    * SpringPhysicalNamingStrategy
- 스프링 부트 신규 설정
    * 엔티티(필드) -> 테이블(컬럼)
    * 1. 카멜 케이스 -> 언터스코어(memberPoint -> member_point)
    * 2. .(점) -> _(언더스코어)
    * 3. 대문자 -> 소문자
- [참고 링크 1](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#howto-configure-hibernate-naming-strategy)
- [참고 링크 2](http://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#naming)
<br>

### 양방향 맵핑에서는 연관관계 편의 메소드를 작성하는 것이 좋다.
- 값을 2번 넣어야 하기 때문에 휴먼에러가 생길 가능성이 많다.
- 따라서 연관관계 편의 메소드를 생성하는 것을 권장한다.
- 메소드 설정을 하면 team.getMembers().add(member);로 역방향 값을 다시 넣을 필요가 없다.
- 참고로 setter와 헷갈리지 않게 changeTeam()으로 함수명을 변경하는 것이 더 좋다.(개인적 취향)
- 편의 메소드가 양쪽에 있으면 문제가 생길 가능성이 높으므로 한쪽의 기준을 정하는 것이 좋다.
- 실습 코드의 양방향 맵핑이 있는 곳에 편의 메소드 추가
```java
@Entity
@Table(name = "orders")
@Getter @Setter
public class Order {

    @Id @GeneratedValue
    @Column(name = "order_id")
    private Long id;

    @ManyToOne(fetch = LAZY)
    @JoinColumn(name = "member_id") // 맵핑을 member_id로 한다.
    private Member member;

    @OneToMany(mappedBy = "order", cascade = ALL)
    private List<OrderItem> orderItems = new ArrayList<>();

    @OneToOne(fetch = LAZY, cascade = ALL)
    @JoinColumn(name = "delivery_id")
    private Delivery delivery;

    private LocalDateTime orderDate;

    @Enumerated(EnumType.STRING)
    private OrderStatus status; // 주문상태 [ORDER, CANCEL]

    // 연관관계 편의 메소드 추가
    public void setMember(Member member) {
        this.member = member;
        member.getOrders().add(this);
    }

    public void addOrderItem(OrderItem orderItem) {
        orderItems.add(orderItem);
        orderItem.setOrder(this);
    }

    public void setDelivery(Delivery delivery) {
        this.delivery = delivery;
        delivery.setOrder(this);
    }

}
```
```java
@Entity
@Getter @Setter
public class Category {

    @Id @GeneratedValue
    @Column(name = "category_id")
    private Long id;

    private String name;

    @ManyToMany
    @JoinTable(name = "category_item",
            joinColumns = @JoinColumn(name = "category_id"),
            inverseJoinColumns = @JoinColumn(name = "item_id"))
    private List<Item> items = new ArrayList<>();

    @ManyToOne(fetch = LAZY)
    @JoinColumn(name = "parent_id")
    private Category parent;

    @OneToMany(mappedBy = "parent")
    private List<Category> child = new ArrayList<>();

    // ==연관관계 편의 메소드==
    public void addChildCategory(Category child) {
        this.child.add(child);
        child.setParent(this);
    }

}
```
<br>
