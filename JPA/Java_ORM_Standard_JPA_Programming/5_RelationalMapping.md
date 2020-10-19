# 연관관계 맵핑 기초

### 목표
- 객체와 테이블 연관관계의 차이를 이해한다.
- 객체의 참조와 테이블의 외래 키를 어떻게 맵핑하는지 이해한다.
- 용어 이해
    * 방향(Direction)
        - 단방향, 양방향
    * 다중성(Multiplicity)
        - 다대일(N:1), 일대다(1:N), 일대일(1:1), 다대다(N:N)
    * 연관관계의 주인(Owner)
        - 객체 양방향 연관관계는 관리하는 주인이 필요하다.
<br>

## 연관관계가 필요한 이유
> `객체지향 설계의 목표는 자율적인 객체들의 협력 공동체를 만드는 것이다' - 조영호(객체지향의 사실과 오해)
<br>

### 예제시나리오
- 멤버과 팀이 있다.
- 멤버은 하나의 팀에만 소속될 수 있다.
- 멤버과 팀은 다대일 관계다.
<br>

### 객체를 테이블에 맞춰 모델링
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_RelationalMapping_1.jpg"></p>

- 참조 대신 외래키를 그대로 사용한다.
```java
@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID") 
    private Long id;
    
    @Column(name = "USERNAME")
    private String username;
    
    private int age;
    
    @Column(name = "TEAM_ID")
    private Long teamId;
    
    // ...getter and setter...
}
```
```java
@Entity
public class Team {
    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;
    private String name;
  
    // ...getter and setter...
}
```
- 저장과 조회가 객체지향스럽지 않다.
    * 저장
        - 외래키 식별자를 직접 다뤄야한다.
    * 조회
        - 조회를 할 때 두 번 따로 조회해야 한다. 연관관계가 없기 때문에 식별자로 다시 팀을 조회한다.
```java
public Class Main {
  public static void main(String[] args) {
   
    // ...
      
    //팀 저장
    Team team = new Team();
    team.setName("TeamA");
    em.persist(team);	// 영속 상태가 된다. PK값이 세팅 된 상태.
    
    //멤버 저장
    Member member = new Memeber();
    member.UserName("member1");
    member.setTeamId(team.getId());	// PK값이 세팅 됐으므로, getId() 가능하지만 객체지향스럽지 않다.
    em.persist(member);

    // 조회
    Member findMember = em.find(Member.class, member.getId());

    // 조회한 멤버의 소속 팀 조회
    // 팀을 바로 가져오지 못하기 때문에 팀의 Id를 가져온 다음, 팀Id를 가지고 팀을 꺼내야한다.
    Long findTeamId = findMember.getTeamId();
    Team findTeam = em.find(Team.class, findTeamId);

    // ...
  }
}
```
- 따라서 객체를 테이블에 맞추어 데이터 중심으로 모델링하면 협력 관계를 만들 수 없다.
    * 테이블은 외래 키로 조인을 사용해서 연관된 테이블을 찾는다.
    * 객체는 참조를 사용해서 연관된 객체를 찾는다.
    * 테이블과 객체 사이에는 이런 큰 간격이 있다.
<br>

## 단방향 연관관계

### 객체지향 모델링
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_RelationalMapping_2.jpg"></p>

- 객체의 참조와 테이블의 외래키를 맵핑한다.
    * teamId 대신 객체 Team을 넣고 TEMA_ID에 맵핑한다.

```java
@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
  
    @Column(name = "USERNAME")
    private String username;
  
    private int age;
  
    //@Column(name = "TEAM_ID")
    //private Long teamId;
  
    @ManyToOne  // 다대일(N:1)관계를 맵핑해준다. 
    @JoinColumn(name = "TEAM_ID") // Member의 team과 MEMBER 테이블의 TEAM_ID(FK)를 맵핑 
    private Team team;
    ...
  
    // ...getter and setter...
}
```
- 저장과 조회가 객체지향스럽다.
    * 저장
        - `member.setTeam(team);`이라고만 해주면 JPA가 알아서 team에서 PK값을 꺼내 FK값으로 사용한다.
    * 조회
        - `Team findTeam = findMember.getTeam();`으로 팀을 바로 꺼낼 수 있다. 
```java
public Class Main {
  public static void main(String[] args) {
   
    // ...
      
    //팀 저장
    Team team = new Team();
    team.setName("TeamA");
    em.persist(team);
    
    //멤버 저장
    Member member = new Memeber();
    member.setUsername("member1");
    //member.setTeamId(team.getId());
    member.setTeam(team);
    em.persist(member);

    // 조회
    Member findMember = em.find(Member.class, member.getId());

    // 조회한 멤버의 소속 팀 조회
    //Long findTeamId = findMember.getTeamId();
    //Team findTeam = em.find(Team.class, findTeamId);
    Team findTeam = findMember.getTeam(); 

    // ...
  }
}
```
<br>

#### 연관관계 수정
- 만약 위의 코드에서 TeamA에서 새로운 팀으로 바꾸고 싶은 경우
    * setTeam()으로 새로운 팀을 할당하면, 연관관계가 바뀐다.
    * UPDATE 쿼리에서 업데이트가 된다. (FK가 업데이트 된다.)
```java
Team newTeam = em.find(Team.class, 100L); //100번 팀이 있다고 가정
finMember.setTeam(newTeam);
```
<br>

## 양방향 연관관계와 연관관계의 주인
- 현재까지는 Member를 통해서 Team을 얻어왔지만 Team을 통해서는 Member를 얻지 못한다.
- 그림과 같이 양방향으로 객체를 참조하게 할 때도 테이블은 변화가 없다. 
    * MEMBER의 TEAM_ID(FK)와 TEAM의 TEAM_ID(PK) JOIN하면 된다.
    * **테이블의 연관관계는 FK하나로 양방향이 다 가능하다.**
    * 문제는 객체이다. 
        - 이전까지는 Member가 Team객체를 가지고 있어서 Member -> Team은 가능했지만 Team -> Member는 불가능하다.
        - 따라서 Team에다 members라는 List를 넣어야 양방향이 가능하다. 
```java
@Entity
public class Team {
    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;
    private String name;

    // Team에 List 추가(관례 : ArrayList로 초기화 해두면 NPE가 안뜬다.)
    @oneToMany(mappedbBy = "team") // Team의 입장에서 1:N로 맵핑, mappedBy는 연결되는 쪽의 연관을 알려준다.
    private List<Member> members = new ArrayList<>();
  
    // ...getter and setter...
}
```
- 이제 반대로 탐색이 가능하다.
```java
Member findMember = em.find(Member.class, member.getId());
List<Member> members = findMember.getTeam().getMembers(); //Member에서 Team으로 Team에서 Member로 양방향

for(Member m : members) {
    System.out.pringln("m = " + m.getUsername());
}
```
<br>

### 연관관계 주인과 mappedBy
- mappedBy는 처음에 이해하기 굉장히 어렵다
- 객체와 테이블간에 연관관계를 맺는 차이를 이해해야한다.
<br>

#### 객체와 테이블간에 연관관계를 맺는 차이
- 객체 연관관계
    * Member -> Team 연관관계 1개(단방향)
    * Team -> Member 연관관계 1개(단방향)
    * 객체의 양방향 관계는 사실 양방향 관계가 아니라 서로 다른 단방향 관계 2개다.
    * 따라서 객체를 양방향으로 참조하려면 억지로 단방향 관계 2개를 만들어야 한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_RelationalMapping_3.jpg"></p>

- 테이블 연관관계
    * Member <-> Team의 연관관계 1개(양방향)
    * 테이블은 외래키 하나로 두 테이블의 연관관계를 관리한다.
    * MEMBER>TEAM_ID 외래키 하나로 양쪽으로 JOIN할 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_RelationalMapping_4.jpg"></p>
<br>

### 딜레마 - 둘 중 하나로 외래 키를 관리해야 한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_RelationalMapping_5.jpg"></p>

- 그림과 같이 객체의 참조를 양방향으로 만들었을 때 둘 중에 어느것으로 맵핑해야할까?
    *  Member의 team값이 바뀌었을 때 외래키값이 수정되어야 할지 Team의 members값이 바뀌었을 때 외래키값이 수정되어야할지 딜레마에 빠진다.
- DB 입장에서는 객체의 참조가 어떻든, MEMBER의 TEAM_ID(FK)값만 UPDATE되면 된다.
- 따라서 둘 중 하나로 외래키를 관리해야 한다.
    * **연관관계 주인**
<br>

### 연관관계의 주인
- 양방향 맵핑 규칙
    * 객체의 두 관계중 하나를 연관관계의 주인으로 지정해야 한다.
    * **연관관계의 주인만이 외래키를 관리(등록, 수정) 한다.**
    * **주인이 아닌쪽은 읽기(조회)만 가능하다.**
        - 값을 변경해도 아무일도 일어나지 않는다. 
        - 객체에 둘다 정보를 수정해도, 연관관계의 주인 것만 DB에 영향을 준다.
    * 주인은 mappedBy 속성을 사용하면 안된다.
        - mappedBy 속성 사용 하지 않고, `@JoinColumn`을 사용한다.
    * 주인이 아니면 mappedBy 속성으로 주인을 지정해야 한다.
<br>

### 누구를 주인으로 정해야 할까
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_RelationalMapping_6.jpg"></p>

- 외래키가 있는 곳을 주인으로 정해야 한다.(권장)
    * Team.members를 주인으로 설정했다고 가정하면, Team의 members의 값을 바꿨을 때 TEAM테이블이 아니라 MEMBER 테이블의 UPDATE 쿼리가 나간다(?)
    * 다른 테이블의 UPDATE쿼리가 나가면 JPA를 잘한다고하더라도 헷갈린다. (성능이슈도 있다.)
- 외래키가 있는 곳을 주인으로 정하면 많은 고민들이 해결된다.
    * DB입장에서는 외래키가 있는 곳이 무조건 N이고 없는 곳이 1이다.
    * 즉, DB의 N쪽이 항상 주인이 된다.
- 연관관계의 주인은 비즈니스적으로 중요하지 않다. 
    * 그냥 단순하게 DB 테이블로 따져서 N쪽이 주인이 되면 된다.
    * 그렇게해야 성능이슈도 없고 설계가 깔끔하다. 
    * 엔티티와 테이블이 맵핑이 되어있는 테이블에서 FK가 관리가 된다.
<br>

### 양방향 맵핑시 가장 많이 하는 실수
- 연관관계의 주인에 값을 입력하지 않는다.
    * `team.getMembers().add(member);`
    * 가짜 맵핑의 값을 INSERT or UPDATE 해봤자 DB에는 의미없다.
    * 주인이 아닌 것에 값을 넣으면 외래키 값이 NULL인 상황이 발생한다.
        - DB에 Member를 조회해보면, TEAM_ID 가 NULL이다.
```java
public Class Main {
  public static void main(String[] args) {
   
    // ...
      
    //멤버 저장
    Member member = new Memeber();
    member.setUsername("member1");
    em.persist(member);  

    //팀 저장
    Team team = new Team();
    team.setName("TeamA");
    team.getMembers().add(member);
    em.persist(team);

    em.flush();
    em.clear();

    // ...
  }
}
```
- 연관관계의 주인에 값을 넣어야 한다.
    * `member.setTeam(team);`
```java
public Class Main {
  public static void main(String[] args) {
   
    // ...
    
    //팀 저장
    Team team = new Team();
    team.setName("TeamA");
    //team.getMembers().add(member);
    em.persist(team);

    //멤버 저장
    Member member = new Memeber();
    member.setUsername("member1");
    member.setTeam(team);
    em.persist(member);  

    em.flush();
    em.clear();

    // ...
  }
}
```
<br>

### 양방향 맵핑시 주의점

#### 순수 객체 상태를 고려해서 항상 양쪽에 값을 설정하자
- JPA입장에서는 연관관계 주인에 값을 넣으면 끝나지만 다음 코드와 같은 문제가 발생한다.
    * `team.getMembers().add(member);`하지 않아도 지연 로딩을 통해서 해당 멤버를 조회해올 수 있기 때문에 (아래의 코드에서는)아무 문제가 없다.
        - `em.flush();`, `em.clear();` 하는 순간에 DB에 FK가 세팅 된다. 그래서 지연 로딩을 해도 FK로 조인해서 가져올 수 있다.
    * 그러나 `em.flush();`, `em.clear();`를 하지 않으면 DB에 쿼리가 안날라가고, FK도 없이 MEMBER가 1차 캐시에만 영속화 되어있는 상태이다.
        - members의 size도 0이다.
    * 추가적으로 JPA 없이 테스트 케이스가 동작하게끔 테스트 코드를 짤때도 에러가 발생한다.
```java
Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setUsername("member1");
// 연관관계 주인에 값 설정
member.setTeam(team);
em.persist(member);

// 역방향 연관관계를 설정하지 않아도, 지연 로딩을 통해서 Member에 접근할 수 있다.
//team.getMembers().add(member);

// 이 동작이 수행되지 않으면 FK가 설정되어 있지 않은 1차캐시에만 영속화 된 상태이다. SELECT 쿼리로 조회해봤자 list 사이즈도 0이다.
em.flush();
em.clear();

Team findTeam = em.find(Team.class, team.getId());
List<Member> findMembers = findTeam.getMembers();

for (Member m : findMembers) {
    // flush, clear가 일어난 후에는 팀의 Members에 넣어주지 않았지만, 조회를 할 수 있다. 이것이 지연로딩
    System.out.println(m.getUsername());
}

tx.commit();
```
- **결론적으로 양방향 맵핑시 양쪽에 값을 다 넣는게 맞다.**
```java
Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setUsername("member1");
// 연관관계 주인에 값 설정
member.setTeam(team);
em.persist(member);

// 주인이 아니어도 값 설정
team.getMembers().add(member);

//em.flush();
//em.clear();

Team findTeam = em.find(Team.class, team.getId());
List<Member> findMembers = findTeam.getMembers();

for (Member m : findMembers) {
    System.out.println(m.getUsername());
}

tx.commit();
```
<br>

#### 연관관계 편의 메소드를 생성하자
- 값을 2번 넣어야 하기 때문에 휴먼에러가 생길 가능성이 많다.
- 따라서 **연관관계 편의 메소드를 생성하는 것을 권장한다.**
- 메소드 설정을 하면 `team.getMembers().add(member);`로 역방향 값을 다시 넣을 필요가 없다.
- 참고로 setter와 헷갈리지 않게 changeTeam()으로 함수명을 변경하는 것이 더 좋다.(개인적 취향)
```java
// 함수명을 변경하고 역방향 값도 설정하는 코드 추가
public void changeTeam(Team team) {
    this.team = team;
    team.getMembers().add(this); // setTeam할 때 역방향 값도 함께 설정한다.
}
```
- 양방향 편의메소드가 양쪽에 있으면 문제가 생길 가능성이 높으므로 member를 기준으로 team을 넣을지, team을 기준으로 member를 넣을지 정하는 것이 좋다.
    * 둘 다 하면 무한 루프에 걸릴 수도 있다.
<br>

#### 양방향 맵핑시 무한 루프를 조심하자
- ex) toString(), lombok, JSON 생성 라이브러리
- 양방향에서 롬복에서 자동으로 toString()을 생성한다면 다음과 같이 서로 toString()을 호출해버려 무한 루프에 걸린다.
    * 롬복에서 toString() 자동생성을 웬만하면 안쓰는게 좋다.
```java
// member의 toString()
@Override
public String toString() {
    return "Team{" +
            "id=" + id +
            ", name='" + name + '\'' +
            ", members=" + members +
            '}';
}
```
```java
// Team의 toString()
@Override
public String toString() {
    return "Member{" +
            "id=" + id +
            ", username='" + username + '\'' +
            ", team=" + team +
            '}';
}
```
- JSON 생성 라이브러리에서의 문제
    * 양방향 관계의 엔티티를 JSON으로 바꾸는 무한루프에 빠져버린다.
    * 스택오버플로우가 발생하거나 장애가 발생한다.
    * 컨트롤러에서는 엔티티를 절대 직접 반환할 때 보통 발생한다.
    * 컨트롤러에서 엔티티를 반환하지 말아야 한다.
        - 무한루프에 걸린다.
        - 엔티티 변경 시 API 스펙이 바뀌어 버린다.
    * 엔티티는 DTO로 변환해서 반환하면 JSON 생성 라이브러리 때문에 생기는 문제는 없다.
<br>

### 양방향 맵핑 정리
- 단방향 맵핑만으로도 이미 연관관계 맵핑은 완료가 된것이다.
    * JPA 모델링 할 때 처음부터 단방향 맵핑으로 설계를 끝내야한다.
- 양방향 맵핑은 반대 방향으로 조회(객체 그래프 탐색) 기능이 추가된 것 뿐이다.
- JPQL에서 역방향으로 탐색할 일이 많다.
- 단방향 맵핑을 잘 하고, 테이블이 굳어진 뒤에 양방향 맵핑은 필요할 때 추가해도 된다. (테이블에 영향을 주지 않는다.)
    * 실습에서도 단방향 맵핑에서 양방향 맵핑으로 넘어가면서 추가된 것은 Team 객체(엔티티)에 members 컬렉션이 들어간것 뿐이다. 테이블은 똑같다.

- 연관관계의 주인을 정하는 기준은 비즈니스 로직을 기준으로 주인을 선택하면 안된다.
    * **연관관계 주인은 외래키의 위치를 기준으로 정해야 한다.**
<br>
