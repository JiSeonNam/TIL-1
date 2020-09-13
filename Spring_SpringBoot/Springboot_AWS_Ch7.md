## 7. AWS 서버 환경을 만들어보자 - AWS RDS
- RDS
    * AWS에서 지원하는 클라우드 기반 관계형 데이터베이스
    * 하드웨어 프로비저닝, 데이터베이스 설정, 패치 및 백업과 같이 잦은 운영 작업을 자동화하여 개발자가 개발에 집중할 수 있게 지원하는 서비스이다.
    * 모니터링, 알람, 백업, HA구성 등 또한 지원한다.

### 7-1. RDS 인스턴스 생성하기
- RDS를 검색하고 RDS 대시보드에서 데이터베이스 생성 버튼을 클릭한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch7_1.jpg"></p>

- DB 엔진 옵션과 템플릿 선택
    * 각각 MariaDB와 프리 티어 선택한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch7_2.jpg"></p>
 
- 상세 설정에서 스토리지를 20으로 입력하고 DB인스턴스와 마스터 사용자 정보를 등록한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch7_3.jpg"></p>

- 네트워크 및 보안
    * 퍼블릭 엑세스를 예로 변경한다.
    * 이후 보안 그룹에서 지정된 IP만 접근하도록 막을 예정
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch7_4.jpg"></p>

- 데이터베이스 옵션 
    * 데이터베이스 이름을 작성하고 포트는 3306으로 입력
    * 파라미터 그룹의 변경을 이후에 따로 진행하므로 기본으로 놔둔다.
<br>

### 7-2. RDS 운영환경에 맞는 파라미터 설정하기
- RDS를 처음 생성하면 필수로 해야하는 설정들이 있다.
    * 타임존
    * Character Set
    * Max Connection
- 카테고리에서 파라미터 그룹을 선택한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch7_5.jpg"></p>

- 파라미터 그룹 생성을 클릭하고 DB엔진을 선택하는 항목에서 전에 생성한 MaridDB와 같은 버전을 맞춰야 한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch7_6.jpg"></p>

- 생성이 완료되면 해당 파라미터 그룹을 클릭하고 파라미터를 편집한다.
    * 각 항목들을 검색해서 편집한다.
    * time_zone : Asia/Seoul
    * character-set-client : utf8mb4
    * character-set-connection : utf8mb4
    * character-set-database : utf8mb4
    * character-set-filesystem : utf8mb4
    * character-set-results : utf8mb4
    * collation_connection : utf8mb4
    * collation_server : utf8mb4
    * max_connections : 150

- 완성된 파라미터 그룹을 데이터베이스에 연결한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch7_7.jpg"></p>

- 데이터베이스 옵션 항목에서 DB 파라미터 그룹을 default에서 방금 생성한 신규 파라미터 그룹으로 변경한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch7_8.jpg"></p>

- 저장을 누르고 수정사항 요약을 즉시 적용으로 한다.
    * 예약된 다음 유지 관리 기간에 적용하면 지금 하지 않고, 새벽 시간대에 진행된다.
    * 수정사항이 반영되는 동안 데이터베이스가 작동하지 않을 수 있으므로 예약 시간을 걸어두는 의미지만 서비스가 오픈되지 않았기 때문에 즉시 적용한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch7_9.jpg"></p>

- 간혹 파라미터 그룹이 제대로 반영되지 않을 때가 있어 정상 적용을 위해 재부팅을 진행한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch7_10.jpg"></p>
<br>

### 7-3. 내 PC에서 RDS 접속하기
- 로컬 PC에서 RDS로 접근하기 위해 RDS의 보안 그룹에 본인 PC의 IP를 추가한다.
    * RDS 세부정보 페이지 - 보안 그룹
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch7_11.jpg"></p>

- 브라우저를 새로 열어 보안 그룹 목록 중 EC2에 사용된 보안 그룹의그룹 ID를 복사한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch7_12.jpg"></p>

- 복사된 보안 그룹 ID와 본인의 IP를 RDS보안 그룹의 인바운드로 추가한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch7_13.jpg"></p>
<br>

#### 7-3-1. 로컬에서 테스트하기
- 실습 기준 IntelliJ Community 버전에서 Database Navigator 플러그인 설치
- Database Navigator를 실행하고 RDS 접속 정보 등록을 한다.
    * Host에는 RDS의 엔드 포인트를 등록한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch7_14.jpg"></p>

- Test Connection을 통해 연결 테스트를 하고 SQL을 실행할 콘솔창을 연다.
- 쿼리가 수행될 database를 선택
    * 만약 database명을 잊었다면 Schema 항목에 추가되어 있다.
```sql
use AWS RDS 웹 콘솔에서 지정한 데이터베이스명;
```
- 현재의 charater_set, collation 설정을 확인한다.
```sql
show variables like 'c%';
```
- 만약 utfmb4가 적용이 안된 필드가 있다면 다음과 같이 직접 변경한다.
```sql
ALTER DATABASE 데이터베이스명
CHARACTER SET = 'utf8mb4'
COLLATE = 'utf8mb4_general_ci';
```
- 타임존 확인
```sql
select @@time_zone, now();
```
- 테이블 생성 및 insert 쿼리 실행
```sql
CREATE TABLE test (
   id bigint(20) NOT NULL AUTO_INCREMENT,
   content varchar(255) DEFAULT NULL,
   PRIMARY KEY (id)
) ENGINE=InnoDB;
```
```sql
insert into test(content) values ('테스트');
```
```sql
select * from test;
```
<br>

### 7-4. EC2에서 RDS 접근 확인
- putty에서 EC2에 ssh 접속을 진행한다. 
- MySQL설치
```
sudo yum install mysql
```
- MySQL에 계정, 비밀번호, 호스트 주소를 사용해 RDS에 접속한다.
```
mysql -u 계정 -p -h Host주소
```
- RDS에 접속되었으면 실제로 생성한 RDS가 맞는지 확인
```sql
show database
```