## SQL
- SQL은 데이터를 보다 쉽게 검색하고 추가, 삭제, 수정 같은 조작을 할 수 있도록 고안된 컴퓨터 언어이다.
- 관계형 데이터베이스에서 데이터를 조작하고 쿼리하는 표준 수단이다.
- DML (Data Manipulation Language): 데이터를 조작하기 위해 사용
    * INSERT, UPDATE, DELETE, SELECT 등
- DDL (Data Definition Language): 데이터베이스의 스키마를 정의하거나 조작하기 위해 사용
    * CREATE, DROP, ALTER 등
- DCL (Data Control Language) : 데이터를 제어하는 언어
    * 권한을 관리하고, 테이터의 보안, 무결성 등을 정의
    * GRANT, REVOKE 등
<br>

### Database 생성
```sql
mysql> create database DB이름;
```
<br>

## Database 사용자 생성과 권한 주기
- Database를 생성했다면, 해당 데이터베이스를 사용하는 계정을 생성해야 한다.
- 또한, 해당 계정이 데이터베이스를 이용할 수 있는 권한을 줘야 한다.
- db이름 뒤의 * 는 모든 권한을 의미한다.
- `@’%’`는 어떤 클라이언트에서든 접근 가능하다는 의미이고, `@’localhost’`는 해당 컴퓨터에서만 접근 가능하다는 의미이다.
- `flush privileges`는 DBMS에게 적용을 하라는 의미로 반드시 실행해줘야 한다.
```sql
grant all privileges on db이름.* to 계정이름@'%' identified by ＇암호’;
grant all privileges on db이름.* to 계정이름@'localhost' identified by ＇암호’;
flush privileges;
```
<br>

### 생성한 Database에 접속하기
```sql
mysql –h호스트명 –uDB계정명 –p 데이터베이스이름
```
<br>

### MySQL 연결끊기
```sql
mysql> QUIT 
또는
mysql> exit
```
<br>

### DBMS에 존재하는 데이터베이스 확인하기
```sql
mysql> show databases;
```
<br>

### 사용중인 데이터베이스 전환하기
```sql
mysql> use mydb;
```
<br>

### 데이터를 저장하는 공간 테이블(Table)
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Database/img/SQL_1.png" width="600px"></p>
- 테이블 : RDBMS의 기본적 저장구조 한 개 이상의 column과 0개 이상의 row로 구성
- 열(Column) : 테이블 상에서의 단일 종류의 데이터를 나타냄. 특정 데이터 타입 및 크기를 가지고 있다.
- 행(Row) : Column들의 값의 조합. 레코드라고 불림. 기본키(PK)에 의해 구분. 기본키는 중복을 허용하지 않으며 없어서는 안 된다.
- Field : Row와 Column의 교차점으로 Field는 데이터를 포함할 수 있고 없을 때는 NULL 값을 가지고 있다.
<br>

### 현재 데이터베이스에 존재하는 테이블 목록 확인하기
```sql
mysql> show tables;
```
<br>

### 테이블 구조를 확인하기 위한 DESCRIBE 명령
```sql
mysql> desc EMPLOYEE;
```
<br>