# 값 타입

### 목차
- 기본값 타입
- **임베디드 타입(복합 값 타입)** : 중요
- 값 타입과 불변 객체
- 값 타입의 비교
- **값 타입 컬렉션** : 중요
<br>

### JPA의 데이터 타입 분류
- 엔티티 타입
    * @Entity로 정의하는 객체
    * 데이터가 변해도 식별자로 지속해서 추적 가능하다.
    * ex) 회원 엔티티의 키나 나이 값을 변경해도 식별자로 인식 가능하다.
- 값 타입
    * int, Integer, String처럼 단순히 값으로 사용하는 자바 기본 타입이나 객체
    * 식별자가 없고 값만 있으므로 변경시 추적 불가능하다.
    * ex) 숫자 100을 200으로 변경하면 완전히 다른 값으로 대체한다.
    * 분류
        - 기본값 타입
        - 임베디드 타입
        - 컬렉션 값 타입
<br>

## 기본값 타입
- 종류
    * 자바 기본 타입(int double)
    * 래퍼 클래스(Integer, Long)
    * String
- 생명 주기를 엔티티에 의존한다.
    * 회원을 삭제하면 이름, 나이 필드도 함께 삭제한다.
- 값 타입은 공유하면 절대 안된다.
    * 회원 이름 변경시 다른 회원의 이름도 함께 변경되면 안된다.(Side Effect가 발생 하면 안된다.)
- 참고
    * int, double과 같은 기본 타입은 절대 공유되지 않는다.
    * 기본 타입은 항상 값을 복사한다.
    * Integer같은 래퍼 클래스나 String같은 특수한 클래스는 공유 가능한 객체이지만, 변경은 할 수 없다.
<br>

## 임베디드 타입(embedded type, 복합 값 타입)
- 새로운 값 타입을 직접 정의할 수 있다.
- JPA는 임베디드 타입(embedded type)이라 한다.
- 주로 기본 값 타입을 모아서 만들기 때문에 복합 값 타입이라고도 한다.
- 주의할 점 : int, String과 같은 값 타입 이다. 엔티티 아니다. 추적이 안되고 변경하면 끝난다.
- 임베디드 타입의 값이 null이라면, 맵핑한 컬럼 값은 모두 null로 저장 된다.

### 예제로 이해하기
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_ValueType_1.jpg"></p>

- 회원 엔티티는 이름, 근무 시작일, 근무 종료일, 주소 도시, 주소 번지, 주소 우편번호를 가진다. 
    * 공통적인 속성들이 보인다.
    * 근무 시작일과 근무 종료일, 그리고 주소의 도시, 번지, 우편번호는 묶어서 시스템에서 공통으로 쓸 수 있다.
    * 회원 엔티티는 이름, 근무 기간, 집 주소를 가진다. 라고 묶어낼 수 있다.
    * 마지막 그림에서는 결국 클래스 2개를 새로 만든 것이다.
<br>

### 임베디드 타입 사용법
- `@Embeddable`
    * 값 타입을 정의하는 곳에 표시
- `@Embedded`
    * 값 타입을 사용하는 곳에 표시
- 기본 생성자가 필수다.
<br>

### 임베디드 타입의 장점
- 재사용이 가능하다
- 클래스 내의 응집도가 높다.
- `Period.isWork()`처럼 해당 값 타입만 사용하는 의미 있는 메소드를 만들 수 있다.
    * 객체지향적인 설계다.
- 임베디드 타입을 포함한 모든 값 타입은, 값 타입을 소유한 엔티티에 생명주기를 의존한다.
<br>

### 임베디드 타입과 테이블 맵핑
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_ValueType_2.jpg"></p>

- DB입장에서는 바뀔 것이 없다.
    * DB는 데이터를 잘 관리하는 것이 목적이기 때문에, 그림과 같이 설계되는 것이 맞다.
- 객체는 데이터뿐만 아니라 메소드라는 기능까지 다 가지고 있기 때문에, 공통 요소를 묶었을 때 가져갈 수 있는 이득이 많다.
- 임베디드 타입은 엔티티의 값일 뿐이다.
- 임베디드 타입을 사용하기 전과 후에 **맵핑하는 테이블은 같다.**
- 객체와 테이블을 아주 세밀하게(find-grained) 맵핑하는 것이 가능 하다.
- **잘 설계한 ORM 애플리케이션은 맵핑한 테이블의 수보다 클래스의 수가 더 많다**
<br>

### 맵핑 실습
- startDate와 endDate를 Period로, city, street, zipcode를 Address로 묶기
```java
@Entity
public class Member extends BaseEntity {
   
    // ...
    
    @Embedded
    private Period workPeriod;

    @Embedded
    private Address homeAddress;

    // ...
}
```
```java
@Embeddable
public class Period {

    private LocalDateTime startDate;
    private LocalDateTime endDate;

    //...getter and setter...
}
```
```java
@Embeddable
public class Address {

    private String city;
    private String street;
    private String zipcode;

    //...getter and setter...
}
```
<br>

### 임베디드 타입과 연관관계
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_ValueType_3.jpg"></p>

- Member 엔티티는 Address와 PhoneNumber라는 임베디드 값 타입을 가진다.
- Address는 Zipcode라는 임베디드 타입을 가지고 있다.(임베디드 타입이 임베디드 타입을 가질 수 있다.)
- PhoneNumber라는 임베디드 타입이 PhoneEntity라는 Entity를 가질 수 있다. 
    * id(FK)만 가지고 있으면 된다.
<br>

### @AttributeOverride : 속성 재정의
- 만약 한 엔티티에서 같은 값 타입을 사용한다면 어떻게 될까?
    * ex) Member에서 Address를 2개 가지고 있으면 어떻게 될까?
- 컬럼명이 중복되어 MappingException: Repeated column 에러가 발생한다.
```java
@Entity
public class Member {
    // ...
  
    @Embedded
    private Address homeAddress;
  
    @Embedded
    private Address workAddress;
  
    // ...
}
```
- 이 경우에 `@AttributeOverrides`를 사용하면 된다.
    * 중복되지 않고 하나면 `@AttributeOverride`를 사용한다.
    * 아래와 같이 새로운 컬럼에 저장하도록 컬럼 명 속성을 재정의 하면 된다.
```java
@Entity
public class Member {
    // ...

    @Embedded
    private Address homeAddress;
  
    @Embedded
        @AttributeOverrides({
        @AttributeOverride(name="city", column=@Column(name = "WORK_CITY"),
        @AttributeOverride(name="street", column=@Column(name = "WORK_street"),
        @AttributeOverride(name="zipcode", column=@Column(name = "WORK_ZIPCODE")            
    })
    private Address workAddress;

    //...
}
```
<br>

## 값 타입과 불변 객체
- 값 타입은 복잡한 객체 세상을 조금이라도 단순화하려고 만든 개념이다. 
- 따라서 값 타입은 단순하고 안전하게 다룰 수 있어야 한다.
<br>

### 값 타입 공유 참조
- 임베디드 타입 같은 값 타입을 여러 엔티티에서 공유하면 굉장히 위험하다.
- 부작용(side effect)이 발생한다.
- 다음 코드와 같이 작성하고 실행하면 멤버가 같은 주소 객체를 가지고 있기 때문에 member1, member2의 city가 모두 newCity로 바뀐다.
    * 실행하면 UPDATE 쿼리가 2번 나간다.
```java
Address address = new Address("city", "street", "10000");

Member member1 = new Member();
member1.setUsername("member1");
member1.setHomeAddress(address);
em.persist(member1);

Member member2 = new Member();
member2.setUsername("member2");
member2.setHomeAddress(address);
em.persist(member2);

// member1의 city만 newCity로 변경하지만 member1, 2 둘다 변경된다.
member1.getHomeAddress().setCity("newCity");

tx.commit();
```
- 이러한 부작용(side effect)는 매우 잡기가 어렵다.
- 만약 엔티티 간에 공유하고 싶은 값은 엔티티로 만들어서 공유해야한다.
<br>

### 값 타입 복사
- 값 타입 공유 참조의 부작용처럼 값 타입의 실제 인스턴스의 값을 공유하는 것은 매우 위험하다.
- 따라서 값(인스턴스)을 복사해서 사용해야 한다.
```java
Address address = new Address("city", "street", "10000");

Member member1 = new Member();
member1.setUsername("member1");
member1.setHomeAddress(address);
em.persist(member1);

// 주소 객체 복사
Address copyAddress = new Address(address.getCity(), address.getStreet(), address.getZipcode());

Member member2 = new Member();
member2.setUsername("member2");  
member2.setHomeAddress(copyAddress);
em.persist(member2);

// member2에는 city값이 변하지 않는다.
member1.getHomeAddress().setCity("newCity");

tx.commit();
```
<br>

### 객체 타입의 한계
- 항상 값을 복사해서 사용하면 공유 참조로 인해 발생하는 부작용을 피할 수 있다.
- 문제는, 임베디드 타입처럼 직접 정의한 값 타입은 자바의 기본타입이 아니라 객체 타입이다.
- 자바의 기본 타입에 값을 대입하면 자바는 값을 복사한다.
- 객체 타입은 참조 값을 직접 대입하는 것을 박을 방법이 없다.
    * 위의 코드에서 copyAdress를 실수로 address를 넣을 경우 컴파일 단계에서 잡을 수가 없다.
- 따라서, 객체의 공유 참조는 피할 수 없다
- 참고
    * 기본 타입의 값 복사
    ```java
    int a = 10;
    int b = a; // 기본 타입은 값을 복사
    b = 4; // b의 값을 바꿔도 a값은 유지가 된다.
    ```
    * 객체 타입의 값 복사
    ```java
    Address a = new Address("Old");
    Address b = a; // 객체 타입은 참조를 전달한다. 같은 인스턴스를 가르킨다.
    b.setCity("New"); // a, b 주소 객체의 필드가 변경된다.
    ```
<br>

### 불변 객체
- 객체 타입을 수정할 수 없게 만들면 부작용을 원천 차단할 수 있다.
- 값 타입은 불변 객체(immutable object)로 설계해야 한다.
    * 불변 객체 : 생성 시점 이후 절대 값을 변경할 수 없는 객체.
    * Integer, String은 자바가 제공하는 대표적인 불변 객체이다.
- 생성자로만 값을 설정하고 Setter를 만들지 않으면 된다.
    * 예제에서 setter를 모두 제거하면 `setCity()`자체가 불가능하다.
    * setter를 private으로 만들어도 된다.
- 불변이라는 작은 제약으로 부작용이라는 큰 재앙을 막을 수 있다.
- 실제로 값을 바꾸면 싶다면?
    * 객체를 새로 만든다.
```java
Address address = new Address("city", "street", "10000");

Member member1 = new Member();
member1.setUsername("member1");
member1.setHomeAddress(address);
em.persist(member1);

Address newAddress = new Address("new City", address.getStreet(), address.getZipcode());
member.setHomeAddress(newAddress);

tx.commit();
```
<br>

## 값 타입의 비교
- 값 타입은 인스턴스가 달라도 그 안의 값이 같으면 같은 것으로 봐야 한다.
```java
int a = 10;
int b = 10;
// a == b : true
```
- 객체 타입은 값이 같아도 주소가 달라서 == 비교하면 false로 나온다.
    * 인스턴스 자체가 달라서 참조값이 다르다.
```java
Address a = new Address("서울시");
Address b = new Address("서울시");
// a == b : false
```
- 동일성(identity) 비교
    * 인스턴스의 참조 값을 비교한다.
    * `==`을 사용한다.
- 동등성(equivalence) 비교
    * 인스턴스의 값을 비교한다.
    * `equals()`를 사용한다.
- 값 타입은 `a.equals(b)`를 사용해서 동등성 비교를 해야 한다.
- 값 타입은 `equals()` 메소드를 적절하게 재정의해야 한다.
    * 주로 모든 필드를 다 재정의 해야 한다.
    * `equals()`를 재정의하면 `hashCode()`도 재정의 해야한다.
<br>

## 값 타입 컬렉션
- 값 타입을 컬렉션에 담아서 쓰는 것을 말한다.
- 값 타입을 하나 이상 저장할 때 사용한다.
- `@ElementCollection`, `@CollectionTable` 어노테이션을 사용해서 맵핑한다.
- 데이터베이스는 컬렉션을 같은 테이블에 저장할 수 없기 때문에 컬렉션을 저장하기 위한 별도의 테이블이 필요하다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_ValueType_3.jpg"></p>

- 예를 들어 Member가 그림과 같은 값 타입을 가지고 있다고 가정하자.
- 그런데 DB테이블로 구현할 때 문제가 발생한다.
    * 단순하게 값 타입이 하나이면 Member에 필드 속성으로 넣으면 되지만
    * 관계형 DB는 내부적으로 컬렉션을 담을 수 있는 구조가 없다.
    * 요즘에야 JSON을 지원하면서 되는 DB가 있긴 하지만 기본적으로는 안된다.
- 따라서 Member 입장에서는 컬렉션이 일대다 관계이기 때문에 위의 그림과 같이 별도의 테이블로 만들어야 한다.

#### 코드 실습
- Member 클래스에 값 타입 컬렉션 추가
    * `@ElmentCollection`으로 맵핑해준다. 
    * `@CollectionTable`으로 테이블 명을 지정해준다.

```java
@Entity
public class Member extends BaseEntity {

    @Id @GeneratedValue
    private Long id;

    // ...

    @ElementCollection
    @CollectionTable(
        name = "FAVORITE_FOODS", // 테이블 명을 지정
        joinColumns = @JoinColumn(name = "MEMBER_ID")) // 외래키 설정
    @Column(name = "FOOD_NAME") // 값이 하나이므로 예외적으로 컬럼명 지정
    private Set<String> favoriteFoods = new HashSet<>();

    @ElementCollection
    @CollectionTable(
        name = "ADDRESS",
        joinColumns = @JoinColumn(name = "MEMBER_ID"))
    private List<Address> addressHistory = new ArrayList<>();

    // ...
}
```
<br>

### 컬렉션 값 타입 저장
- 다음 코드와 같이 컬렉션에 값을 저장한다.
```java
Member member = new Member();
member.setUsername("member1");
member.setHomeAddress(new Address("homeCity", "street", "10000"));

member.getFavoriteFoods().add("치킨");
member.getFavoriteFoods().add("족발");
member.getFavoriteFoods().add("피자");

member.getAddressHistory().add(new Address("old1", "street1", "10001"));
member.getAddressHistory().add(new Address("old2", "street2", "10002"));

em.persist(member);

tx.commit();
```
- `em.persist()`한번에 Insert 쿼리가 나간다.
- 흥미로운 것은 값 타입 컬렉션을 따로 persist하지 않아도 Member만 저장하니까 자동으로 INSERT 쿼리가 나갔다.
    * 값 타입이기 때문이 그렇다.
    * Member에 소속된 값 타입들은 생명주기가 Member를 따라간다.
    * 별도로 persist하거나 update할 필요없이 자동으로 변경된다.
    * 일대다 연관관계에서 Cascade ALL로 설정하고, orphanRemoval true 설정한 것과 비슷하다.
    * 값 타입 컬렉션은 영속성 전이(Cascade) + 고아 객체 제거 기능을 필수로 가진다고 볼 수 있다.
<br>

### 컬렉션 값 타입 조회
- 다음 코드와 같이 멤버를 조회하면 Member와 Member에 소속된 Embedded 타입의 homeAddress만 조회되고 컬렉션들은 조회 되지 않는다.
- 컬렉션 값 타입들은 지연로딩이다.
```java
Member member = new Member();
member.setUsername("member1");
member.setHomeAddress(new Address("homeCity", "street", "10000"));

member.getFavoriteFoods().add("치킨");
member.getFavoriteFoods().add("족발");
member.getFavoriteFoods().add("피자");

member.getAddressHistory().add(new Address("old1", "street1", "10001"));
member.getAddressHistory().add(new Address("old2", "street2", "10002"));

em.persist(member);

// 영속성 컨텍스트를 비워 놓고. 깔끔한 상태가 된다.
em.flush();
em.clear();

System.out.println("============== START ==============");
Member finMember = em.find(Member.class, member.getId());
```
- 직접 컬렉션 값 타입을 사용하면 쿼리가 나간다.
```java
System.out.println("============== START ==============");
Member finMember = em.find(Member.class, member.getId());

List<Address> addressHistory = findMember.getAddressHistory();
for (Address address : addressHistory) {
    System.out.println("address = " + address.getCity());
}

Set<String> favoriteFoods = findMember.getFavoriteFoods();
for (String favoriteFood : favoriteFoods) {
    System.out.println("favoriteFood = " + favoriteFood);
}

tx.commit();
```
<br>

### 컬렉션 값 타입 수정
- 값 타입 수정
    * 값 타입은 Immutable 해야 한다.
    * 따라서 homeAddress를 수정하려면 Address 인스턴스 자체를 new로 갈아 끼워야 한다.
```java
// ...

Member findMember = em.find(Member.class, member.getId());

// findMember.getHomeAddress().setCity("newCity"); 이렇게 하면 안된다. 부작용 발생
findMember.setHomeAddress(new Address("newCity", "newStreet", "20001"));

tx.commit();
```
- 값 타입 컬렉션 수정
    * 컬렉션에 값을 지운 다음 새로 컬렉션에 추가한다.
    * 코드만 변경했을 뿐인데 JPA가 알아서 테이블에 DELETE, INSERT 해준다.
```java
Member findMember = em.find(Member.class, member.getId());

// 치킨 -> 한식 수정하고 싶은 경우
findMember.getFavoriteFoods().remove("치킨");
findMember.getFavoriteFoods().add("한식");

tx.commit();
```
- 값 타입 컬렉션 수정 2 - Address 객체
    * 대부분의 컬렉션들은 대상을 찾을 때 `equals()`를 사용하기 때문에 `equals()`, `hashCode()` 재정의가 중요하다.
    * 수정한 뒤 실행해보면 테이블을 통으로 날린 뒤에 새롭게 Address 2개를 새로 INSERT 하는 것을 볼 수 있다. (값 타입 컬렉션의 제약사항 참조)
```java
Member findMember = em.find(Member.class, member.getId());

// 대부분의 컬렉션들은 대상을 찾을 때 equals()를 사용하기 때문에 완전히 똑같은 객체를 넣어준다.
findMember.getAddressHistory().remove(new Address("old1", "street1", "10001"));
findMember.getAddressHistory().add(new Address("new city1", "street1", "10001"));

tx.commit();
```
<br>

### 값 타입 컬렉션의 제약사항
- 값 타입은 엔티티와 다르게 식별자 개념이 없다.
- 값은 변경하면 추척이 어렵다.
- **값 타입 컬렉션에 변경 사항이 발생하면, 주인 엔티티와 연관된 모든 데이터를 삭제하고, 값 타입 컬렉션에 있는 현재 값을 모두 다시 저장한다.**
    * 참고) `@OrderColumn`을 사용해서 어떻게 해결이 되긴 하지만 복잡하고 위험하다. 따라서 다른 방법으로 풀어내는 것이 낫다.(값 타입 컬렉션 대안 참고)
- 값 타입 컬렉션을 매핑하는 테이블은 모든 컬럼을 묶어서 기본 키를 구성해야 한다.
    * NULL 입력 X, 중복 저장X
<br>

### 값 타입 컬렉션 대안
- 실무에서는 상황에 따라 값 타입 컬렉션 대신에 일대다 관계를 고려하는 것이 낫다.
- 일대다 관계를 위한 엔티티를 만들고, 여기에서 값 타입을 사용하는 것이다.
- 영속성 전이 + 고아 객체 제거를 사용해서 값 타입 컬렉션 처럼 사용하자.
- 실무에서 쿼리 최적화도 훨씬 유리하다.
<br>

#### 실습
- AddressEntity 생성
```java
@Entity
@Table(name = "ADDRESS")
public class AddressEntity {

    @Id @GeneratedValue
    private Long id;

    private Address address;

    public AddressEntity() {
    }

    public AddressEntity(String city, String street, String zipcode) {
        this.address = new Address(city, street, zipcode);
    }
    
    // ...getter and setter...
}
```
- Member 수정
    * 값 타입이 아닌 엔티티 타입으로 맵핑
    * 일대다 단방향 or 양방향 맵핑
```java
@Entity
public class Member extends BaseEntity {

    @Id @GeneratedValue
    private Long id;

    // ...getter and setter...

    @ElementCollection
    @CollectionTable(
        name = "FAVORITE_FOOD",
        joinColumns = @JoinColumn(name = "MEMBER_ID"))
    @Column(name = "FOOD_NAME")
    private Set<String> favoriteFoods = new HashSet<>();

    // @ElementCollection
    // @CollectionTable(
    //     name = "ADDRESS",
    //     joinColumns = @JoinColumn(name = "MEMBER_ID"))
    // private List<Address> addressHistory = new ArrayList<>();

    // 값 타입으로 맵핑하는 것이 아니라 엔티티로 맵핑
    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true) 
    @JoinColumn(name = "MEMBER_ID") // 일대다 단방향 맵핑
    private List<AddressEntity> addressHistory = new ArrayList<>();

    // ...
}
```
- 이제 ADDRESS 테이블에 ID라는 개념이 생겼다. 그리고 Member ID를 FK로 가지고 있다.
    * 엔티티라는 얘기고, 이제는 특정 row를 찾아서 내부의 값을 수정할 수 있다.
- `@OneToMany`와 `@JoinColumn`으로 일대다 단방향 매핑을 했으므로, Member 테이블의 반대인 AddressEntity(Address 테이블)에 Update 쿼리가 나가는 것은 어쩔 수 없다.
- 이런식으로 값 타입 컬렉션은 엔티티로 승격 시켜서 많이 사용한다.
    * 정말 단순하다면 값 타입을 써도 된다.
<br>

### 정리
- 엔티티 타입의 특징
    * 식별자(ID)가 있다.
    * 생명 주기가 관리 된다.
    * 공유 할 수 있다.
- 값 타입의 특징
    * 식별자가 없다.
    * 생명 주기를 엔티티에 의존한다.
    * 공유하지 않는 것이 안전하다.(복사해서 사용)
    * 불변 객체로 만드는 것이 안전하다.
- 값 타입은 정말 값 타입이라 판단될 때만 사용해야 한다.
- 엔티티와 값 타입을 혼동해서 엔티티를 값 타입으로 만들면 안된다.
- 식별자가 필요하고, 지속해서 값을 추적, 변경해야 한다면 그것은 값 타입이 아닌 엔티티다.