# 고급 맵핑

## 상속 관계 맵핑
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_AdvancedMapping_1.jpg"></p>

- 객체는 상속관계가 있지만 관계형 데이터베이스는 상속 관계가 없다.
- 그나마 슈퍼타입 서브타입 관계라는 모델링 기법이 객체 상속과 유사하다.
- 상속관계 맵핑이라는 것은 객체의 상속 구조와 DB의 슈퍼타입 서브타입 관계를 맵핑하는 것이다.
- 슈퍼타입 서브타입 논리 모델을 실제 물리 모델로 구현하는 방법에는 3가지가 있다.
    * 각각 테이블로 변환 -> 조인 전략
    * 통합 테이블로 변환 -> 단일 테이블 전략
    * 서브타입 테이블로 변환 -> 구현 클래스마다 테이블 전략
<br>

### 조인 전략
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_AdvancedMapping_2.jpg"></p>

- ITEM, ALBUM, MOVIE, BOOK 테이블을 만들 고 필요할 경우 JOIN으로 데이터를 구성한다.
- NAME, PRICE가 ITEM 테이블에만 저장되고, ALBUM, MOVIE, BOOK이 각자의 데이터만 저장한다.
    * INSERT 할때는 2번 INSERT한다.
    * 조회할 때는 PK, FK로 JOIN해서 데이터를 가져온다.
- 가장 정규화 된 방법으로 구현하는 방식이다.
- 보통 구분하는 컬럼(DTYPE)를 둔다.
- 사용 방법 : 부모클래스에서 `@Inheritance(strategy = InheritanceType.JOINED)`로 설정한다.
```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn // 하위 테이블의 구분 컬럼 생성(default = DTYPE) 
public class Item {

    @Id @GeneratedValue
    private Long id;

    private String name;
    private int price;
}
```
```java
@Entity
public class Album extends Item {
    private String artist;
}
```
```java
@Entity
public class Movie extends Item {
    private String director;
    private String actor;

    // ...getter and setter...
}
```
```java
@Entity
public class Book extends Item {
    private String author;
    private String isbm;
}
```
- Movie 객체 저장
    * ITEM 테이블에 id, name, price 생성, MOVIE 테이블에 actor, director, id 생성
    * INSERT 쿼리가 2번 나간다.
    * DTYPE에 Entity 이름이 기본값으로 저장된다.
```java
Movie movie = new Movie();
movie.setDirector("aaaa");
movie.setActor("bbbb");
movie.setName("바람과함께사라지다.");
movie.setPrice(10000);

em.persist(movie);

tx.commit();
```
- Movie 객체 조회
    * ITEM 테이블과 MOVIE 테이블을 JOIN해서 데이터를 가져온다.
```java
Movie movie = new Movie();
movie.setDirector("aaaa");
movie.setActor("bbbb");
movie.setName("바람과함께사라지다.");
movie.setPrice(10000);

em.persist(movie);

em.flush();
em.clear();

Movie findMove = em.find(Movie.class, movie.getId());
System.out.println("findMove = " + findMove);

tx.commit();
```
<br>

### 단일 테이블 전략
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_AdvancedMapping_3.jpg"></p>

- 논리 모델을 한 테이블로 합치는 것이다.
- ALBUM, MOVIE, BOOK을 구분할 DTYPE 컬럼으로 어떤 것인지 구분한다.
    * 반드시 구분하는 컬럼이 있어야 한다.
    * `@DiscriminatorColumn`을 선언해 주지 않아도, 기본으로 DTYPE 컬럼이 생성된다.
    * 한 테이블에 모든 컬럼을 저장하기 때문에, DTYPE 없이는 테이블을 판단할 수 없다.
- INSERT 쿼리가 1번만 나가고, JOIN할 필요도 없기 때문에 성능이 좋다.
- 단일 테이블 적용
    * strategy를 SINGLE_TABLE로 변경하기만 하면 된다.
        - JPA의 장점이다. 테이블 구조의 변동이 일어났는데, 코드를 거의 손대지 않고 어노테이션만 수정하면 된다.
    * `@Inheritance(strategy = InheritanceType.SINGLE_TABLE)`
```java
@Entity
@DiscriminatorColumn
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
public class Item {

    @Id @GeneratedValue
    private Long id;

    private String name;
    private int price;
}
```
<br>

### 구현 클래스마다 테이블 전략
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/JPA_AdvancedMapping_4.jpg"></p>

- ALBUM, MOVIE, BOOK 테이블을 만들어 컬럼들을 각 테이블마다 모두 가지는 방법이다.
    * NAME, PRICE 컬럼들이 중복되도록 허용한다.
- 구현 클래스마다 테이블 생성 전략 적용
    * strategy를 TABLE_PER_CLASS로 변경한다.
    * `@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)`
```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Item { //  ITEM 엔티티는 실제 생성되는 테이블이 아니므로 abstract 클래스여야 하고, @DiscriminatorColumn도 필요가 없어진다.

    @Id @GeneratedValue
    private Long id;

    private String name;
    private int price;
}
```
- 구현 클래스마다 테이블 생성 전략 문제점
    * INSERT는 간단하지만 조회는 굉장히 비효율적으로 동작한다.
    * union all로 전체 하위 테이블을 다 찾는다.
<br>

### 상속 관계 맵핑 전략 정리
- 조인 전략
    * 장점
        - 테이블이 정규화가 되어있다.(따라서 조인 전략이 가장 권장된다.)
        - 외래키 참조 무결성 제약조건을 활용 가능하다.
            * ITEM의 PK가 ALBUM, MOVIE, BOOK의 PK이자 FK이므로 다른 테이블에서 ITEM 테이블만 바라보도록 설계하는 것이 가능하다. 이렇게 하면 설계가 깔끔하다.
        - 저장 공간이 효율화된다.
    * 단점
        - 조회시 조인을 많이 사용한다. (성능 저하)
        - 조회 쿼리가 복잡하다.
        - 데이터를 저장할 때 INSERT 쿼리가 2번 나간다.(이건 큰 단점은 아니다)
- 단일 테이블 전략
    * 장점
        - JOIN이 필요 없으므로 일반적인 조회 성능이 빠르다.
        - 조회 쿼리가 단순하다.
    * 단점
        - 자식 엔티티가 맵핑한 컬럼은 모두 NULL을 허용해야 한다.
        - 단일 테이블에 모든 것을 저장하므로 테이블이 커질 수 있다.
        - 상황에 따라서 조인 전략보다 성능이 오히려 느려질 수 있다.
- 구현 클래스마다 테이블 전략
    * 결론적으로 이 전략은 쓰지 않는게 좋다.
        - ORM을 하다보면 DB쪽과 객체지향 개발쪽에서 trade off를 할 때가 있는데, 이 전략은 둘 다 추천하지 않는다.
    * 장점
        - 서브 타입을 명확하게 구분해서 처리할 때 효과적이다.
        - NOT NULL 제약조건을 사용할 수 있다.
    * 단점
        - 여러 자식 테이블을 함께 조회할 때 성능이 느리다.(UNION SQL)
        - 자식 테이블을 통합해서 쿼리하기 어렵다.
        - 변경이라는 관점으로 접근할 때 굉장히 좋지 않다.
- 결론
    * 기본적으로는 조인 전략을 가져가는게 좋다.
    * 조인 전략과 단일 테이블 전략의 장단점을 고민해서 알맞은 전략을 선택하는 것이 좋다. 
    * 구현 클래스마다 테이블 전략은 쓰지 않는게 좋다.
    * 굉장히 심플하고 확장의 가능성도 적으면 단일 테이블 전략을 사용하고 비즈니스 적으로 중요하고, 복잡하고, 확장될 확률이 높으면 조인 전략을 사용하는 것이 권장된다.
<br>

