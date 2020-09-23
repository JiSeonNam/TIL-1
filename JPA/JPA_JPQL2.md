# 객체지향 쿼리 언어2 - 중급 문법

## 경로 표현식
- 경로 표현식은 .(점)을 찍어 객체 그래프를 탐색하는 것이다.
- 3가지 경로 표현식이 있고 3가지 중 어디로 가는냐에 따라서 내부적으로 동작하는 방식이 달라져 결과가 달라진다. 
    * 상태 필드
    * 단일 값 연관 필드
    * 컬렉션 값 연관 필드
- 따라서 3가지를 구분해서 이해해야 한다.
```sql
select m.username -> 상태 필드
from Member m
join m.team t   -> 단일 값 연관 필드
join m.orders o -> 컬렉션 값 연관 필드
where t.name = '팀A'
```
<br>

### 경로 표현식 용어 정리
- 상태 필드(state field) : 단순히 값을 저장 하기 위한 필드(ex: m.username)
- 연관 필드(association field) : 연관관계를 나타내기 위한 필드
    * 단일 값 연관 필드
        - `@ManyToOne`, `@OneToOne`
        - 대상이 엔티티 (ex: m.team)
    * 컬렉션 값 연관 필드
        - `@OneToMany,` `@ManyToMany` 
        - 대상이 컬렉션 (ex: m.orders)
<br>

### 경로 표현식의 특징
- 상태 필드(state field)
    * 경로 탐색의 끝. 더 이상 탐색할 수 없다.
```java
// m.username은 상태 필드이다. (username에서 .(점)을 찍어서 어디로 갈 수 없다)
String query = "select m.username from Member m"; 
```
- 단일 값 연관 경로
    * **묵시적 내부 조인(inner join) 발생**
        - 객체 입장에서는 .을 찍어서 필드에 접근하면 되지만, DB 입장에서는 조인이 필요하다.
        - 따라서 조심해서 써야하므로 내부 조인이 발생하도록 웬만하면 코드 짜지 않는게 좋다. (성능 튜닝에 영향을 준다.)
    * 추가 탐색이 가능하다.
```java
// m.team으로 Member에서 Team으로 가면 묵시적인 내부 조인이 발생한다.
// t.team.name 으로 탐색을 더 할 수 있다.
String query = "select m.team from Member m"; 
```
- 컬렉션 값 연관 경로
    * 묵시적 내부 조인 발생한다.
    * 추가 탐색이 불가능하다.
    * FROM 절에서 명시적 조인을 통해 별칭을 얻으면 별칭을 통해 탐색 가능하다.
```java
// t.members는 컬렉션이므로 필드에 접근하거나 더 이상 탐색할 수 없다.
// size는 가능하다. t.members.size
String query = "select t.members from Team t"; 
```
```java
// 명시적 조인을 통해 얻은 별칭으로 탐색
String query = "select m from Team t join t.members m"; 
```
<br>

### 명시적 조인과 묵시적 조인
- 명시적 조인
    * join 키워드를 직접 사용
```sql
select m 
from Member m 
join m.team t
```
- 묵시적 조인
    * 경로 표현식에 의해 묵시적으로 SQL조인 발생(내부 조인만 가능)
```sql
select m.team 
from Member m
```
<br>

### 경로 표현식 - 예제
- SELECT o.member.team FROM Order o
    * 성공
    * 조인이 2번 일어난다.
    * 경로 표현식은 쉽지만 어떤 SQL이 실행될지 예측하기 어렵다.
- SELECT t.members FROM Team
    * 성공
    * 컬렉션에서 추가 탐색을 안했기 때문에 된다.
- SELECT t.members.username FROM Team t
    * 실패
    * 컬렉션에서 size 빼고는 추가 탐색이 불가능하다.
- SELECT m.username FROM Test t Join t.members m
    * 성공
    * 컬렉션을 명시적 조인으로 별칭을 얻어서 별칭을 통해 탐색했다.
<br>

### 경로 탐색을 사용한 묵시적 조인시 주의 사항 
- 묵시적 조인은 항상 내부 조인(inner join)이 발생
- 컬렉션은 경로 탐색의 끝이므로 명시적 조인을 통해 별칭을 얻어야 추가 탐색이 가능하다.
- 경로 탐색은 주로 SELECT, WHERE 절에서 사용하지만 묵시적 조인으로 인해 SQL의 FROM (JOIN) 절에 영향을 준다.
<br>

### 결론(실무 조언)
- **가급적 묵시적 조인 대신에 명시적 조인을 사용한다.**
    * 아예 쓰지말자.
- 조인은 SQL 튜닝에 중요한 포인트이므로, 명시적 조인을 쓰자.
- 묵시적 조인은 조인이 일어나는 상황을 한눈에 파악하기 어렵다.
<br>

## 페치 조인(fetch join)
- **실무에서 매우매우 중요하다.**
- SQL 조인의 종류가 아니다.
- JPQL에서 성능 최적화 를 위해 제공하는 전용 기능이다.
- 연관된 엔티티나 컬렉션을 SQL 한번에 함께 조회 하는 기능이다.
- join fetch 명령어를 사용한다.
    * `[ LEFT [OUTER] | INNER ] JOIN FETCH 조인경로`
<br>

### 엔티티 페치 조인
- 예를 들어 그림과 같이 회원을 조회하면서 연관된 팀도 함께 조회하고 싶은 경우(SQL 한번에)
- SQL을 보면 회원 뿐만 아니라 팀(T.*)도 함께 SELECT 한다.
- JPQL
```sql
select m 
from Member m join fetch m.team
```
- 실제 번역된 SQL
```sql
SELECT M.*, T.* 
FROM MEMBER M  INNER JOIN TEAM T ON M.TEAM_ID = T.ID
```
<br>

#### 코드 실습
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_JPQL2_1.jpg"></p>

- fetch join을 사용하지 않은 경우
    * Member와 Team은 `@ManyToOne` 맵핑으로 Team은 프록시로 들어와 지연 로딩이 일어난다.
    * 실제 `m.getTeam().getName()`을 호출한 시점(Team을 호출한 시점)에 마다 DB에 쿼리를 날린다. 
        - N+1 문제 발생
        - N+1 문제는 쿼리를 1개 날려 결과를 얻기 위해 N 개의 쿼리를 날려야 한다.
        - 회원 100을 조회했을 때 최악의 경우(팀이 모두 다른 경우) 100 + 1개의 쿼리가 나간다.
```java
String query = "select m from Member m";

List<Member> result = em.createQuery(query, Member.class).getResultList();

for (Member member : result) {
     System.out.println("member = " + m.getName() + ", " + m.getTeam().getName()));
     // 회원 1, 팀A(영속성 컨텍스트에 없기 떄문에 쿼리를 날려 SQL에서 가져온다.)
     // 회원 2, 팀A(팀A는 1차 캐시에 있으므로 쿼리를 날리지 않고 가져온다.)
     // 회원 3, 팀B(팀B는 영속성 컨텍스트에 없기 때문에 쿼리를 날려 SQL에서 가져온다.)
}
```
- fetch join 사용
    * 한 번에 쿼리가 나간다.(1번)
    * 호출한 Team이 프록시가 아니라 실제 엔티티가 담긴다.
    * fetch join 사용시 지연 로딩을 하지 않고, 회원과 팀을 한번에 조회한다.
        - 지연 로딩으로 세팅해도 fetch join이 우선이다.
```java
// fetch join 사용
String query = "select m from Member m join fetch m.team";

List<Member> result = em.createQuery(query, Member.class).getResultList();

for (Member member : result) {
     System.out.println("member = " + m.getName() + ", " + m.getTeam().getName());
}
// member = 회원1, 팀A
// member = 회원2, 팀A
// member = 회원3, 팀B
```
<br>

### 컬렉션 페치 조인
- 일대다(`@OneToMany`) 관계, 컬렉션 페치 조인
- JPQL
```sql
select t 
from Team t join fetch t.members 
where t.name = '팀A'
```
- 실제 번역된 SQL
```sql
SELECT T.*, M.* 
FROM TEAM T INNER JOIN MEMBER M ON T.ID = M.TEAM_ID 
WHERE T.NAME = '팀A'
```
<br>

#### 코드 실습
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_JPQL2_2.jpg"></p>

- 컬렉션에 fetch join을 사용
    * 결과가 팀은 2개인데 결과는 3개가 찍힌다.(일대다 join이라서 데이터가 뻥튀기 된다.)
        - 참고) 일대다는 데이터가 뻥튀기 되고 다대일은 데이터 뻥튀기 X
    * 위의 그림의 [TEAM JOIN MEMBER] 테이블처럼 join된다.(팀 A가 2줄)
    * 같은 영속성 컨텍스트에 있는 것을 똑같이 쓰고 주소값도 같다.
    * 중복을 제거할 수 있다.(페치 조인과 DISTINCT 참조)
```java
// fetch join 사용
String query = "select t from Team t join fetch t.members";

List<Team> result = em.createQuery(query, Team.class).getResultList(); // result.size()는 3이다.

for (Team team : result) {
     System.out.println("team = " + team.getName() + "|" + team.getMembers().size());
}
// team = 팀A|members=2
// team = 팀A|members=2
// team = 팀B|members=1
```
<br>

### 페치 조인과 DISTINCT
- SQL의 DISTINCT는 중복된 결과를 제거하는 명령이다.
- JPQL의 DISTINCT는 2가지 기능을 제공한다.
    * SQL에 DISTINCT를 추가
    * 애플리케이션에서 엔티티 중복을 제거해준다.
        - DB가 아니라 결과가 애플리케이션에 올라왔을 때 똑같은 엔티티가 있으면 중복을 제거한다.
<br>

#### 코드 실습
- 위의 컬렉션 페치 조인 예제에서 distinct 명령을 사용해서 조회해보자.
    * 실행하면 SQL에 DISTINCT가 추가하지만 데이터가 다르므로 SQL 결과에서 중복제거가 실패한다.
        * SQL의 DISTINCT은 완전히 동일해야 중복 제거를 한다.(예제는 ID도 다르고 NAME도 다르다.)
    * 따라서 JPA에서는 JPQL의 DISTINCT가 추가로 애플리케이션에서 중복 제거를 시도한다.
    * 시도해서 같은 식별자를 가진 Team 엔티티를 제거해버린다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_JPQL2_3.jpg"></p>

```java
String query = "select distinct t from Team t join fetch t.members";

List<Team> result = em.createQuery(query, Team.class).getResultList();

System.out.println("resultSize = " + result.size()); // 2
```
<br>

### 페치 조인과 일반 조인의 차이
- 일반 조인 실행시 연관된 엔티티를 함께 조회하지 않는다.
    * 실행된 SQL에서 SELECT문을 보면 team의 정보들만 조회한다.
    * team의 members는 초기화 되지 않은 상태라서, 접근 시 쿼리가 추가로 나간다.
    * JPQL 은 결과를 반환할 때 연관관계 고려하지 않는다.
    * SELECT 절에 지정한 엔티티만 조회할 뿐이다.
```java
String query = "select t from Team t join t.members";

List<Team> result = em.createQuery(query, Team.class).getResultList();

System.out.println("resultSize = " + result.size()); // 3
```
- 페치 조인을 사용하면 쿼리 1번만 나가고 연관된 엔티티도 함께 조회 한다.(즉시 로딩)
- **페치 조인은 객체 그래프를 SQL 한번에 조회하는 개념이다.**
    * 실무에서 정말 많이 쓰인다. 
<br>

### 페치 조인의 특징과 한계
- 페치 조인 대상에는 별칭을 줄 수 없다. 주지 말자.
    * 하이버네이트는 가능하지만, 가급적 사용하지 않는 것이 관례이다.
    * 페치 조인이라는 것은 기본적으로 연관된 엔티티를 다 끌고 오는 것이다.
    * 중간에 원하는 것만 가져오고 싶으면 페치 조인을 쓰지 않고 따로 조회하는 것이 맞다. 
    * JPA의 의도한 설계는 객체 그래프로 team.getMembers() 조회했을때, 모든 Members를 접근 할 수 있어야 한다. 하지만 where 조건이 붙어버리면 누락되는 Member가 생긴다.
```java
// 이렇게 별칭을 주고 where~ 이렇게 하면 안된다.(매우 위험)
String query = "select t from Team t join t.members m where m.age > 10";
```
- 둘 이상의 컬렉션은 페치 조인 할 수 없다.
    * 일대다 연관관계도 데이터 뻥튀기가 되는데 둘 이상의 컬렉션을 페치 조인하면 일대다대다 관계가 된다.
- 컬렉션을 페치 조인하면 페이징 API(setFirstResult, setMaxResults)를 사용할 수 없다.
    * 일대일, 다대일 같은 단일 값 연관 필드들은 페치 조인해도 페이징이 가능하다.(데이터 뻥튀기가 안되기 때문)
    * 그러나 일대다는 데이터가 뻥튀기 되기 때문에 페이징할 수 없다.
    * 하이버네이트는 경고 로그를 남기고 메모리에서 페이징한다.(매우 위험하다)
    * 방향을 뒤집어 다대일 페치 조인 후 페이징하는 방법을 사용하거나
    * 페이징을 그대로 하고 페치 조인을 하지 않고 지연 로딩을 사용해 N+1 문제를 일으키면서 페이징한다.
    * 엔티티의 일대다 페치 조인 대상(Team엔티티의 members 컬렉션)에 `@BatchSize(100)`으로 둬서 연관된 객체 100개씩만 가져오는 방법이 있다.
    * 보통 이 설정을 global setting으로 가져간다.
        * persistence.xml에 `<property name="hibernate.defalult_batch_fetch_size" value="100"/>`
            - 지연로딩 N+1 쿼리가 나가지 않고 테이블 갯수만큼만 쿼리가 나간다.
- 페치 조인은 연관된 엔티티들을 SQL 한 번으로 조회한다. (성능 최적화가 된다.)
- 페치 조인은 직접 적용하는 글로벌 로딩 전략보다 항상 우선시 된다.
    * 글로벌 로딩 전략 : `@OneToMany(fetch = FetchType.LAZY)`
- 실무에서는 글로벌 로딩 전략은 모두 지연로딩으로 해놓고 최적화가 필요한 곳(N+1 문제 발생)은 페치 조인을 적용한다. 
    * 이 전략은 대부분의 성능 문제를 해결 할 수 있다.
<br>

### 페치 조인 정리
- 모든 것을 페치 조인으로 해결할 수는 없다
- 페치 조인은 객체 그래프를 유지할 때 사용하면 효과적이다
- 여러 테이블을 조인해서 엔티티가 가진 모양이 아닌 전혀 다른 결과를 내야 하면, 페치 조인 보다는 일반 조인을 사용하고 필요한 데이터들만 조회해서 DTO로 반환하는 것이 효과적이다.
    * 세 가지 방법이 있다.
        - 페치 조인해서 엔티티를 조회해서 사용한다.
        - 페치 조인해서 가져온 결과를 애플리케이션에서 DTO로 변환해서 사용한다.
        - JPQL을 처음 작성할 때 부터, DTO로 결과를 받아올 수 있게 한다.
- 페치 조인을 잘 알아야 성능상 이점이 된다.
<br>

## 다형성 쿼리
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_JPQL2_3.jpg"></p>

- 그림과 같이 다형적으로 설계를 한 경우 JPA는 특수 기능을 지원한다.
- 조회 대상을 특정 자식으로 한정 지을 수 있다.
- ex) Item 중에 Book, Movie를 조회
- JPQL
```sql
select i from Item i
where type(i) IN (Book, Movie)
```
- 실제 번역된 SQL
```sql
select i from i
where i.DTYPE in ('B', 'M')
```
<br>

### TREAT(JPA 2.1)
- 자바의 타입 캐스팅과 유사하다.
- 상속 구조에서 부모 타입을 특정 자식 타입으로 다룰 때 사용한다.
- FROM, WHERE, SELECT(하이버네이트 지원)절에서 사용한다.
- ex) 부모인 Item과 자식 Book이 있다.
- JPQL
```sql
select i from Item i
where treat(i as Book).author = 'kim'
```
- 실제 번역된 SQL
```sql
select i.* from Item i
where i.DTYPE = 'B' and i.author = 'kim'
```
<br>

## 엔티티 직접 사용

### 엔티티 직접 사용 - 기본 키 값
- JPQL에서 엔티티를 직접 사용하면 SQL에서 해당 엔티티의 기본키 값을 사용한다.
- JPQL
```sql
select count(m.id) from Member m // 엔티티의 아이디를 사용
select count(m) from Member m // 엔티티를 직접 사용
```
- 실제 번역된 SQL
    * 둘 다 같은 SQL이 실행된다.(id가 기본키 값이므로)
    * 참고) 파라미터로 넘겨도, 식별자를 직접 전달해도 똑같다.
```sql
select count(m.id) as cnt from Member m
```
<br>

### 엔티티 직접 사용 - 외래 키 값
- JPQL
```java
Team team = em.find(Team.class, 1L);
// 파라미터로 넘긴 team은 Team 엔티티의 PK이고 
// m.team이 가르키는 것은 Member 엔티티가 가지고 있는(@JoinColumn에 선언되어 있는) TEAM_ID(FK)이다.
String query = "select m from Member m where m.team = :team";
List result = em.createQuery(query)
	            .setParameter("team", teamA)
	            .getResultList();
```
```java
String query = "select m from Member m where m.team.id = :teamId";
List result = em.createQuery(query)
	.setParameter("teamId", teamId)
	.getResultList();
```
- 실제 번역된 SQL
```sql
select m.* from Member m where m.team_id = ?
```