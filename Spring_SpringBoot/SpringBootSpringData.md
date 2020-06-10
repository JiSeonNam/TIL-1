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