# 에러 처리 및 배포 준비

## 에러 핸들러 및 뷰 추가
- 지금 애플리케이션은 잘못된 요청을 보냈을 때 스프링 부트가 제공하는 Whitelabel Error Page가 나온다.
- 직접 원하는 에러 페이지를 보여주고 추가적인 에러 처리를 한다.
- 잘못된 요청의 예
    * 없는 스터디 페이지 조회 시도
    * 없는 프로필 페이지 조회 시도
    * 무작위 이벤트 조회 시도
    * 허용하지 않는 요청 시도
    * 이미 종료된 스터디의 모임 생성 시도
    * 이미 종료된 모임에 참가 신청 시도
    * 관리자 권한이 없는 스터디 수정 시도
    * ...
<br>

### 구현
- 에러 페이지 추가
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments.html :: head"></head>
<body>
<section class="jumbotron text-center">
    <div class="container">
        <h1>스터디올래</h1>
        <p class="lead text-muted">
            잘못된 요청입니다.<br/>
        </p>
        <p>
            <a th:href="@{/}" class="btn btn-primary my-2">첫 페이지로 이동</a>
        </p>
    </div>
</section>
</body>
</html>
```
- 이렇게 하면 완성이지만 부가적인 처리를 하고 싶은 경우 에러 핸들러를 따로 만들면 된다.
    * 로그를 남겨놓거나
    * 악의적인 잘못된 요청을 보내거나
    * 어떤 요청들이 자주 잘못된 요청이 오는가를 파악
```java
@Slf4j
@ControllerAdvice
public class ExceptionAdvice {

    @ExceptionHandler
    public String handleRuntimeException(@CurrentAccount Account account, HttpServletRequest req, RuntimeException e) {
        if (account != null) {
            log.info("'{}' requested '{}'", account.getNickname(), req.getRequestURI());
        } else {
            log.info("requested '{}'", req.getRequestURI());
        }
        log.error("bad request", e);
        return "error";
    }
}
```
- 실행해서 잘못된 요청을 보내면 다음과 같이 로깅된 것을 확인할 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/errorHandling_distribution_1.jpg"></p>

<br>

## 배포 시 고려할 것
- 고려해야 할 것
    * 환경(프로필)에 따라 각기 다른 설정 파일 제공하는 방법
    * 로깅
    * 패키징
    * 배포 방법

- 프로필별 설정 파일
    * application-{profile}.properties
    * 설정에 중복되는 값이 있으면 특정 profile에 있는 값이 기본 값을 오버라이딩한다.
    * 위치에 따른 우선 순위
        - 파일 시스템 “현재 디렉토리/config”에 있는 application-{profile}.properties
        - 파일 시스템 “현재 디렉토리”에 있는 application-{profile}.properties
        - 클래스패스의 “.config”에 들어있는 application-{profile}.properties
        - 클래스패스 루트에 있는 application-{profile}.properties
- 로깅
    * 모니터링 시스템과 연동 필요
    * 민감한 데이터를 로깅하지 않도록 설정
    * 각 배포 환경에 알맞은 로길 설정 필요
    * local이 아니면 설정하지 않는 것이 좋다.
- 패키징
    * 외부 톰캣 인스턴스에 WAR로 배포할 것인가
        - 이미 다른 서버에서 관리하고 있는 톰캣 인스턴스가 있고 그 곳에 배포하는 형식
    * 톰캣을 내장한 형태의 JAR로 배포할 것인가
        - packaging을 하면 jar파일 하나가 만들어지는데 
- 배포방법
    * CI / CD 구축 필요
        - CI(Continuous Integration) : 지속적인 통합
        - CD(Continuous Deployment) : 지속적인 배포
    * 코드를 변경했을 때마다 CI 서버가 모든 테스트를 실행하고 packaging한다.
    * CD는 코드를 배포할 준비가 됐거나, 주기적으로 특정 환경에 패키징한 애플리케이션을 배포.