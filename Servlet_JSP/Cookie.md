> [신입 프로그래머를 위한 실전 JSP 강좌 강의](https://www.inflearn.com/course/%EC%8B%A4%EC%A0%84-jsp-%EA%B0%95%EC%A2%8C/dashboard)를 듣고 공부한 내용을 정리하여 기록

<br>

### 쿠키
- 웹브라우저에서 서버로 어떤 데이터를 요청하면, 서버 측에서는 알맞은 로직을 수행한 후 데이터를 웹브라우저에 응답한다.
- 서버는 웹브라우저와의 관계를 종료한다. 이렇게 웹브라우저에 응답 후 관계를 끊는 것은 http 프로토콜의 특징이다.
- 연결이 끊겼을 때 어떤 정보를 지속적으로 유지하기 위한 수단으로 쿠키라는 방식을 사용한다.
- 쿠키는 서버에서 생성하여, 서버가 아닌 클라이언트 측에 특정 정보를 저장한다.
- 서버에 요청할 때마다 쿠키의 속성값을 참조 또는 변경할 수 있다.
- 쿠키는 4KB의 용량을 가지며, 300개까지 데이터 정보를 가질 수 있다. (총 1.2MB)
<br>

### 쿠키 문법
1. 쿠키 클래스를 이용하여 쿠키 생성
2. setter를 이용하여 속성 설정
3. response.addCookie( )를 이용하여 response 객체에 쿠키 탑재

- 쿠키 생성 : Cookie cookie = new Cookie("cookieName", "cookieValue")
- cookie.setMaxAge(60) : 쿠키의 유효기간을 1분으로 설정
- 삭제할 때는 cookie.setMaxAge(0)으로 하고 반드시 response.addCookie( )해서 전송해줘야 한다.
- Cookie[ ] cookies = request.getCookies( ) : 쿠키를 받을 때는 request로 받아야 하고 배열로 받는다.
