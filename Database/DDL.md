## 데이터 정의어(Data definition language, DDL)
- 데이터베이스의 스키마 객체를 생성, 변경, 제거 하는 일을 수행한다.

### 테이블 생성
- 데이터 형 외에도 속성값의 빈 값 허용 여부는 NULL 또는 NOT NULL로 설정
- DEFAULT 키워드와 함께 입력하지 않았을 때의 초기값을 지정
- 입력하지 않고 자동으로 1씩 증가하는 번호를 위한 AUTO_INCREMENT
```sql
create table 테이블명( 
    필드명1 타입 [NULL | NOT NULL][DEFAULT ][AUTO_INCREMENT], 
    필드명2 타입 [NULL | NOT NULL][DEFAULT ][AUTO_INCREMENT], 
    필드명3 타입 [NULL | NOT NULL][DEFAULT ][AUTO_INCREMENT], 
    ........... 
    PRIMARY KEY(필드명) 
    );
```
<br>

### 테이블 수정(추가/삭제)
```sql
alter table 테이블명
    add  필드명 타입 [NULL | NOT NULL][DEFAULT ][AUTO_INCREMENT];

alter table 테이블명
    drop  필드명;
```

### 컬럼 수정
- change 키워드를 사용하고  칼럼을 새롭게 재정의 (이름부터 속성까지 전부)
```sql
alter table  테이블명
    change  필드명  새필드명 타입 [NULL | NOT NULL][DEFAULT ][AUTO_INCREMENT];
```
- ex) EMPLOYEE2 테이블의 부서번호(deptno)를 dept_no로 수정
    * alter table EMPLOYEE2 change deptno dept_no int(11);

### 테이블 이름 변경
- `alter table 테이블명 rename 변경이름`

### 테이블 삭제
- `drop table 테이블명`
