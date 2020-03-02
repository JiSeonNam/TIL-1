> [신입 프로그래머를 위한 실전 JSP 강좌 강의](https://www.inflearn.com/course/%EC%8B%A4%EC%A0%84-jsp-%EA%B0%95%EC%A2%8C/dashboard)를 듣고 공부한 내용을 정리하여 기록

<br>

### JDBC
- JAVA 프로그램에서 SQL문을 실행하여 데이터를 관리하기 위한 JAVA API이다.
- 다양한 데이터 베이스에 대해서 별도의 프로그램을 만들 필요 없이, 해당 데이터 베이스의 JDBC를 이용하면 하나의 프로그램으로 데이터베이스 관리할 수 있다.
<br>

### 데이터베이스 연결 순서
- JDBC 드라이버 로드 - DriverManager
	* Class.forName("oracle.jdbc.driver.OracleDriver"); - 메모리에 OracleDriver가 로드된다.
	<br>
- 데이터베이스 연결 - Connection
	* DriverManager.getConnection(JDBC URL, 계정아이디, 비밀번호); - Connection 객체 생성
	<br>
- SQL문 실행 - Statement
	* connection.createStatement( ); - Statement객체를 통해 SQL문이 실행된다.
	<br>
- 데이터베이스 연결 해제 - ResultSet
	* statement.executeQuery( ), statement.executeUpdate( ) - SQL문의 결과값을 ResultSet객체로 받는다.
	* statement.executeQuery( ) : SQL문 실행 후 여러 개의 결과값이 생기는 경우 사용. 반환형은 ResultSet
		ex) select
	* statement.executeUpdate( ) : SQL문 실행 후 테이블의 내용만 변경되는 경우 사용. 반환형은 int
		ex) insert, delete, update
<br>

### DAO와 DTO
- DAO(Data Access Object)
	* 데이터베이스에 접속해서 데이터 추가, 삭제, 수정 등의 작업을 하는 클래스
	* 일반적인 JSP 혹은 Servlet 페이지 내에 기술할 수도 있지만, 유지보수 및 코드의 모듈화를 위해 별도의 	DAO 클래스를 만들어 사용한다.
	<br>
- DTO(Data Transfer Object)
	* DAO클래스를 이용하여 데이터베이스에서 데이터를 관리할 때 데이터를 일반적인 변수에 할당하여 사용할 수도 있지만, 해당 데이터의 클래스를 만들어서 사용한다.
<br>

### PreparedStatement 객체
- SQL문 실행을 위해 사용하던 Statement 객체의 경우 중복 코드가 많아지는 단점이 있다.
- PreparedStatement 객체를 사용하면 이 단점을 개선할 수 있다.
```java
Class.forName(driver);
connection = DriverManager.getConnection(url,uid,upw);
int n;
String query = "insert into memberforpre (id, pw, name, phone) values (?,?,?,?)";
preparedStatement = connection.preparedStatement(query);

preparedStatement.setString(1, "abc");
preparedStatement.setString(2, "123");
preparedStatement.setString(3, "홍길동");
preparedStatement.setString(4, "010-1234-5678");
n = preparedStatement.executeUpdate();
```
<br>

### 커넥션 풀(DBCP : Database Connection Pool)
- 클라이언트에서 다수의 요청이 발생할 경우 데이터베이스에 부하가 발생하게 되는데
- 이러한 문제를 해결하기 위해 커넥션 객체를 미리 생성해 놓는 커넥션 풀 기법을 이용한다.
- 커넥션 풀(DBCP)의 사용방법 : tomcat 컨테이너가 데이터베이스 인증을 하도록 context.xml 파일에 아래의 코드를 추가한다.
```java
<Resource
	auth = "Container"driverClassName = "oracle.jdbc.driver.OracleDriver"
	url = "jdbc:oracle:thin:@localhost:1521:xe"
	username = "scott"
	password = "tiger"
	name = "jdbc/Oracle11g"
	type = "javax.sql.DataSource"
	maxActive = "50"		<!-- 미리 만들어 놓을 커넥션 풀의 갯수 -->
	maxWait = "1000"		<!-- 커넥션풀을 다 사용했을 경우 기다려야할 시간 설정 -->
/>
```
