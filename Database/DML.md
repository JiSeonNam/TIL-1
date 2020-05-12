## 데이터 조작어(Data Manipulation Language, DML)
- SELECT – 검색
- INSERT - 등록
- UPDATE - 수정
- DELETE - 삭제
<br>

### SELECT
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Database/img/DML_1.png" width="600px"></p>

- 전체 데이터 검색 : * 
    * `SELECT * FROM  DEPARTMENT;`
- 특정 컬럼 검색
    * SELECT 뒤에 콤마(,)로 구별해서 나열
    * `select empno, name, job from employee;`
- 컬럼에 Alias(별칭) 부여
    * `select empno as 사번, name as 이름, job as 직업 from employee;`
- 컬럼의 합성
    * `SELECT concat( empno, '-', deptno) AS '사번-부서번호' FROM employee;`
- 중복행의 제거
    * 중복되는 행이 출력되는 경우, DISTINCT 키워드로 중복행을 제거
    * `select distinct deptno from employee;`
<br>

### ORDER BY
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Database/img/DML_2.png" width="600px"></p>

- 정렬하기
    * `select empno, name, job from employee order by name;`
    * `select empno as 사번, name as 이름, job as 직업 from employee order by 이름;`
    * `select empno, name, job from employee order by name desc;`
<br>

### WHERE
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Database/img/DML_3.png" width="600px"></p>

- 산술비교 연산자
    * `select name, hiredate from employee where hiredate < '1981-01-01';`
- 논리 연산자
    * `select name, deptno from employee where deptno = 30;`
- IN 키워드
    * employee 테이블에서 부서번호가 10 또는 30인 사원이름과 부서번호
    * `select name, deptno from employee where deptno in (10, 30);`
- LIKE 키워드
    * 와일드 카드를 사용하여 특정 문자를 포함한 값에 대한 조건을 처리
    * % 는 0에서부터 여러 개의 문자열을 나타냄
    * _ 는 단 하나의 문자를 나타내는 와일드 카드
    * employee 테이블에서 이름에 'A'가 포함된 사원의 이름(name)과 직업(job)
    * `select name, job from employee where name like '%A%';`
<br>

### CAST 형변환
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Database/img/DML_4.png" width="600px"></p>
<br>

### 그룹함수
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Database/img/DML_5.png" width="600px"></p>

- `SELECT AVG(salary) , SUM(salary) FROM employee WHERE deptno = 30;`
- GROUP BY
    * `SELECT deptno, AVG(salary) , SUM(salary) FROM employee group by deptno;`
 <br>
   
### 데이터 입력(INSERT)
- 필드명을 지정해주는 방식은 디폴트 값이 세팅되는 필드는 생략할 수 있다.
- 필드명을 지정해주는 방식은  추 후, 필드가 추가/변경/수정 되는 변경에 유연하게 대처 가능하다.
- 필드명을 생략했을 경우에는 모든 필드 값을 반드시 입력해야 한다.
- `insert into ROLE (role_id, description) values ( 200, 'CEO');`
<br>

### 데이터 수정(UPDATE)
- 조건식을 통해 특정 row만 변경할 수 있다.
- 조건식을 주지 않으면 전체 로우가 영향을 미친다.
- `update ROLE set description = 'CTO' where role_id = 200;`
<br>

### 데이터 삭제(DELETE)
- 조건식을 통해 특정 row만 삭제할 수 있다.
- 조건식을 주지 않으면 전체 로우가 영향을 미친다.
- `delete from ROLE where role_id = 200;`