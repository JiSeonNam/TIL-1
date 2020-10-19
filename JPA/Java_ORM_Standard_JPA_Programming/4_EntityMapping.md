# 엔티티 맵핑
- 객체와 테이블 맵핑
    * `@Entity`, `@Table`
- 필드와 컬럼 맵핑
    * `@Column`
- 기본 키 맵핑
    * `@Id`
- 연관관계 맵핑
    * `@ManyToOne`, `@JoinColumn`
    * 관계가 있는 것들 끼리의 맵핑
<br>

## 객체와 테이블 맵핑

### @Entity
- `@Entity`가 붙은 클래스는 JPA가 관리하며 엔티티라고 한다.
- JPA를 사용해서 테이블과 맵핑할 클래스는 `@Entity` 필수
- 주의점
    * 기본 생성자가 꼭 있어야 한다.(파라미터가 없는 public 또는 protected 생성자)
    * final 클래스, enum, interface, inner 클래스는 사용하지 못한다.
    * 저장할 필드에 final을 사용할 수 없다.
- @Entity 속성
    * `@Entity(name = "Member")`
        - JPA에서 사용할 엔티티 이름 지정
        - 기본값은 클래스 이름을 그대로 사용
        - 다른 패키지에 같은 클래스 이름이 없으면, 가급적 기본값을 사용한다.
<br>

### @Table
- 엔티티와 맵핑할 테이블을 지정한다.
- `@Table(name = "MBR")`
- `INSERT INTO ... MBR`

| 속성                   | 기능                               | 기본값      |
| ---------------------- | ---------------------------------- | ----------- |
| name                   | 맵핑할 테이블 이름                 | 엔티티 이름 |
| catalog                | 데이터베이스 catalog 맵핑          |             |
| schema                 | 데이터베이스 schema 맵핑           |             |
| uniqueConstraints(DDL) | DDL 생성시에 유니크 제약 조건 생성 |             |

<br>

## 데이터베이스 스키마 자동 생성
- 엔티티의 맵핑 정보만 보면 어떤 테이블인지, 어떤 쿼리를 만들어야 되는지 알 수 있다. 
- 그래서 JPA에서는 아예 애플리케이션 로딩시점에 DB 테이블을 생성해주는 기능을 지원한다.
    * DDL을 애플리케이션 실행 시점에 자동 생성해준다.
    * 운영에서는 사용하면 안되고, 개발단계이나 로컬에서 개발할 때 도움이 된다.
    * 사용하더라도 생성된 DDL을 적절히 다듬은 후에 사용해야 한다.
- 이렇게 하면 테이블 만들어 놓고 객체를 개발하는게 아니라 객체를 만들고 객체 맵핑을 해놓으면 애플리케이션이 실행될 때 필요한 테이블을 만들어준다.
    * 테이블 중심 -> 객체 중심
- DB 방언을 활용해서 데이터베이스에 맞는 적절한 DDL문을 생성해준다.
<br>

### hibernate.hbm2ddl.auto 옵션
- persistence.xml 파일에서 옵션 설정을 할 수 있다.
- ex) `<property name="hibernate.hbm2ddl.auto" value="create" />`

| 옵션        | 설명                                                         |
| ----------- | ------------------------------------------------------------ |
| create      | 기존 테이블 삭제 후 다시 생성(DROP + CREATE)                 |
| create-drop | create와 같으나 종료시점에 테이블 DROP(테스트에서 사용하면 도움된다) |
| update      | 변경분만 반영(운영DB에는 사용하면 안됨). 추가하는 변경분만 반영, DROP X |
| validate    | 엔티티와 테이블이 정상 맵핑되었는지만 확인                   |
| none        | 사용하지 않음                                                |

<br>

### 데이터베이스 스키마 자동 생성 - 주의점
- **운영 장비에는 절대 create, create-drop, update를 사용하면 안된다.**
- 개발 초기 단계는 create 또는 update로 로컬과 개발 서버에서 쓰면 된다.
- 어느정도 개발 진행 후 테스트 서버는 update 또는 validate를 쓰면 된다.
    * 여러 명의 개발자가 같이 쓰는 테스트 서버에서는 create를 쓰면 안된다.
    * 테스트 했던 데이터 다 날아간다.
- 스테이징과 운영 서버는 validate 또는 none을 사용해야 한다.
- 왠만하면 그냥 안쓰는게 낫다.(직접 작성하자!)
- 웹 어플리케이션에서 사용하는 DB계정은 운영 장비에서 alter나, drop 못하도록 계정 자체를 분리하는 것이 좋다.
<br>

### DDL 생성 기능
- 제약 조건을 추가
    * ex) 회원 이름은 필수, 10자 초과 X
    * `@Column(nullable = false, length = 10)`
- 유니크 제약 조건 추가
    * `@Column(unique = true)`
- DDL 생성 기능은 DDL을 자동 생성할 때만 사용되고 JPA의 실행 로직에는 영향을 주지 않는다.
<br>

## 필드와 컬럼 맵핑
- 요구사항 예시
    * 회원은 일반 회원과 관리자로 구분해야 한다.
    * 회원 가입일과 수정일이 있어야 한다.
    * 회원을 설명할 수 있는 필드가 있어야 한다. 이 필드는 길이 제한이 없다.
```java
@Entity
public class Member {

    @Id
    private Long id;

    @Column(name = "name") // 객체는 username, DB에는 name으로 써야할 때
    private String username;

    private Integer age;

    @Enumerated(EnumType.STRING)
    private RoleType roleType;

    @Temporal(TemporalType.TIMESTAMP)
    private Date createdDate;

    @Temporal(TemporalType.TIMESTAMP)
    private Date lastModifiedDate;

    @Lob
    private String description;

    //..getter and setter
}
```
<br>

### 필드-컬럼 맵핑 어노테이션 정리

#### @Column
- name
    - 필드와 맵핑할 테이블의 컬럼 이름
    - 기본값 : 객체의 필드이름
- insertable, updatable
    - 컬럼의 등록, 변경을 할건지 하지 않을건지 정해줄 수 있다.
    - TRUE/FALSE로 설정하며 기본값은 TRUE이다.
- nullable(DDL)
    - null 허용여부 설정한다.
    - false로 설정하면 DDL 생성 시 not null 제약조건이 붙는다.
- unique(DDL)
    - `@Table`의 uniqueConstraints 속성과 같지만 한 컬럼에 간단히 유니크 제약조건을 걸 때 사용한다.
    - 실제로는 유니크 제약조건을 만들어주긴 하지만 컬럼명이 랜덤으로 생성되서 `@Table`의 uniqueConstraints 속성을 보통 사용한다.
    - @Table의 uniqueConstrains 옵션에 이름까지 줄 수 있다.
- columnDefinition(DDL)
    - 컬럼 정의를 직접 할 수 있다.
    - ex) `columnDefinition = "varchar(100) default 'EMPTY'"`
    - 정의한 문구가 DDL에 그대로 들어간다.
    - default는 자바 필드 타입과 DB 방언 정보를 사용해서 만든다.
- length(DDL)
    - 문의 길이 제약조건, String 타입에만 사용한다.
    - 기본값 : 255
- precision, scale(DDL)
    - 큰 숫자를 표현하는 자료형 BigDecimal 타입에서 사용한다.(BigInteger도 사용가능)
    - precision은 소수점을 포함한 전체 자릿수를 지정하는 옵션이고, scale은 소수점 자리수를 지정하는 옵션이다.
    - Double, float 타입에는 적용되지 않으며, 정밀한 소수를 다루어야 할 때 사용한다.
    - 기본값 :  precision = 19, scale = 20
<br>

#### @Enumerated
- 자바의 Enum 타입 맵핑을 지원한다.
- EnumType.STRING은 enum의 이름을 DB에 저장하고 EnumType.ORDINAL은 enum의 순서를 DB에 저장한다.
- 기본값인 ORDINAL로 설정하면 Enum 순서로 숫자가 맵핑되는데, Enum 중간에 필드가 하나 추가 되어 순서가 꼬이게 되면 매우 위험하다.
- EnumType을 반드시 EnumType.STRING으로 지정해야 한다.
<br>

#### @Temporal
- 날짜 타입(java.util.Date, java.util.Calendar)을 맵핑할 때 사용한다.
- 요즘에는 자바 8부터 LocalDate와 LocalDateTime이 들어와 최신 하이버네이트에서는 생략하고 그냥 쓰면 된다.
```java
private LocalDate testLocalDate;
private LocalDateTime testLocalDateTime;
```
<br>

#### @Lob
- DB에 varchar를 넘어서는 큰 내용을 넣고 싶은 경우에 사용한다.
- `@Lob`에는 지정할 수 있는 속성이 없다.
- 맵핑하는 필드 타입이 문자면 CLOB 맵핑, 나머지는 BLOB 맵핑
    * CLOB : String, char[], java.sql.CLOB
    * BLOB : byte[], java.sql.BLOB
<br>
 
#### @Transient
- `@Transient`애노테이션이 붙은 필드는 맵핑하지 않는다.
- 데이터베이스에 저장, 조회되지 않는다.
- 주로 메모리상에서만 임시로 관리하고 싶을 경우 사용한다.
<br>

## 기본 키 맵핑

### 기본 키 맵핑 어노테이션
- `@Id`
    * 직접 할당
- `@GeneratedValue`
    * 자동 생성
    * `@GenetratedValue(strategy = GenerationType.타입)`

### @GeneratedValue strategy
- 기본값은 AUTO로 DB 방언에 따라 IDENTITY, SEQUENCE, TABLE 중 자동으로 선택된다.
<br>

#### IDENTITY
- 기본 키 생성을 데이터베이스에 위임한다.
- 주로 MYSQL, PostgreSQL, SQL Server, DB2 등에서 사용한다.
    * ex) MYSQL : AUTO_INCREMNET
- IDENTITY 전략의 특징
    * JPA는 보통 트랜잭션 커밋 시점에 INSERT SQL을 실행한다.
    * 하지만 AUTO_INCREMENT는 DB에 INSERT SQL 실행 후에야 ID값을 알 수 있다.
    * 문제는 영속성 컨텍스트에 관리되려면 PK값이 있어야하는데 PK값을 DB에 들어가봐야 안다는 것이다.
    * 이 문제를 해결하기 위해 IDENTITY 전략에서만 예외적으로 `em.persist()`를 호출한 시점에 DB에 INSERT 쿼리를 날려버린다.
    * 결과적으로 쓰기 지연(버퍼링)을 사용할 수 없다. (사실 버퍼링하는게 큰 의미가 없어서 성능 차이가 비약적으로 나진 않는다.)

```java
@Entity
public class Member {
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    // ...
```
<br>

#### SEQUENCE 
- 데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 DB 오브젝트(Oracle 시퀀스)
- 주로 Oracle, PostgreSQL, DB2, H2 DB에서 사용한다.
- 기본적으로 하이버네이트가 만드는 기본 시퀀스 오브젝트를 쓰는데, 테이블마다 시퀀스를 따로 관리하고 싶으면 `@SequenceGenerator`를 사용하면 된다.
- SEQUENCE 전략의 특징
    * SEQUENCE 전략도 DB에서 만드는 시퀀스 오브젝트를 참조해야 한다.
        - 시퀀스 오브젝트도 DB에서 관리한다.
    * `em.persist()`호출 시점에(영속성 컨텍스트에 저장할 때) DB 시퀀스 오브젝트에서 next value를 호출해서 값을 얻어온 다음 member에 Id값을 넣어주고 영속성 컨텍스트에 저장한다.
        - 이 때까지 DB에 INSERT 쿼리는 날라가지 않는다.
    * PK값만 얻고 영속성 컨텍스트에 쌓여있다가 트랜잭션 커밋시점에 INSERT 쿼리가 호출된다. (버퍼링 사용 가능)
    * 이 방식의 경우 네트워크를 많이 왔다갔다 하기 때문에 성능이 나쁘다고 생각될 수 있지만 allocationSize로 해결 가능하다.
        - allocationSize만큼 next call 한번 할때 한꺼번에 DB에 올려놓고 메모리에 1씩 쓰는 것이다.
            * 쭉 사용하다가 시퀀스값이 51일 때 다시 한번 호출한다.
            * 이론적으로는 많이 쓰면 좋지만 적절히 값을 설정해서 쓰는게 좋다.
        - 기본값 : 50
        - 여러 웹 서버에서도 동시성 문제가 없다.

```java
@Entity
@SequenceGenerator(
    name = "MEMBER_SEQ_GENERATOR",
    sequesceName = "MEMBER_SEQ",  // 맵핑할 데이터베이스 시퀀스 이름
    initialValue = 1, allocationSize = 1)
public class Member {
    @Id
    @GenerateValue(strategy = GenerationType.SEQUESCE, 
                   generator = "MEMBER_SEQ_GENERATOR")
    private Long id;

    // ...
}
```
<br>

#### TABLE
- 키 생성 전용 테이블을 생성해서, 데이터베이스 시퀀스를 흉내내는 전략.
- 모든 데이터베이스에 적용 가능하지만 최적화가 되어있지 않은 테이블을 직접 사용해 성능이 떨어진다.
- TABLE 전략의 특징
    * SEQUENCE 전략의 allocationSize를 똑같이 사용가능하다.
```java
@Entity
@TableGenerator(
    name = "MEMBER_SEQ_GENERATOR",
    table = "MY_SEQUENCES", // 테이블 이름
    pkColumnValue = "MEMBER_SEQ", allocationSize = 1) // pkColumn 이름
public class Member {
    @Id
    @GenerateValue(strategy = GenerationType.TABLE, 
                   generator = "MEMBER_SEQ_GENERATOR")
    private Long id;

    // ...
}
```
<br>

### 권장하는 식별자 전략
- 기본 키 제약 조건 : null이면 안된다. 유일해야한다. 변하면 안된다.
- 미래까지 이 조건을 만족하는 자연키는 찾기 어렵다.
    * 대리키(대체키)를 사용하자.
        - 자연키 : 비즈니스적으로 의미있는 키 ex) 전화번호, 주민등록번호
        - 대리키 : 비즈니스와 관련없는 키 ex) GeneratedValue, 랜덤값
- 예를 들어, 주민등록번호도 기본 키로 적절하지 않다.
    * PK로 주민번호를 쓰면 FK로 주민번호가 퍼져있어서 주민번호를 쓰지 못할 때 다 바꾸려면 마이그레이션이 난리가 난다.
    * 만약 PK를 대체키로 썼으면 간단한 문제이다.
- 권장사항 : Long형 + 대체키 + 키 생성전략 사용
<br>
