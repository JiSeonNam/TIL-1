## 8. EC2 서버에 프로젝트 배포하기

### 8-1. EC2에 프로젝트 clone 받기
- EC2에 깃 설치
```
sudo yum install git
```
- 프로젝트 clone할 디렉토리 생성
```
mkdir ~/app && mkdir ~/app/step1
```
- 디렉토리로 이동한 후 clone 진행
```
git clone 프로젝트주소
```
- 코드가 정상 수행되는지 테스트
    * 만약 gradlew 실행 권한이 없다는 메세지(`-bash: ./gradlew: Permission denied`)가 뜨면 실행 권한을 추가(`chmod +x ./gradlew`)한 뒤 다시 테스트를 수행한다.
```
./gradlew test
```
<br>

### 8-2. 배포 스크립트 만들기
- ~/app/step1/에 deploy.sh 파일 생성
```
vim ~/app/git/deploy.sh
```
- deploy.sh 파일 작성
```vim
#!/bin/bash

REPOSITORY=/home/ec2-user/app/step1
PROJECT_NAME=springboot-webservice

cd $REPOSITORY/$PROJECT_NAME/

echo "> Git Pull"

git pull

echo "> 프로젝트 Build 시작"

./gradlew build

echo "> step1 디렉토리 이동"

cd $REPOSITORY

echo "> Build 파일 복사"

cp $REPOSITORY/$PROJECT_NAME/build/libs/*.jar $REPOSITORY/

echo "> 현재 구동중인 애플리케이션 pid 확인"

CURRENT_PID=$(pgrep -f ${PROJECT_NAME}*.jar)

echo "현재 구동 중인 애플리케이션 pid: $CURRENT_PID"

if [ -z "$CURRENT_PID" ]; then
   echo "> 현재 구동 중인 애플리케이션이 없으므로 종료하지 않습니다."
else
   echo "> kill -15 $CURRENT_PID"
   kill -15 $CURRENT_PID
   sleep 5
fi

echo "> 새 애플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/ | grep *.jar | tail -n 1)

echo "> JAR Name: $JAR_NAME"

nohup java -jar $REPOSITORY/$JAR_NAME 2>&1 &
```
- 스크립트에 실행 권한 추가
```
chmod +x ./deploy.sh
```
- 스프립트 실행
    * nohup.out 파일을 열어보면 로그를 볼 수 있다.
    * `vim nohup.out`
```
./deploy.sh
```
- nohup.out에는 ClientRegistrationRepository를 찾을 수 없다는 에러가 난다. 
    * 8-3에서 고친다.
<br>

### 8-3. 외부 Security 파일 등록하기
- 에러가 난 이유는 로컬에는 application-oauth.properties가 있지만 깃에는 올라가 있지 않아 서버가 가지고 있지 않기 때문에 서버에서 직접 이 설정들을 가지고 있게 해야한다.
- app 디렉토리에 properties 파일 생성
```
vim /home/ec2-user/app/application-oauth.properties
```
- 로컬에 있는 application-oauth.properties파일 내용을 그대로 붙여넣는다.
- application-oauth.properties를 사용하도록 deploy.sh파일을 수정한다.
```vim
...
nohup java -jar \
        -Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties \
        $REPOSITORY/$JAR_NAME 2>&1 &
```
- deploy.sh를 실행해서 정상적으로 실행되는지 확인
<br>

### 8-4. 스프링 부트 프로젝트로 RDS 접근하기

#### 8-4-1. RDS 테이블 생성
- JPA가 사용될 엔티티 테이블 생성
```sql
create table posts (
    id bigint not null auto_increment,
    create_date datetime, 
    modified_date datetime, 
    author varchar(255), 
    content TEXT not null, 
    title varchar(500) not null, 
    primary key (id)
    ) engine=InnoDB;
```
```sql
create table user (
    id bigint not null auto_increment,
    create_date datetime,
    modified_date datetime,
    email varchar(255) not null, 
    name varchar(255) not null, 
    picture varchar(255), 
    role varchar(255) not null, 
    primary key (id)
    ) engine=InnoDB;
```
- 스프링 세션이 사용될 테이블 생성
```sql
CREATE TABLE SPRING_SESSION (
	PRIMARY_ID CHAR(36) NOT NULL,
	SESSION_ID CHAR(36) NOT NULL,
	CREATION_TIME BIGINT NOT NULL,
	LAST_ACCESS_TIME BIGINT NOT NULL,
	MAX_INACTIVE_INTERVAL INT NOT NULL,
	EXPIRY_TIME BIGINT NOT NULL,
	PRINCIPAL_NAME VARCHAR(100),
	CONSTRAINT SPRING_SESSION_PK PRIMARY KEY (PRIMARY_ID)
) ENGINE=InnoDB ROW_FORMAT=DYNAMIC;
```
```sql
CREATE UNIQUE INDEX SPRING_SESSION_IX1 ON SPRING_SESSION (SESSION_ID);
CREATE INDEX SPRING_SESSION_IX2 ON SPRING_SESSION (EXPIRY_TIME);
CREATE INDEX SPRING_SESSION_IX3 ON SPRING_SESSION (PRINCIPAL_NAME);
```
```sql
CREATE TABLE SPRING_SESSION_ATTRIBUTES (
	SESSION_PRIMARY_ID CHAR(36) NOT NULL,
	ATTRIBUTE_NAME VARCHAR(200) NOT NULL,
	ATTRIBUTE_BYTES BLOB NOT NULL,
	CONSTRAINT SPRING_SESSION_ATTRIBUTES_PK PRIMARY KEY (SESSION_PRIMARY_ID, ATTRIBUTE_NAME),
	CONSTRAINT SPRING_SESSION_ATTRIBUTES_FK FOREIGN KEY (SESSION_PRIMARY_ID) REFERENCES SPRING_SESSION(PRIMARY_ID) ON DELETE CASCADE
) ENGINE=InnoDB ROW_FORMAT=DYNAMIC;
```
<br>

#### 8-4-2. 프로젝트 설정
- MariaDB 드라이버를 build.gradle에 등록한다.
```gradle
compile("org.mariadb.jdbc:mariadb-java-client")
```
- 서버에서 구동될 환경 구성
    * scr/main/resources/에 application-real.properties 파일 생성
    * profile=real인 환경이 구성된다고 생각하면 된다.
    * 실제 운영될 환경이기 때문에 보안/로그상 이슈가 될 만한 설정들을 모두 제거하며 RDS환경 profile 설정이 추가된다.
```properties
spring.profiles.include=oauth,real-db
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.session.store-type=jdbc 
```
- 모든 설정이 되었다면 깃허브로 푸시한다.
<br>

#### 8-4-3. EC2 설정
- OAuth와 마찬가지로 RDS 접속 정보도 보호해야할 정보이므로 EC2 서버에 직접 설정 파일을 둔다.
- app디렉토리에 application-real-db.properties 파일 생성
```
vim ~/app/application-real-db.properties
```
- application-real-db.properties 파일 작성
```properties
spring.jpa.hibernate.ddl-auto=none
spring.datasource.url=jdbc:mariadb://rds주소:포트명(기본은 3306)/database명
spring.datasource.username=db계정
spring.datasource.password=db계정 비밀번호
spring.datasource.driver-class-name=org.mariadb.jdbc.Driver
```
- deploy.sh가 real profile을 쓸 수 있도록 수정
```vim
...
nohup java -jar \
   -Dspring.config.location=classpath:/application.properties,classpath:/application-real.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
   -Dspring.profiles.active=real \
   $JAR_NAME > $REPOSITORY/nohup.out 2>&1 &
```
- deploy.sh 파일을 실행하고 nohup.out에 성공적으로 로그가 나오는지 확인한다.
- curl 명령어로 html 코드가 정상적으로 보인다면 성공한 것이다.
```
curl localhost:8080
```
<br>

### 8-5. EC2에서 소셜 로그인하기
- EC2에 스프링 부트 프로젝트가 8080포트로 배포되었으므로 8080포트가 보안 그룹에 열려있는지 확인한다.
    * 만약 열려 있지 않다면 편집 버튼을 눌러 추가해 준다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_Springboot_AWS_Ch8_1.jpg"></p>

- 인스턴스 메뉴를 클릭하면 다음과 같이 퍼블릭 DNS를 확인할 수 있다.
    * 이 주소가 EC2에 자동으로 할당된 도메인이다.
    * 어디에서나 이 주소를 입력하면 EC2서버에 접근할 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_Springboot_AWS_Ch8_2.jpg"></p>

<br>

#### 8-5-1. 구글 로그인 서비스 등록
- 구글 웹 콘솔로 접속하여 본인의 프로젝트로 이동한 다음 API 및 서비스 -> 사용자 인증 정보로 이동한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_Springboot_AWS_Ch8_3.jpg"></p>

- OAuth 동의 화면 탭을 선택하고 승인된 도메인에 http:// 없이 EC2의 퍼블릭 DNS를 등록한다. 
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_Springboot_AWS_Ch8_4.jpg"></p>

- 사용자 인증 정보 탭을 클릭하여 본인의 서비스 이름을 선택하고 퍼블릭 DNS 주소에 :8080/login/oauth2/code/google주소를 추가하여 승인된 리디렉션 URI에 등록한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_Springboot_AWS_Ch8_5.jpg"></p>
<br>

#### 8-5-2. 네이버 로그인 서비스 등록
- 네이버 개발자 센터로 접속해서 본인의 프로젝트로 이동한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_Springboot_AWS_Ch8_6.jpg"></p>

- PC 웹 항목의 서비스 URL과 Callback URL 2개를 수정한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_Springboot_AWS_Ch8_7.jpg"></p>

- 완료하면 구글 로그인과 네이버 로그인이 정상적으로 연동 완료된다.