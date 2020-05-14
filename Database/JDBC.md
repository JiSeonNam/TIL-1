# JDBC

### JDBC(Java Database Connectivity)
- 자바를 이용한 데이터베이스 접속과 SQL 문장의 실행, 그리고 실행 결과로 얻어진 데이터의 핸들링을 제공하는 방법과 절차에 관한 규약
- 자바 프로그램 내에서 SQL문을 실행하기 위한 자바 API
- SQL과 프로그래밍 언어의 통합 접근 중 한 형태
- JAVA는 표준 인터페이스인 JDBC API를 제공한다.
- 데이터베이스 벤더, 또는 기타 써드파티에서는 JDBC 인터페이스를 구현한 드라이버(driver)를 제공한다.

### JDBC를 이용한 프로그래밍 방법
1. import java.sql.*; <br>
`import java.sql.*;`
<br>

2. 드라이버를 로드 한다. <br>
`Class.forName( "com.mysql.jdbc.Driver" );`
<br>

3. Connection 객체를 생성한다.
```java
String dburl  = "jdbc:mysql://localhost/dbName";

Connection con =  DriverManager.getConnection ( dburl, ID, PWD );
```

- ex)
```java
public static Connection getConnection() throws Exception{
	String url = "jdbc:oracle:thin:@117.16.46.111:1521:xe";
	String user = "smu";
	String password = "smu";
	Connection conn = null;
	Class.forName("oracle.jdbc.driver.OracleDriver");
	conn = DriverManager.getConnection(url, user, password);
	return conn;
}
```
4. Statement 객체를 생성 및 질의 수행
```java
// Statement 생성
Statement stmt = con.createStatement();
// 질의 수행
ResultSet rs = stmt.executeQuery("select no from user" );

/*
stmt.execute(“query”);          //any SQL
stmt.executeQuery(“query”);     //SELECT
stmt.executeUpdate(“query”);    //INSERT, UPDATE, DELETE
*/
```
5. SQL문에 결과물이 있다면 ResultSet 객체를 생성한다.
```java
ResultSet rs =  stmt.executeQuery( "select no from user" );
while ( rs.next() )
      System.out.println( rs.getInt( "no") );
```

6. 모든 객체를 닫는다.
```java
rs.close();

stmt.close();

con.close();
```
<br>

### JDBC 클래스의 생성 관계
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Database/img/JDBC_1.png" width="600px"></p>
<br>




