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

## SMTP 설정
- SMTP 서버와 연결해서 실제 이메일 전송
    * Gmail
- Gmail 서버에 App Passwords 발급받기
    * 2차 인증을 설정하고
    * 앱 비밀번호를 설정하면 된다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/db_email_setting_1.jpg"></p>

- application-dev.properties 파일에 설정
```properties
...

# SMTP
spring.mail.host=smtp.gmail.com
spring.mail.port=587
spring.mail.username=khy07181@gmail.com
spring.mail.password=hgiauoajiemdumcu
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.timeout=5000
spring.mail.properties.mail.smtp.starttls.enable=true
```
- 실행해서 가입하면 다음과 같이 실제 이메일로 인증 메일이 온다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/db_email_setting_2.jpg"></p>

- 그러나 Gmail SMTP는 하루에 보낼 수 있는 이메일이 한정되어 있어 실제 운영환경에 쓰기에는 적합하지 않다.
- 대체 서비스 
    * [SendGrid](https://sendgrid.com/)
    * [mailgun](https://www.mailgun.com/)
    * [Amazon](https://aws.amazon.com/ko/ses/)
    * [Google G Suite](https://workspace.google.com/)
<br>
