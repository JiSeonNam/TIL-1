# 스프링 부트와 AWS로 혼자 구현하는 웹 서비스

## 인텔리제이로 스프링 부트 시작하기

### 그레이들 프로젝트 생성 후 스프링 부트 프로젝트로 변경하기
- build.gradle 파일에 다음 코드와 같이 작성한다.
    * ext 키워드는 build.gradle에서 사용하는 전역변수를 설정하겠다는 의미
    * repositoris는 각종 의존성들을 어떠한 원격 저장소에서 받을 지를 지정하는 것
        - 버젼을 명시하지 않아야 `dependencies { classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")`에서 작성한 대로 버젼을 따라간다.
    }
```java
// 플러그인 의존성 관리 설정
buildscript {
    ext {
        springBootVersion = '2.1.7.RELEASE'
    }
    repositories {
        mavenCentral()
        jcenter()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}

// 앞서 선언한 플러그인 의존성들을 적용할 것인지를 결정하는 코드
// 자바와 스프링 부트를 사용하기 위한 필수 플러그인
apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management' // 스프링 부트의 의존성을 관리해 주는 플러그인으로 반드시 추가해야 한다.

group 'org.example'
version '1.0-SNAPSHOT'

repositories {
    mavenCentral()
}

dependencies {
    compile('org.springframework.boot:spring-boot-starter-web')
    testCompile ('org.springframework.boot:spring-boot-starter-test')
}
```
<br>
