## 10. 24시간 365일 중단없는 서비스 만들기
- 여기서는 Nginx를 이용한 무중단 배포를 한다.
    * Nginx는 웹 서버, 리버스 프록시, 캐싱, 로드 밸런싱, 미디어 스트리밍 등을 위한 오픈소스 소프트웨어이다.
- Nginx가 가지고 있는 기능 중 리버스 프록시를 사용해 무중단 배포 환경을 구축할 예정이다.
    * 리버스 프록시란 Nginx가 외부의 요청을 받아 백엔드 서버로 요청을 전달하는 행위이다.
- 구조는 EC2 서버에 Nginx 1대와 스프링 부트 Jar 2개를 사용하는 것이다.
    * Nginx는 80(http), 433(https) 포트 할당
    * 스프링 부트 1은 8081포트로 실행
    * 스프링 부트 2는 8082포트로 실행
<br>

### 10-1. Nginx 설치와 스프링 부트 연동하기
- EC2에 접속해서 Nginx 설치
```
sudo yum install nginx
```
- Nginx 실행
```
sudo service nginx start
```
- Nginx의 포트번호를 보안 그룹에 추가
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_75.jpg"></p>

- 리다이렉션 주소 추가
    * 8080이 아닌 80포트로 주소가 변경되기 때문에 구글과 네이버 로그인에도 변경된 주소를 등록해야 한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_76.jpg"></p>
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_77.jpg"></p>

- 이제 EC2 도메인으로 접근할 때 8080포트를 제거하고 접근하면 다음과 같이 Nginx 웹 페이지를 볼 수 있다.
    * 80번 포트는 기본적으로 도메인에서 포트번호가 제거된 상태이다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_78.jpg"></p>

- Nginx와 스프링 부트 연동
    * Nginx가 현재 실행 중인 스프링 부트 프로잭트를 바라볼 수 있도록 프록시 설정을 한다.
- Nginx 설정 파일을 열고 server 아래의 location / 부분을 찾아 다음과 같이 추가한다.
```
sudo vim /etc/nginx/nginx.conf
```
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_79.jpg"></p>

- 수정이 끝나면 Nginx를 재시작한다.
```
sudo service nginx restart
```
- 브라우저로 접속해 Nginx 시작페이지를 새로고침하면 다음과 같이 Nginx가 스프링 부트 프로젝트를 프록시하는 것이 확인된다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_80.jpg"></p>

### 10-2 무중단 배포 스크립트 만들기
- 무중단 배포 스프립트 만들기 전 API 추가
    * 배포 시에 8081을 쓸지, 8082를 쓸지 판단하는 기준이 되는 API
- profile API 추가
    * 프로젝트에 ProfileController를 만들어 API 코드를 추가한다.
```java
@RequiredArgsConstructor
@RestController
public class ProfileController {
    private final Environment env;

    @GetMapping("/profile")
    public String profile() {
        List<String> profiles = Arrays.asList(env.getActiveProfiles());
        List<String> realProfiles = Arrays.asList("real", "real1", "real2");
        String defaultProfile = profiles.isEmpty()? "default" : profiles.get(0);

        return profiles.stream()
                .filter(realProfiles::contains)
                .findAny()
                .orElse(defaultProfile);
    }
}
```
- 테스트코드 작성
```java
public class ProfileControllerUnitTest {
    @Test
    public void real_profile이_조회된다() {
        //given
        String expectedProfile = "real";
        MockEnvironment env = new MockEnvironment();
        env.addActiveProfile(expectedProfile);
        env.addActiveProfile("oauth");
        env.addActiveProfile("real-db");

        ProfileController controller = new ProfileController(env);

        //when
        String profile = controller.profile();

        //then
        assertThat(profile).isEqualTo(expectedProfile);
    }

    @Test
    public void real_profile이_없으면_첫번째가_조회된다() {
        //given
        String expectedProfile = "oauth";
        MockEnvironment env = new MockEnvironment();

        env.addActiveProfile(expectedProfile);
        env.addActiveProfile("real-db");

        ProfileController controller = new ProfileController(env);

        //when
        String profile = controller.profile();

        //then
        assertThat(profile).isEqualTo(expectedProfile);
    }

    @Test
    public void active_profile이_없으면_default가_조회된다() {
        //given
        String expectedProfile = "default";
        MockEnvironment env = new MockEnvironment();
        ProfileController controller = new ProfileController(env);

        //when
        String profile = controller.profile();

        //then
        assertThat(profile).isEqualTo(expectedProfile);
    }
}
```
- /profile이 인증 없이도 호출될 수 있게 SecurityConfig 클래스에 제외 코드 추가
```java
@RequiredArgsConstructor
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    private  final CustomOAuth2UserService customOAuth2UserService;
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        CharacterEncodingFilter filter = new CharacterEncodingFilter();
        http
                .csrf().disable()
                .headers().frameOptions().disable()
                .and()
                    .authorizeRequests()
                    .antMatchers("/", "/css/**", "/images/**", "/js/**", "/h2-console/**").permitAll()
                    // "/profile"도 추가
                    .antMatchers("/", "/css/**", "/images/**", "/js/**", "/h2-console/**", "/profile").permitAll()
                    .antMatchers("/api/v1/**").hasRole(Role.USER.name())
                    .anyRequest().authenticated()
                .and()
                    .logout()
                        .logoutSuccessUrl("/")
                .and()
                    .oauth2Login()
                        .userInfoEndpoint()
                            .userService(customOAuth2UserService);
    }
}
```
- 테스트 코드 검증
```java
@RequiredArgsConstructor
@RestController
public class ProfileController {
    private final Environment env;

    @GetMapping("/profile")
    public String profile() {
        List<String> profiles = Arrays.asList(env.getActiveProfiles());
        List<String> realProfiles = Arrays.asList("real", "real1", "real2");
        String defaultProfile = profiles.isEmpty()? "default" : profiles.get(0);

        return profiles.stream()
                .filter(realProfiles::contains)
                .findAny()
                .orElse(defaultProfile);
    }
} 
```
- 테스트가 완료되면 깃허브로 푸시하여 배포한다. 
- 배포가 끝나면 브라우저에서 /profile로 접속해서 profile이 잘 나오는지 확인한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_81.jpg"></p>
<br>

#### 무중단 배포를 위한 profile 2개 추가하기
- real1, real2 profile 생성
- 프로젝트에 application-real1.properties 파일 생성
```properties
server.port=8081
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.profiles.include=oauth,real-db
spring.session.store-type=jdbc  
```
- 프로젝트에 application-real2.properties 파일 생성
```properties
server.port=8082
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.profiles.include=oauth,real-db
spring.session.store-type=jdbc  
```
<br>

#### Nginx 설정 수정
- 배포 때마다 Nginx의 프록시 설정이 교체될 수 있도록 설정 추가
- Nginx의 설정이 모여있는 /etc/nginx/conf.d/에 service-url.inc 파일 생성
```
sudo vim /etc/nginx/conf.d/service-url.inc
```
- 다음 코드를 입력한다
```
set $service_url http://127.0.0.1:8080;
```
- 해당 파일을 Nginx가 사용할 수 있게 ngix.conf 파일을 열어서 설정
```
sudo vim /etc/nginx/nginx.conf
```
- location / 부분을 찾아서 다음과 같이 변경한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Webservice_82.jpg"></p>

- Nginx 재시작
```
sudo service nginx restart
```
<br>

#### 배포 스크립트 작성
- 무중단 배포를 진행할 스크립트들은 총 5개이다.
    * stop.sh : 기존 Nginx에 연결되어 있진 않지만, 실행 중이던 스프링 부트 종료
    * start.sh : 배포할 신규 버전 스프링 부트 프로젝트를 stop.sh로 종료한 'profile'로 실행
    * health.sh : 'start.sh'로 실행시킨 프로젝트가 정상적으로 실행됐는지 체크
    * switch.sh : Nginx가 바라보는 스프링 부트를 최신 버전으로 변경
    * profile.sh : 앞선 4개 스크립트 파일에서 공용으로 사용할 'profile'과 포트 체크 로직
    * 각 스크립트들 역시 프로젝트의 scripts 디렉토리에 작성하면 된다.
- EC2에 step3 디렉토리 생성
```
mkdir ~/app/step3 && mkdir ~/app/step3/zip
```
- appspec.yml에 step3로 배포되도록 수정하고 스크립트를 사용하도록 설정
```yml

version: 0.0
os: linux
files:
  - source: /
    destination: /home/ec2-user/app/step3/zip/
    overwrite: yes

permissions:
  - object: /
    pattern: "**"
    owner: ec2-user
    group: ec2-user

hooks:
  AfterInstall:
    - location: stop.sh # Nginx와 연결되어 있지 않은 스프링 부트를 종료한다.
      timeout: 60
      runas: ec2-user
  ApplicationStart:
    - location: start.sh # Nginx와 연결되어 있지 않은 Port로 새 버전의 스프링 부트를 시작한다.
      timeout: 60
      runas: ec2-user
  ValidateService:
    - location: health.sh # 새 스프링 부트가 정상적으로 실행됐는지 확인한다.
      timeout: 60
      runas: ec2-user
```
- profile.sh 스크립트 작성
```sh
#!/usr/bin/env bash

# 쉬고 있는 profile 찾기: real1이 사용중이면 real2가 쉬고 있고, 반대면 real1이 쉬고 있음
function find_idle_profile()
{
    RESPONSE_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost/profile)

    if [ ${RESPONSE_CODE} -ge 400 ] # 400 보다 크면 (즉, 40x/50x 에러 모두 포함)
    then
        CURRENT_PROFILE=real2
    else
        CURRENT_PROFILE=$(curl -s http://localhost/profile)
    fi

    if [ ${CURRENT_PROFILE} == real1 ]
    then
      IDLE_PROFILE=real2
    else
      IDLE_PROFILE=real1
    fi

    echo "${IDLE_PROFILE}"
}

# 쉬고 있는 profile의 port 찾기
function find_idle_port()
{
    IDLE_PROFILE=$(find_idle_profile)

    if [ ${IDLE_PROFILE} == real1 ]
    then
      echo "8081"
    else
      echo "8082"
    fi
} 
```
- stop.sh 스크립트 작성
```sh
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh

IDLE_PORT=$(find_idle_port)

echo "> $IDLE_PORT 에서 구동중인 애플리케이션 pid 확인"
IDLE_PID=$(lsof -ti tcp:${IDLE_PORT})

if [ -z ${IDLE_PID} ]
then
  echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다."
else
  echo "> kill -15 $IDLE_PID"
  kill -15 ${IDLE_PID}
  sleep 5
fi 
```
- start.sh 스크립트 작성
```sh
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh

REPOSITORY=/home/ec2-user/app/step3
PROJECT_NAME=aws-springboot2-webservice

echo "> Build 파일 복사"
echo "> cp $REPOSITORY/zip/*.jar $REPOSITORY/"

cp $REPOSITORY/zip/*.jar $REPOSITORY/

echo "> 새 어플리케이션 배포"
JAR_NAME=$(ls -tr $REPOSITORY/*.jar | tail -n 1)

echo "> JAR Name: $JAR_NAME"

echo "> $JAR_NAME 에 실행권한 추가"

chmod +x $JAR_NAME

echo "> $JAR_NAME 실행"

IDLE_PROFILE=$(find_idle_profile)

echo "> $JAR_NAME 를 profile=$IDLE_PROFILE 로 실행합니다."
nohup java -jar \
    -Dspring.config.location=classpath:/application.properties,classpath:/application-$IDLE_PROFILE.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
    -Dspring.profiles.active=$IDLE_PROFILE \
    $JAR_NAME > $REPOSITORY/nohup.out 2>&1 & 
```
- health.sh 스크립트 작성
```sh
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh
source ${ABSDIR}/switch.sh

IDLE_PORT=$(find_idle_port)

echo "> Health Check Start!"
echo "> IDLE_PORT: $IDLE_PORT"
echo "> curl -s http://localhost:$IDLE_PORT/profile "
sleep 10

for RETRY_COUNT in {1..10}
do
  RESPONSE=$(curl -s http://localhost:${IDLE_PORT}/profile)
  UP_COUNT=$(echo ${RESPONSE} | grep 'real' | wc -l)

  if [ ${UP_COUNT} -ge 1 ]
  then # $up_count >= 1 ("real" 문자열이 있는지 검증)
      echo "> Health check 성공"
      switch_proxy
      break
  else
      echo "> Health check의 응답을 알 수 없거나 혹은 실행 상태가 아닙니다."
      echo "> Health check: ${RESPONSE}"
  fi

  if [ ${RETRY_COUNT} -eq 10 ]
  then
    echo "> Health check 실패. "
    echo "> Nginx에 연결하지 않고 배포를 종료합니다."
    exit 1
  fi

  echo "> Health check 연결 실패. 재시도..."
  sleep 10
done 
```
- switch.sh 스크립트 작성
```sh
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh

function switch_proxy() {
    IDLE_PORT=$(find_idle_port)

    echo "> 전환할 Port: $IDLE_PORT"
    echo "> Port 전환"
    echo "set \$service_url http://127.0.0.1:${IDLE_PORT};" | sudo tee /etc/nginx/conf.d/service-url.inc

    echo "> Nginx Reload"
    sudo service nginx reload
}
```
<br>

### 10-3 무중단 배포 테스트
- 무중단 배포 테스트를 하기 전 잦은 배포로 Jar파일명이 겹칠 수 있고 매번 버전을 올리는 것이 번거로우므로 자동으로 버전값이 변경될 수 있도록 한다.
    * build.gradle 수정
```gradle
version '1.0.1-SNAPSHOT-'+new Date().format("yyyyMMddHHmmss")
```
- 최종 코드를 깃허브로 푸시한다. 
    * CodeDeploy 로그 확인
        - `tail -f /opt/codedeploy-agent/deployment-root/deployment-logs/codedeploy-agent-deployments.log`
    * 스프링 부트 로그 확인
        - `vim ~/app/step3/nohup.out`
    * 자바 애플리케이션 실행 여부
        - `ps -ef | grep java`
- 한번 더 배포하면 real2로 배포되고 중단 없이 배포되는 것을 확인할 수 있다.
- 이제 이 시스템은 마스터 브랜치에 푸시가 발생하면 자동으로 서버 배포가 진행되고, 서버 중단 역시 전혀 없는 시스템이 되었다!