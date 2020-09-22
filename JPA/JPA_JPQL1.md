# 객체지향 쿼리 언어 - 기본 문법
- JPA는 다향한 쿼리 방법을 지원한다.
    * **JPQL**
    * JPA Criteria
    * **QuertDSL**
    * 네이티브 SQL
    * JDBC API 직접 사용, MyBatis, SpringJdbcTemplate 함께 사용
    * 참고) 보통 JPQL + QueryDSL을 조합해서 사용하고(95%) 나머지 풀리지않는 쿼리(5%)는 SpringJdbcTemplate사용한다.
<br>

## 소개 

### JPQL 소개
- 가장 단순한 조회 방법이다.
    * `Entitymanager.find()`
    * 객체 그래프 탐색(`a.getB().getC()`)
- 만약 나이가 18살 이상인 회원을 모두 검색하고 싶다면?
<br>

### JPQL
- JPA를 사용하면 엔티티 객체를 중심으로 개발해야 한다. 
- 문제는 검색 쿼리이다.
- 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색을 해야한다.
- 그러나 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능하다.
- 애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL이 필요하다.
- 그래서 JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어를 제공한다.
- SQL과 문법이 유사하고, SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN등을 지원한다.
    * 결국 JPQL을 짜면 SQL로 번역되어 실행된다.
- JPQL은 엔티티 객체를 대상으로 쿼리하고 SQL은 데이터베이스 테이블을 대상으로 쿼리한다.
- 테이블이 아닌 객체를 대상으로 검색하는 객체지향 쿼리라고 이해하면 되며,
- SQL을 추상화해서 특정 데이터베이스 SQL에 의존하지 않는다.
- JPQL을 한마디로 정의하면 객체 지향 SQL이다.
<br>

#### 예시 코드
- 다음과 같이 JPQL을 작성하여 실행하면
```java
List<Member> result = em.createQuery("select m From Member m where m.username like '%kim%'", Member.class).getResultList();

for( Member member : result) {
    System.out.prinln("member = " + member);
}
```
- 다음과 같이 주석으로 JPQL이 보이고 실제 SQL로 번역되어 실행되는 것을 볼 수 있다.
    * 엔티티를 대상으로 쿼리를 한 것이고 엔티티 맵핑 정보를 읽어서 적절한 SQL을 만들어 낸다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_JPQL1_1.jpg"></p>

<br>

### Criteria 소개
- JPA 표준 스펙에 들어가 있는 기술이다.
- 동적 쿼리를 자바 코드로 짤 수 있게 해준다.
    * JPQL은 동적 쿼리를 짜기 어렵다.
- 부분 동적 쿼리를 떼어 내는 것이 편리 하다.
- 컴파일 타임에 오류를 발견할 수 있다.
- 단점 : SQL스럽지 않고 너무 복잡하고 실용성이 없다.
    * 실무에서 사용하지 않는 것이 좋다. 유지보수가 어렵고 복잡하다.
- 따라서 **QueryDSL(오픈소스 라이브러리)사용을 권장한다.**
```java
//Criteria 사용 준비
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Member> query = cb.createQuery(Member.class);

//루트 클래스 (조회를 시작할 클래스)
Root<Member> m = query.from(Member.class);

//쿼리 생성
CriteriaQuery<Member> cq = query.select(m).where(cb.equal(m.get("username"), "kim"));
List<Member> resultList = em.createQuery(cq).getResultList();

tx.commit();
```
<br>

### QueryDSL 소개
- 문자가 아닌 자바코드로 JPQL을 작성할 수 있다.
- JPQL 빌더 역할을 한다.
- 컴파일 시점에 문법의 오류를 찾을 수 있다.
- 동적 쿼리 작성이 편리하다.
- 단순하고 쉬우며 굉장히 직관적이다.
- 실무 사용이 권장된다.
- JPQL을 잘 알면 QueryDSL은 쓰기 쉽다.
<br>

### 네이티브 SQL 소개
- JPA가 제공하는 SQL을 직접 사용하는 기능
- JPQL로 해결할 수 없는 특정 데이터베이스에 의존적인 기능을 사용할 때 쓴다. 
    * 예) 오라클 CONNECT BY, 특정 DB만 사용하는 SQL 힌트
- 참고) 특정 특정 DB만 사용하는 SQL 힌트는 하이버네이트가 데이터베이스 방언에 설정해서 사용할 수 있도록 지원한다.
```java
em.createNativeQuery("SELECT ID, city, street, zipcode, USERNAME form MEMBER").getResultList();

tx.commit();
```
<br>

### JDBC 직접사용, SpringJdbcTemplate 등
- JPA를 사용하면서 JDBC 커넥션을 직접 사용하거나, 스프링 JdbcTemplate, MyBatis등을 함께 사용할 수 있다.
- 단, 영속성 컨텍스트를 적절한 시점에 강제로 플러시 하는 것이 필요하다.
- ex) JPA를 우회해서 SQL을 실행하기 직전에 영속성 컨텍스트 수동 플러시
<br>
