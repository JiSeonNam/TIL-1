# DB와 이메일 설정
- 지금까지 DB는 인메모리 데이터베이스를 썼고 기본적인 profile을 local로 사용했다.
- 이메일 관련 기능 또한 Profile이 local일때 ConsoleMailSender 사용해서 처리했었다.
- 이제 DB를 PostgreSQL을 사용하도록 바꾸고 
- JavaMailSender 또한 실제 이메일을 사용하도록 한다.
<br>

## PostgreSQL 설치 및 설정
- [PostgreSQL 설치](https://www.postgresql.org/download/)
- [psql 접속](https://www.postgresqltutorial.com/connect-to-postgresql-database/)
- DB와 유저(role) 만들고 유저에게 권한 할당하기
    * `create database testdb;`
    * `create user testuser with encrypted password 'testpass';`
    * `grant all privileges on database testdb to testuser;`
- application-dev.properties에 DB 정보 설정
    * dev Profile용 설정 파일
```properties
# dev 모드이므로 create-drop이 아닌 update를 사용한다.
spring.jpa.hibernate.ddl-auto=update

spring.datasource.url=jdbc:postgresql://localhost:5432/testdb
spring.datasource.username=testuser
spring.datasource.password=testpass
```
<br>
