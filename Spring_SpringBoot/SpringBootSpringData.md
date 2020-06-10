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
