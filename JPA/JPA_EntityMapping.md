# 엔티티 맵핑
- 객체와 테이블 맵핑
    * `@Entity`, `@Table`
- 필드와 컬럼 매핑
    * `@Column`
- 기본 키 매핑
    * `@Id`
- 연관관계 매핑
    * `@ManyToOne`, `@JoinColumn`
    * 관계가 있는 것들 끼리의 맵핑
<br>

## 객체와 테이블 맵핑

### @Entity
- `@Entity`가 붙은 클래스는 JPA가 관리하며 엔티티라고 한다.
- JPA를 사용해서 테이블과 매핑할 클래스는 `@Entity` 필수
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
- 엔티티와 매핑할 테이블을 지정한다.
- `@Table(name = "MBR")`
- `INSERT INTO ... MBR`

| 속성                   | 기능                               | 기본값      |
| ---------------------- | ---------------------------------- | ----------- |
| name                   | 매핑할 테이블 이름                 | 엔티티 이름 |
| catalog                | 데이터베이스 catalog 매핑          |             |
| schema                 | 데이터베이스 schema 매핑           |             |
| uniqueConstraints(DDL) | DDL 생성시에 유니크 제약 조건 생성 |             |

<br>

## 데이터베이스 스키마 자동 생성
- 엔티티의 매핑 정보만 보면 어떤 테이블인지, 어떤 쿼리를 만들어야 되는지 알 수 있다. 
- 그래서 JPA에서는 아예 애플리케이션 로딩시점에 DB 테이블을 생성해주는 기능을 지원한다.
    * DDL을 애플리케이션 실행 시점에 자동 생성해준다.
    * 운영에서는 사용하면 안되고, 개발단계이나 로컬에서 개발할 때 도움이 된다.
    * 사용하더라도 생성된 DDL을 적절히 다듬은 후에 사용해야 한다.
- 이렇게 하면 테이블 만들어 놓고 객체를 개발하는게 아니라 객체를 만들고 객체 맵핑을 해놓으면 애플리케이션이 실행될 때 필요한 테이블을 만들어준다.
    * 데이블 중심 -> 객체 중심
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
| validate    | 엔티티와 테이블이 정상 매핑되었는지만 확인                   |
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

### 필드-컬럼 매핑 어노테이션 정리

#### @Column
- name
    - 필드와 매핑할 테이블의 컬럼 이름
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
- 자바의 Enum 타입 매핑을 지원한다.
- EnumType.STRING은 enum의 이름을 DB에 저장하고 EnumType.ORDINAL은 enum의 순서를 DB에 저장한다.
- 기본값인 ORDINAL로 설정하면 Enum 순서로 숫자가 매핑되는데, Enum 중간에 필드가 하나 추가 되어 순서가 꼬이게 되면 매우 위험하다.
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
- `@Transient`애노테이션이 붙은 필드는 매핑하지 않는다.
- 데이터베이스에 저장, 조회되지 않는다.
- 주로 메모리상에서만 임시로 관리하고 싶을 경우 사용한다.
<br>
