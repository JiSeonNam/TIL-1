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
