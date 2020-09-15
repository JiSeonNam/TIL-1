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