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
> 객체지향 설계의 목표는 자율적인 객체들의 협력 공동체를 만드는 것이다' - 조영호(객체지향의 사실과 오해)
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
    team.setUsername("TeamA");
    em.persist(team);	// 영속 상태가 된다. PK값이 세팅 된 상태.
    
    //멤버 저장
    Member member = new Memeber();
    member.setName("member1");
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
    team.setUsername("TeamA");
    em.persist(team);
    
    //멤버 저장
    Member member = new Memeber();
    member.setName("member1");
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
