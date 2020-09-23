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
select m from Member m join m.team t
```
- 묵시적 조인
    * 경로 표현식에 의해 묵시적으로 SQL조인 발생(내부 조인만 가능)
```sql
select m.team from Member m
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
