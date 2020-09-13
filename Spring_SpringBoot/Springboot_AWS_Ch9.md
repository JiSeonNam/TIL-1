## 9. 코드가 푸시되면 자동으로 배포하기(Travis CI 배포 자동화)

### 9-1. Travis CI 연동하기

#### TRavis CI 웹 서비스 설정
- [Travis](https://travis-ci.org/)에서 깃허브 계정으로 로그인 한뒤 Setting에서 저장소 이름을 검색한 뒤 상태바를 활성화 시킨다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_39.jpg"></p>
<br>

#### 프로젝트 설정
- travis CI의 상세한 설정은 프로젝트에 존재하는 .travis.yml파일로 할 수 있다.
- 프로젝트의 build.gradle과 같은 위치에 .travis.yml파일을 생성하고 다음과 같은 코드를 추가한다.
```yml
language: java
jdk:
  - openjdk11

# Travis Ci를 어느 브랜치가 푸시될 때 수행할지 지정
branches: 
  only:
    - master

# Travis CI 서버의 Home
# 그레이들을 통해 의존성을 받게 되면 이를 해당 디렉토리에 캐시하여, 같은 의존성은 다음 배포 때부터 다시 받지 않도록 설정
cache:
  directories:
    - '$HOME/.m2/repository'
    - '$HOME/.gradle'

# master 브랜치에 푸시되었을 때 수행하는 명령어로 여기서는 프로젝트 내부에 둔 gradlew을 통해 clean & build를 수행
script: "./gradlew clean build"

# CI 실행 완료 시 메일로 알람
notifications:
  email:
    recipients:
      - khy07181@gmail.com 
```
- master 브랜치에 커밋과 푸시하고 Travis CI 저장소 페이지를 확인하고 이메일을 확인하면 빌드가 성공했다고 알려준다.
<br>

### 9-2. Travis CI와 AWS S3 연동하기
- S3는 AWS에서 제공하는 일종의 파일 서버이다.
- 실제 배포는 AWS의 CodeDeploy라는 서비스를 이용한다.
- Jar파일을 전달하기 위해 S3연동을 먼저하며, CodeDeploy는 저장 기능이 없기 때문에 Travis CI가 빌드한 결과물을 받아서 CodeDeploy가 가져갈 수 있도록 보관할 수 있는 공간이 필요해서 AWS S3를 사용한다.
<br>

#### AWS Key 발급
- 일반적으로 AWS 서비스에 외부 서비스가 접근할 수 없기 때문에 접근 가능한 권한을 가진 Key를 생성해서 사용해야 한다.
    * AWS에서는 이러한 인증과 관련된 기능을 제공하는 서비스로 IAM이 있다.
- IAM은 AWS에서 제공하는 서비스의 접근 방식과 권한을 관리한다. 
- IAM을 통해 Travis CI가 AWS의 S3와 CodeDeply에 접근할 수 있도록 한다.
- AWS에서 IAM을 검색하고 사용자 추가
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_40.jpg"></p>

- 생성할 사용자의 이름과 엑세스 유형을 선택한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_41.jpg"></p>

- 권한 설정 방식은 기존 정책 직접 연결을 선택하고 정책 검색 화면에서 s3full과 CodeDeployFull을 검색하여 각각 체크한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_42.jpg"></p>

- 태그 추가에서는 Name값을 본인이 인지 가능한 정도의 이름으로 만든다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_43.jpg"></p>

- 권한 최종 확인
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_44.jpg"></p>

- 최종 생성이 완료되면 엑세스 키와 비밀 엑세스 키가 생성된다.
    * 이 두 값이 Travis CI에서 사용될 Key이다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_45.jpg"></p>
<br>

#### Travis CI에 Key 등록
- Travis 설정 화면에서 Environment Variables 항목에 AWS_ACESS_KEY, AWS_SECRET_KEY를 변수로 IAM 사용자에서 발급받은 키 값들을 등록한다.
    * AWS_ACESS_KEY : 엑세스 키 ID
    * AWS_SECRET_KEY : 비밀 엑세스 키
    * 등록하면 이제 .travis.yml에서 `$AWS_ACESS_KEY`, `$AWS_SECRET_KEY`로 사용할 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_46.jpg"></p>
<br>

#### S3 버킷 생성
 - AWS에서 S3를 검색 후 버킷 만들기를 선택한다.
 - 원하는 버킷명을 작성한다.
    * 이 버킷에 배포할 Zip 파일이 모여있ㄴ튼 장소임을 의미하도록 짓는 것이 좋다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_47.jpg"></p>

- 버전관리 설정은 별다른 설정 할 것 없이 넘어가고 버킷의 보안과 권한 설정 부분에서 모든 차단을 해야 한다.
    * 실제 서비스를 햘 경우 Jar 파일이 public일 경우 누구나 내려받을 수 있어 코드나 설정값, 주요 키값들이 발취될 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_48.jpg"></p>

- 버킷이 생성되면 다음과 같이 버킷 목록에서 확인할 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_49.jpg"></p>

#### .travis.yml 수정
- Travis CI에서 빌드하여 만든 Jar 파일을 S3에 올릴 수 있도록 다음 코드를 .travis.yml에 추가한다.
```yml
# ...

# deploy 명령어가 실행되기 전에 수행되며 CodeDeploy는 Jar 파일을 인식하지 못하므로 Jar+기타 설정 파일들을 모아 압축한다.
before_deploy:
  - zip -r aws-springboot2-webservice *
  - mkdir -p deploy
  - mv aws-springboot2-webservice.zip deploy/aws-springboot2-webservice.zip

# S3로 파일 업로드 혹은 CodeDeploy로 배포 등 외부 서비스와 연동될 행위들을 선언한다.
deploy:
  - provider: s3
    access_key_id: $AWS_ACCESS_KEY # Travis repo settings에 설정된 값
    secret_access_key: $AWS_SECRET_KEY # Travis repo settings에 설정된  값
    bucket: aws-springboot2-build # S3 버킷
    region: ap-northeast-2
    skip_cleanup: true
    acl: private # zip 파일 접근을 private으로
    local_dir: deploy # before_deploy에서 생성한 디렉토리
    wait-until-deploy: true

# ...
```
- 설정이 완료되면 깃허브로 푸시한다. Travis CI에서 자동으로 진행되는 것과 모든 빌드과 성공하는지 확인한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_50.jpg"></p>

- S3 버킷에도 업로드가 성공한 것을 확인할 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_51.jpg"></p>
<br>

### 9-4. Travis CI와 AWS S3, CodeDeploy 연동하기
- AWS의 배포 시스템인 CodeDeploy를 이용하기 전에 배포 대상인 EC2가 CodeDeploy를 연동 받을 수 있게 IAM 역할을 하나 생성한다.
<br>

#### EC2에 IAM 역할 추가하기
- IAM을 검색하고 역할 -> 역할만들기를 차례로 클릭한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_52.jpg"></p>

- 서비스 선택에서 AWS 서비스, EC2를 선택한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_53.jpg"></p>

- 정책에선 EC2RoleForA를 검색하여 AmazonEc2RoleforAWS-CodeDeploy를 선택한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_54.jpg"></p>

- 태그는 원하는 이름으로 만든다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_55.jpg"></p>

- 마지막으로 역할의 이름을 등록하고 나머지 등록 정보를 최종적으로 확인한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_56.jpg"></p>

- 만든 역할을 EC2 서비스에 등록하기
    * EC2 인스턴스 목록이르 이동하고 인스턴스를 다음과 같이 선택한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_57.jpg"></p>

- 방금 생성한 역할을 선택한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_58.jpg"></p>

- 역할 선택이 완료되면 해당 EC2 인스턴스를 재부팅한다.
    * 재부팅해야만 역할이 정상적으로 적용된다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_59.jpg"></p>
<br>

#### CodeDeploy 에이전트 설치
- EC2 서버에 접속해 다음 명령어를 입력한다.
```
aws s3 cp s3://aws-codedeploy-ap-northeast-2/latest/install . --region ap-northeast-2
```
- install 파일에 실행권한을 추가한다.
```
chmod +x ./install
```
- insatll 파일로 설치 진행하기
```
sudo ./install auto
```
- 설치가 끝나면 Agent가 정상적으로 실행되고 있는지 상태 검사를 한다.
    * `The AWS CodeDeploy angent is running as PID XXX`라고 메세지가 출력되면 정상이다.
```
sudo service codedeploy-agent status
```
<br>

#### CodeDeploy를 위한 권한 생성
- CodeDeploy에서 EC2의 접근도 권한이 필요하므로 IAM 역할을 생성한다.
- 서비스는 AWS 서비스 - CodeDeploy를 차례로 선택한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_60.jpg"></p>

- CodeDeploy는 권한이 하나뿐이라서 선택 없이 다음으로 넘어간다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_61.jpg"></p>

- 태그도 원하는 이름으로 짓고 생성완료한다.
<br>

#### CodeDeploy 생성
- CodeDeploy 서비스로 이동해 애플리케이션 생성을 한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_62.jpg"></p>

- 생성할 CodeDeploy 이름과 컴퓨팅 플랫폼을 선택한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_63.jpg"></p>

- 생성이 완료되면 배포 그룹 생성을 한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_64.jpg"></p>

- 배포 그룹 이름과 서비스 역할 등록한다. 
    * 서비스 역할은 CodeDeploy용 IAM 역할을 선택하면 된다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_65.jpg"></p>

- 배포 유형에서는 현재 위치를 선택한다.
    * 배포할 서비스가 2대 이상이라면 블루/그린 선택
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_66.jpg"></p>

- 환경 구성에서는 Amazon EC2 인스턴스에 체크한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_67.jpg"></p>

- 마지막으로 배포 설정과 로드 밸런서를 선택한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_68.jpg"></p>
<br>

#### Travic CI, S3, CodeDeploy 연동
- EC2 서버에 S3에서 넘겨줄 zip 파일을 저장할 디렉토리 생성
```
mkdir ~/app/step2 && mkdir ~/app/step2/zip
```
- 프로젝트에 AWS CodeDeploy 설정을 위해 appspec.yml 작성
```yml
version: 0.0
os: linux
files:
  - source: /
    destination: /home/ec2-user/app/step2/zip/
    overwrite: yes 

```
- Travis CI 설정을 위해 .travis.yml에 코드 추가
```yml
# ...

  - provider: codedeploy
    access_key_id: $AWS_ACCESS_KEY # Travis repo settings에 설정된 값
    secret_access_key: $AWS_SECRET_KEY # Travis repo settings에 설정된  값
    bucket: aws-springboot2-build # S3 버킷
    key: aws-springboot2-webservice.zip # 빌드 파일을 압축해서 전달
    bundle_type: zip
    application: aws-springboot2-webservice # 웹 콘솔에서 등록한 CodeDeploy 애플리케이션
    deployment_group: aws-springboot2-webservice-group # 웹 콘솔에서 등록한 CodeDeploy 배포 그룹
    region: ap-northeast-2
    wait-until-deployed: true

# ...
```
- 프로젝트를 커밋하고 푸시한다.
- Travis Ci가 끝나면 CodeDeploy 화면에서 배포가 수행되는 것을 확인할 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_69.jpg"></p>

- 배포가 끝나면 EC2 서버에서 파일들이 잘 도착했는지 확인
```
cd /home/ec2-user/app/step2/zip
```
- `ll`명령어를 사용해 파일들을 확인한다.
<br>

### 9-5 배포 자동화 구성

#### Travis CI, S3, CodeDeploy 연동까지 구현 후 실제로 Jar를 배포하여 실행하기
- deploy.sh 파일 추가
    * 프로젝트의 scripts 디렉토리를 생성하고 여기에 스크립트를 생성한다.
```sh
#!/bin/bash

REPOSITORY=/home/ec2-user/app/step2
PROJECT_NAME=aws-springboot2-webservice

echo "> Build 파일 복사"

cp $REPOSITORY/zip/*.jar $REPOSITORY/

echo "> 현재 구동 중인 애플리케이션 pid 확인"

CURRENT_PID=$(pgrep -fl aws-springboot2-webservice | grep jar | awk '{print $1}')

echo "현재 구동 중인 애플리케이션 pid: $CURRENT_PID"

if [ -z "$CURRENT_PID" ]; then
    echo "> 현재 구동 중인 애플리케이션이 없으므로 종료하지 않습니다."
else
    echo "> kill -15 $CURRENT_PID"
    kill -15 $CURRENT_PID
    sleep 5
fi

echo "> 새 애플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/*.jar | tail -n 1)

echo "> JAR Name: $JAR_NAME"

echo "> $JAR_NAME 에 실행권한 추가"

chmod +x $JAR_NAME

echo "> $JAR_NAME 실행"

nohup java -jar \
    -Dspring.config.location=classpath:/application.properties,classpath:/application-real.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
    -Dspring.profiles.active=real \
    $JAR_NAME > $REPOSITORY/nohup.out 2>&1 & 
```
- .travis.yml 파일 수정
```yml
# ...

before_deploy:
  - mkdir -p before-deploy # zip에 포함시킬 파일들을 담을 디렉토리 생성
  - cp scripts/*.sh before-deploy/
  - cp appspec.yml before-deploy/
  - cp build/libs/*.jar before-deploy/
  - cd before-deploy && zip -r before-deploy * # before-deploy로 이동 후 전체 압축
  - cd ../ && mkdir -p deploy # 상위 디렉토리로 이동 후 deploy 디렉토리 생성
  - mv before-deploy/before-deploy.zip deploy/aws-springboot2-webservice.zip # deploy로 zip파일 이동

# ...
```
- appspec.yml 파일 수정
```yml
# ...

permissions:
  - object: /
    pattern: "**"
    owner: ec2-user
    group: ec2-user

hooks:
  ApplicationStart:
    - location: deploy.sh
      timeout: 60
      runas: ec2-user 
```
- 모든 설정이 완료되면 깃허브로 커밋과 푸시해서 Travis와 CodeDeploy의 성공 메세지를 확인한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_70.jpg"></p>
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_71.jpg"></p>
<br>

- 웹 브라우저에서 EC2 도메인을 입력해서 확인하기
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_72.jpg"></p>
<br>

#### 실제 배포 과정 체험
- build.gradle의 프로젝 버전을 변경한다.
```gradle
version '1.0.1-SNAPSHOT'
```
- 변경 내용을 알 수 있게 index.mustache에 Ver.2 테스트를 추가한다.
```html
<h1>스프링 부트로 시작하는 웹 서비스 Ver.2</h1>
```
- 깃허브로 커밋과 푸시하면 다음과 같이 변경된 코드가 배포된 것을 확인할 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_73.jpg"></p>
<br>

### 9-6 CodeDeploy 로그 확인
- CodeDeploy와 같이 AWS가 지원하는 서비스에서 오류가 발생하면 오류를 해결하기 힘들다. 
- 따라서 배포가 실패하면 로그를 보고 오류를 해결하는 방법이 좋다.
- CodeDeploy의 대부분 내용은 /opt/codedeploy-agent/deployment-root에 있다.
- 이 디렉토리로 이동한 뒤 목록을 확인하면 로그를 확인할 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_74.jpg"></p>