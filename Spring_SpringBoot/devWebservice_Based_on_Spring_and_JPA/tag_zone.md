# 관심 주제와 지역 정보
- 주요 기능
    * 관심 주제 등록
    * 관심 주제 삭제
    * 지역 정보 데이터 초기화
    * 지역 정보 등록 
    * 지역 정보 삭제
<br>

## 관심 주제 도메인
- 관심 주제(tag)는 엔티티로 만든다.
    * tag로 특정 데이터를 검색도 하고 다른 곳(study)에서 참조도 할 것이기 때문
- 객체 관점에서의 관계
    * Account -> Tag
    * ManyToMany
    * Account에서 Tag를 참조(단방향)
- DB 관점에서의 관계
    * Account <- Account_Tag -> Tag
    * 조인 (join) 테이블을 사용해서 다대다 관계를 표현.
    * Account_Tag에서 Account의 PK 참조.
    * Account_Tag에서 Tag의 PK 참조.
<br>

### 구현
- Tag 엔티티 생성
```java
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
@Builder @AllArgsConstructor @NoArgsConstructor
public class Tag {

    @Id
    @GeneratedValue
    private Long id;

    @Column(unique = true, nullable = false)
    private String title;
}
```
- Account 엔티티에 연관관계 맵핑
```java
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
@Builder @AllArgsConstructor @NoArgsConstructor
public class Account {

    ...

    @ManyToMany
    private Set<Tag> tags;

    ...
}
```
- DB 관련 설정
```properties
# 개발할 때에만 create-drop 또는 update를 사용하고 운영 환경에서는 validate를 사용합니다.
spring.jpa.hibernate.ddl-auto=create-drop

# 개발시 SQL 로깅을 하여 어떤 값으로 어떤 SQL이 실행되는지 확인합니다.
spring.jpa.properties.hibernate.format_sql=true
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```
- [Tagify](https://github.com/yairEO/tagify)를 사용해 tag기능 구현
    * `npm install @yaireo/tagify`
    * [예제](https://yaireo.github.io/tagify/)
<br>
