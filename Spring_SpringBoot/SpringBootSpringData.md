# 스프링 데이터

## 소개
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/SpringBootSpringData_1.jpg"></p>

## 인메모리 데이터베이스
- Spring-JDBC가 classpath에 있으면 자동 설정이 필요한 빈을 설정 해준다.
    * DataSource
    * JdbcTemplate
<br>

### 스프링 부트가 지원하는 인메모리 데이터베이스
- H2
- HSQL
- Derby
<br>

### 인 메모리 데이터베이스 기본 연결 정보 확인 방법
- URL : "testdb"
- username : "sa"
- password : ""
<br>

### H2 콘솔 사용하는 방법
1. pom.xml에 spring-boot-devtools 의존성 추가
2. application.properties에 spring.h2.console.enalbed=true 추가
- /h2-console로 접속(이 path도 바꿀 수 있다)
<br>

### 실습 코드
```java
@Component  // bean으로 등록되야 한다.
public class H2Runner implements ApplicationRunner {

    @Autowired  // 기본적으로 DataSource가 bean으로 등록되므로 주입 받아 사용 가능
    DataSource dataSource;

    @Autowired
    JdbcTemplate jdbcTemplate;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        try(Connection connection = dataSource.getConnection()) {
            // connection의 메타 정보에서 URL, UserName 확인
            System.out.println(connection.getMetaData().getURL());
            System.out.println(connection.getMetaData().getUserName());

            Statement statement = connection.createStatement();
            String sql = "CREATE TABLE USER (id INTEGER NOT NULL, name VARCHAR(255), PRIMARY KEY(id))";
            statement.executeUpdate(sql);
        }
        jdbcTemplate.execute("INSERT INTO USER VALUES (1, 'hayoung')");

    }
}
```
<br>

## DBCP와 MySQL

### DBCP(DataBase Connection Pool)
- Database의 connection을 미리 여러 개 만들어 놓고 애플리케이션이 필요로 할 때마다 가져다 쓰는 개념
- connection 유지 갯수, 시간 등 여러가지 설정들을 할 수 있다.
- DBCP가 애플리케이션 성능에 아주 핵심적인 역할을 하므로 DBCP에 버그가 있으면 애플리케이션에 아주 심각한 문제가 발생한다.
    * DBCP에 대해서 충분히 알고 사용을 해야 한다.
<br>

### 스프링 부트가 지원하는 DBCP
1. [HikariCP](https://github.com/brettwooldridge/HikariCP)
    * [HikariCP가 제공하는 주요 설정](https://github.com/brettwooldridge/HikariCP#frequently-used)
2. [Tomcat CP](https://tomcat.apache.org/tomcat-9.0-doc/jdbc-pool.html)
3. [Commons DBCP2](https://commons.apache.org/proper/commons-dbcp/)
- 스프링 부트는 기본적으로 HikariCP라는 DBCP를 선택했다.
    * 3가지 DBCP가 classpath에 있는 경우 HikariCP를 먼저 사용한다.
<br>

### 스프링 부트에서 DBCP 설정 방법
- application.properties에서 spring.datasource뒤에 사용하는 DBCP 이름, DBCP가 제공하는 설정값들을 설정한다.
    * 설정할 떄 위의 문서를 확인해서 설정하는게 좋다.
- spring.datasource.hikari.*
- spring.datasource.tomcat.*
- spring.datasource.dbcp2.*
<br>

### 스프링 부트에서의 MySQL
- MySQL 커넥터 의존성 추가
```html
<dependency>
    <groupIDd>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```
<br>

### MySQL 추가 (도커 사용) 
- 위의 의존성만 추가하면 사용가능한 것이 아니라 MySQL을 설치해야 한다. 
- 도커를 사용하면 MySQL을 손쉽게 설치할 수 있다.
    * docker : 컨테이너 솔루션으로 커널을 공유하기 떄문에 가상 머신을 사용하는 것보다 훨씬 빠르게 설치 가능하다.
- `docker run -p 3306:3306 --name mysql_boot ​-e MYSQL_ROOT_PASSWORD=1​ -e MYSQL_DATABASE=springboot​ -e MYSQL_USER=hayoung -e MYSQL_PASSWORD=pass​ -d mysql`
- MySQL 접속 : `docker exec -i -t mysql_boot bash` (컨테이너 안에 들어가 bash를 실행하라는 명령어)
- `mysql -u hayoung -p`
<br>

### MySQL용 Datasource 설정
- `spring.datasource.url=jdbc:mysql://localhost:3306/springboot?useSSL=false`
- `spring.datasource.username=hayoung`
- `spring.datasource.password=pass`
<br>

### MySQL 접속시 에러
- 특정 버전 이상부터 SSL connection을 강제(추천)화 한다.
- useSSL=false를 하거나 useSSL=true로 주고 truststore를 줘서 SSL 접속을 할 수 있게끔 만들어주면 된다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/SpringBootSpringData_2.jpg"></p>

<br>

### MySQL 라이센스 (GPL) 주의
- 라이센스 비용을 내고 싶지 않으면 MySQL 대신 MariaDB 사용하면 된다.
- 소스 코드 공개 의무 여부 확인
- 사실 가장 좋은건 라이센스 비용도 없고 소스 코드 공개 의무도 없는 Postgre이다.
<br>

## PostgreSQL 설정
- 의존성 추가
```html
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
</dependency>
```
- application.properties에 설정 추가
```
spring.datasource.url=jdbc:postgresql://localhost:3306/springboot?useSSL=false
spring.datasource.username=hayoung
spring.datasource.password=pass
```
- PostgreSQL 설치 및 서버 실행 (도커 사용)
```
docker run -p 5432:5432 -e POSTGRES_PASSWORD=pass -e POSTGRES_USER=hayoung -e POSTGRES_DB=springboot --name postgres_boot -d postgres

docker exec -i -t postgres_boot bash

su - postgres

psql springboot

데이터베이스 조회
\list

테이블 조회
\dt

쿼리
SELECT * FROM account;
```
<br>

### PostgreSQL 경고 메세지
- 해결 메세지를 application.properties에 추가
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/SpringBootSpringData_3.jpg"></p>

<br>

## 스프링 데이터 JPA
- ORM(Object-Relational Mapping)과 JPA (Java Persistence API)
    * ORM : 객체와 릴레이션을 맵핑할 때 발생하는 개념적 불일치를 해결하는 프레임워크
    * JPA: ORM을 위한 자바 (EE) 표준
    * JPA는 [하이버네이트](http://hibernate.org/orm/what-is-an-orm/) 기반으로 만들어져 있다.
        * 하이버네이트에 있는 모든 기능을 JPA가 모두 커버하진 않는다.
- 스프링 데이터 JPA
    * JPA를 아주 쉽게 사용할 수 있게 스프링 데이터로 추상화 시켜놓은 것
    * Repository 빈 자동 생성
    * 쿼리 메소드 자동 구현
    * @EnableJpaRepositories (스프링 부트가 자동으로 설정 해줌.)
    * SDJ(스프링데이터 JPA) -> JPA -> Hibernate -> Datasource
<br>

### 스프링 데이터 JPA 연동
- 스프링 데이터 JPA 의존성 추가
```html
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```
- 원래는 @EnableJpaRepositories를 사용해서 설정해야 사용할 수 있지만 자동 설정해준다.
- @Entity 클래스 만들기
```java
@Entity
public class Account {

    @Id
    @GeneratedValue // Repository를 통해 저장할 때 값을 자동으로 값 생성, 없으면 Id를 일일히 줘야한다. 
    private Long id;
    private String username;
    private String password;

    ...getter and setter...

    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (o == null || getClass() != o.getClass()) {
            return false;
        }
        Account account = (Account) o;
        return Objects.equals(id, account.id) &&
            Objects.equals(username, account.username) &&
            Objects.equals(password, account.password);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, username, password);
    }
}
```
- Repository 만들기
```java
@Repository
public interface AccountRepository extends JpaRepository<Account, Long> {

    Optional<Account> findByUsername(String username);
}
```
- H2 DB를 테스트 의존성 추가
```html
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>test</scope>
</dependency>
```
- Repository 테스트 만들기
    * 슬라이스 테스트(Repository와 관련된 bean들만 등록해서 테스트)
```java
@RunWith(SpringRunner.class)
@DataJpaTest    // 슬라이스 테스트
public class AccountRepositoryTest {

    @Autowired
    DataSource dataSource;

    @Autowired
    JdbcTemplate jdbcTemplate;

    @Autowired
    AccountRepository accountRepository;

    @Test
    public void di() throws SQLException {  //di : dependency injection
    //try resource를 사용하면 finally 블록에서 자원 해제 동작을 구현하지 않아도, 자원을 종료해준다.
        try(Connection connection = dataSource.getConnection()) {
            DatabaseMetaData metaData = dataSource.getConnection().getMetaData();
            System.out.println(metaData.getURL());
            System.out.println(metaData.getDriverName());
            System.out.println(metaData.getUserName());
        }
    }
}
```
- @DataJpaTest를 사용하지 않고 @SpringBootTest로 테스트 하면 integration 테스트가 된다.
    * Application에 있는 @SpringBootApplication을 찾아서 모든 빈들을 다 등록하고 application.properties가 적용되고 postgreSQL 사용하게 된다.
    * @SpringBootTest로 실제 DB 테스트하는 것보다 빠르고, 실제 DB를 사용한다고 했을때 테스트 코드에서 데이터를 변경하면 실제 DB의 데이터가 변경되기 때문에 별도의 Test DB 설정을 가져가야 한다.
    * @SpringBootTest(properties = "spring.datasource.url=''") 로 다른 데이터베이스를 설정해서 사용이 가능하긴 하지만 테스트 할때는 인메모리 데이터베이스로 슬라이스 테스트 하기를 권장된다.
    * 슬라이스 테스트를 할 때는 반드시 In-memory 데이터베이스가 필요 하므로 H2를 test 의존성으로 추가해야 한다. (실습 기준)
 - Optional 사용
```java
@Repository
public interface AccountRepository extends JpaRepository<Account, Long> {

    Optional<Account> findByUsername(String username);
}
```
```java
@RunWith(SpringRunner.class)
@DataJpaTest    // 슬라이스 테스트
public class AccountRepositoryTest {

    @Autowired
    DataSource dataSource;

    @Autowired
    JdbcTemplate jdbcTemplate;

    @Autowired
    AccountRepository accountRepository;
    
    @Test
    public void account() {
        Account account = new Account();
        account.setUsername("hayoung");
        account.setPassword("pass");
    
        //Optional<>로 받으면 결과값 검증을 isEmpty(), isNotEmpty()로 해야한다.
        Account newAccount = accountRepository.save(account);
        assertThat(newAccount).isNotNull();

        Optional<Account> existingAccount = accountRepository.findByUsername(newAccount.getUsername());
        assertThat(existingAccount).isNotEmpty();

        Optional<Account> notExistingAccount = accountRepository.findByUsername("Kimhayoung");
        assertThat(notExistingAccount).isEmpty();
    }
}
```
<br>

## 데이터 베이스 초기화

### JPA를 사용한 데이터베이스 초기화
- application.properties에 추가
- `spring.jpa.hibernate.ddl-auto` : update, create, create-drop 셋 중 하나를 주면 자동으로 스키마가 생성된다.
    * update : 기존에 있는 스키마를 유지하고 추가된 것만 스키마 변경한다.
        - 데이터 유지 가능
        - update로 놓고 쓰는 동안에는 스키마가 지저분해진다
        - 따라서 개발할 때는 편리하지만 운영 시 위험하다. (개발할 때 잠깐 잠깐 쓰는게 좋다)
    * create : 초반에 띄울 때 지우고 새로 만든다.
    * create-drop : 처음에 만들어주고 애플리케이션 종료 시 스키마를 지운다. 
    * validate: 현재 Entity 맵핑이 릴레이션 DB에 맵핑할 수 있는 상황인지 맵핑이 되는지를 검증(ddl에 변경을 가하는건 아니기 때문에 ddl=false로 준다)
- `spring.jpa.generate-dll=true`로 설정 해줘야 위의 명령어가 동작한다. (기본은 false로 되어있다)
- `spring.jpa.show-sql=true`로 설정하면 스키마가 생성되는 것을 볼 수 있다.
    * console에 hibernate 로그를 보여준다.
<br>

### SQL 스크립트를 사용한 데이터베이스 초기화
- JPA를 사용하지 않고 DB 초기화를 진행할 수 있다.
- 순서는 schema.sql이 먼저 호출되고 다음으로 data.sql이 호출된다.
    * 초기화 데이터가 필요한 경우 data.sql에 정의해서 초기 데이터를 넣을 수 있다.
- 각각 platform을 정의해서 platform에 특화된 스크립트도 정의할 수 있다. 
    * ex) `spring.datasource.platform=postgresql`
- `schema.sql` 또는 `schema-${platform}.sql`
- `data.sql` 또는 `data-${platform}.sql`
- `${platform}` 값은 `spring.datasource.platform` 으로 설정 가능.
- 테스트 코드 : 데이터베이스 초기화 SQL 생성
```SQL
drop table account if exists
drop sequence if exists hibernate_sequence
create sequence hibernate_sequence start with 1 increment by 1
create table account (id bigint not null, email varchar(255), password varchar(255), username varchar(255), primary key (id))
```
- src/resources 경로에 schema.sql 파일을 생성하고 SQL을 붙여넣는다.
<br>

## 데이터베이스 마이그레이션
- Flyway와 Liquibase가 대표적이다. (강의에서는 Flyway마 사용)
- DB 스키마 변경, 데이터 변경을 버전 관리하듯이 관리할 수 있다.

### [Flayway](https://docs.spring.io/spring-boot/docs/2.0.3.RELEASE/reference/htmlsingle/#howto-execute-flyway-database-migrations-on-startup)
- 기본적으로 SQL 파일을 사용한다.
- 의존성 추가
```html
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
```
<br>

### 마이그레이션 디렉토리
- db/migration 또는 db/migration/{vendor}
- spring.flyway.locations로 변경 가능
<br>

### 마이그레이션 파일 이름
- V숫자__이름.sql
- V는 꼭 대문자로
- 숫자는 순차적으로 (타임스탬프 권장)
- 숫자와 이름 사이에 언더바 두 개(__).​
- 이름은 가능한 서술적으로
<br>

### 마이그레이션 실습
-  /src/resources 에 db/migration 폴더 생성 
    * db.migration 으로 생성하면 에러 난다
    * 이 안에 차곡차곡 SQL을 쌓아 나가면 된다.
-  V1__init.sql 파일 생성
-  schema.sql의 SQL을 복사해 V1_init.sql에 붙여넣고 schema.sql 파일 제거
-  V1__init.sql 문법 수정
    * 한 줄 마다 세미콜론(;)으로 닫아야 한다.
```SQL
drop table if exists account;
drop sequence if exists hibernate_sequence;
create sequence hibernate_sequence start with 1 increment by 1;
create table account (id bigint not null, email varchar(255), password varchar(255), username varchar(255), primary key (id));
```
- application.properties
    * `spring.jpa.hibernate.ddl-auto=validate`로 변경(실제 db와 엔티티 맵핑 검증)
- 애플리케이션을 실행해서 Schema가 정상적으로 생성되는 지 확인
- Account Entity에 새로운 컬럼 추가 
    * active boolean
```java
@Entity
public class Account {

    @Id
    @GeneratedValue //Repository를 통해 저장을 할 때 ID를 자동으로 생성
    private Long id;
    private String username;
    private String password;
    private String email;

    private boolean active;

    ...getter and setter...

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Account account = (Account) o;
        return active == account.active &&
                Objects.equals(id, account.id) &&
                Objects.equals(username, account.username) &&
                Objects.equals(password, account.password) &&
                Objects.equals(email, account.email);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, username, password, email, active);
    }
}
```
- 추가하고 바로 실행하면 맵핑이 안되기 때문에 hibernate가 validation하다가 에러가 난다.
- 스키마 변경 방법
    * 한번 적용이 된 migration 스크립트(V1_init.sql)는 절대로 다시 건드리면 안된다.
    * 따라서 새로운 파일로 만들어야 한다. (V2_add_active.sql)
    * 스키마 변경뿐만 아니라 데이터 변경도 가능
```SQL
ALTER TABLE account ADD COLUMN active BOOLEAN;
```
<br>

## Redis
- 캐시, 메세지 브로커, key-value 스토어 등으로 사용 가능
- redis 의존성 추가
```html
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
<br>

### 스프링 데이터 [Redis](https://spring.io/projects/spring-data-redis)
- 기본적으로 Reids를 사용하는 2가지 방법
    * StringRedisTemplate 또는 RedisTemplate
- extends CrudRepository
<br>

### Redis 설치 및 실행(도커 사용)
- Redis 설치
    * `docker run -p 6379:6379 --name redis_boot -d redis`
    * 기본적으로 컨테이너 밖으로 expose하는 포트가 6379이므로 localhost에서 6379로 받는다
- Redis 접속
    * `docker exec -i -t redis_boot redis-cli`
<br>

### [Redis 주요 커맨드](https://redis.io/commands)
- keys *
- get {key}
- hgetall {key}
- hget {key} {column}
<br>

### Redis 커스터마이징
- spring.redis에 있는 프로퍼티들을 수정하면 된다.
- 아무런 설정없이 Redis를 쓸 수 있는 이유는 포트번호 6379로 컨테이너에 있는 6379포트를 연결했기 떄문이다.
    * 스프링 부트의 Redis 기본 포트 : 6379
- 만약 다른 포트에 연결 했다면 포트 번호를, localhost가 아니라 다른 위치에 있는 Redis에 접속한다면 url도 바꿔야 한다.
- `spring.redis.*`
<br>

### Redis 실습
```java
@Component
public class RedisRunner implements ApplicationRunner {

    @Autowired
    StringRedisTemplate redisTemplate;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        ValueOperations<String, String> values = redisTemplate.opsForValue();
        values.set("hayoung", "Kimhayoung");
        values.set("springboot","2.0");
        values.set("hello", "world");
    }
}
```
- Repository를 만들어 사용하는 방법
    * Account 클래스 생성 하고 @RedisHash 추가
    * Account에 대한 Repository 생성
    * Runner로 AccountRepository 사용해 Account를 저장하거나 가져올 수 있다.
```java
@RedisHash("accounts")
public class Account {

    @Id
    private String id;
    private String username;
    private String email;

    ...getter and setter...
}
```
```java
public interface AccountRepository extends CrudRepository<Account, String> {

}
```
```java
@Component
public class RedisRunner implements ApplicationRunner {

    @Autowired
    StringRedisTemplate redisTemplate;

    @Autowired
    AccountRepository accountRepository;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        ValueOperations<String, String> values = redisTemplate.opsForValue();
        values.set("hayoung", "Kimhayoung");
        values.set("springboot","2.0");
        values.set("hello", "world");

        Account account = new Account();
        account.setEmail("khy07181@gmail.com");
        account.setUsername("hayoung");

        accountRepository.save(account);
        // 저장하면 Id가 생기기 때문에 가져올 수 있다.
        Optional<Account> byId = accountRepository.findById(account.getId());
        System.out.println(byId.get().getUsername());
        System.out.println(byId.get().getEmail());
    }
}
```
- 해쉬 값 조회
```
hget accounts:해쉬값 field
```
- 전체 해쉬 값 조회
    * 기본적으로 적은 정보 이외에 class 정보가 추가적으로 들어가 있다.
```
hgetall accounts:해쉬값
```
<br>

## [MongoDB](https://www.mongodb.com/)
- JSON 기반의 도큐먼트 데이터베이스이다.
    * 스키마가 없다.
- 의존성 추가
```html
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```
<br>

### 스프링 데이터 MongoDB
- bean들을 자동으로 설정해주고 지원해줘서 바로 주입받아 사용할 수 있다.
    * MongoTemplate
    * MongoRepository
- 내장형 MongoDB (테스트용)
    * de.flapdoodle.embed:de.flapdoodle.embed.mongo 의존성 추가 시 사용 가능
- @DataMongoTest
    * MongoRepository에 관련된 bean들만 등록
<br>

### MongoDB 설치 및 실행 (도커)
- `docker run -p 27017:27017 --name mongo_boot -d mongo`
- `docker exec -i -t mongo_boot bash`
- `mongo`
<br>

### MongoDB 실습
- Account 클래스 생성
```java
// collection이 SQL로 치면 table이름이고 각각의 document들이 collection에 들어간다. 
@Document(collection = "accounts")
public class Account {

    @Id
    private String id;
    private String username;
    private String email;

    ...getter and setter...
}
```
```java
@SpringBootApplication
public class Application {

    @Autowired
    MongoTemplate mongoTemplate;

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    public ApplicationRunner applicationRunner() {
        return args -> {
            Account account = new Account();
            account.setEmail("aaa@bbb");
            account.setUsername("aaa");

            mongoTemplate.insert(account);

            System.out.println("finished");
        };
    }
}
```
- Repository를 만들어 사용하는 방법
```java
public interface AccountRepository extends MongoRepository<Account, String> {

}
```
```java
@SpringBootApplication
public class Application {

    @Autowired
    AccountRepository accountRepository;

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    public ApplicationRunner applicationRunner() {
        return args -> {
            Account account = new Account();
            account.setEmail("khy07181@gmail.com");
            account.setUsername("Kimhayoung");
            accountRepository.insert(account);

            System.out.println("finished");
        };
    }
}
```
<br>

### 내장형 MongoDB (테스트용)을 사용한 Slicing Test
- 테스트용 코드가 운영용 MongoDB에 데이터를 조회하면 문제가 번거롭기 때문에 내장형 MongoDB를 사용하면 된다.
- 운영용 MongoDB에 데이터를 조회하는 것이 아니라 Test용 MongoDB를 사용한다.
- 의존성 추가
```html
<dependency>
    <groupId>de.flapdoodle.embed</groupId>
    <artifactId>de.flapdoodle.embed.mongo</artifactId>
    <scope>test</scope>
</dependency>
```
- AccountRepository에 findByEmail을 Optional로 받기
```java
public interface AccountRepository extends MongoRepository<Account, String> {
    Optional<Account> findByEmail(String email);
}
```
- Slicing Test 작성 
```java
@RunWith(SpringRunner.class)
@DataMongoTest  // MongoRepository에 관련된 bean들만 등록된다.
public class AccountRepositoryTest {

    @Autowired
    AccountRepository accountRepository;

    @Test
    public void findByEmail() {
        Account account = new Account();
        account.setUsername("hayoung");
        account.setEmail("hayoung07181@gmail.com");

        accountRepository.save(account);

        Optional<Account> byId = accountRepository.findById(account.getId());
        assertThat(byId).isNotEmpty();

        Optional<Account> byEmail = accountRepository.findByEmail(account.getEmail());
        assertThat(byEmail).isNotEmpty();
        assertThat(byEmail.get().getUsername()).isEqualTo("hayoung");
    }
}
``` 
<br>

## [Neo4j](https://neo4j.com/)
- 노드간의 연관 관계를 영속화하는데 유리한 그래프 데이터베이스이다.
- 의존성 추가
    * 버전에 따라 여러가지 bean들을 설정해준다. 
    * 하위호환성이 안좋다
        - 최신 버전에서는 이전에 제공하던 bean이 등록되지 않는 경우가 있다.
```html
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-neo4j</artifactId>
</dependency>
```
<br>

### 스프링 데이터 Neo4J
- Neo4jTemplate (Deprecated)
- SessionFactory
- Neo4jRepository
<br>

### Neo4j 설치 및 실행 (도커)
- `docker run -p 7474:7474 -p 7687:7687 -d --name noe4j_boot neo4j`
    * 최신 버전에서는 포트 맵핑을 HTTP용과 Bolt라는 프로토콜용으로 2개 해줘야 한다.
- UI 브라우저 환경 제공
    * [http://localhost:7474/browser](http://localhost:7474/browser)
    * 기본 password는 neo4j이고 로그인하면 무조건 password를 바꿔야한다.
- password가 바뀌었기 때문에 반드시 application.properties에 neo4j 설정을 해줘야 한다.
```
spring.data.neo4j.password=1111
spring.data.neo4j.username=neo4j
```
<br>

### Neo4j 실습
- Account 클래스 생성
```java
@NodeEntity
public class Account {

    @Id @GeneratedValue
    private Long id;
    private String username;
    private String email;

    ...getter and setter...
}
```
- Neo4jRunner 생성
```java
@Component
public class Neo4jRunner implements ApplicationRunner {

    @Autowired
    SessionFactory sessionFactory;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Account account = new Account();
        account.setEmail("hayoung@gmail.com");
        account.setUsername("hayoung");

        Session session = sessionFactory.openSession();
        session.save(account); 
        sessionFactory.close();

        System.out.println("finished");
    }
}
```
- 관계 만들기
```java
@NodeEntity
public class Role {

    @Id @GeneratedValue
    private Long id;
    private String name;

    ...getter and setter...
}
```
- Role을 Acoount에게 주기
```java
@NodeEntity
public class Account {

    @Id @GeneratedValue
    private Long id;
    private String username;
    private String email;

    @Relationship(type = "has")
    private Set<Role> roles = new HashSet<>();

    ...getter and setter...
}
```
- Runner에서 Role을 만들고 admin 추가
```java
@Component
public class Neo4jRunner implements ApplicationRunner {

    @Autowired
    SessionFactory sessionFactory;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Account account = new Account();
        account.setEmail("hayoung@gmail.com");
        account.setUsername("hayoung");

        //Role을 만들고 admin 추가
        Role role = new Role();
        role.setName("admin"); 

        account.getRoles().add(role);   // Role 셋팅

        Session session = sessionFactory.openSession();
        session.save(account);
        sessionFactory.close();

        System.out.println("finished");
    }
}
```
- Repository를 만들어 사용하는 방법
    * AccountRepository 생성
    * Repository를 사용하는 코드
```java
public interface AccountRepository extends Neo4jRepository<Account, Long> {

}
```
```java
@Component
public class Neo4jRunner implements ApplicationRunner {

    @Autowired
    AccountRepository accountRepository;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Account account = new Account();
        account.setEmail("aaaa@gmail.com");
        account.setUsername("aaaa");

        Role role = new Role();
        role.setName("user");

        account.getRoles().add(role);

        accountRepository.save(account);

        System.out.println("finished");
    }
}
```