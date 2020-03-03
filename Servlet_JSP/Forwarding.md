> [신입 프로그래머를 위한 실전 JSP 강좌 강의](https://www.inflearn.com/course/%EC%8B%A4%EC%A0%84-jsp-%EA%B0%95%EC%A2%8C/dashboard)를 듣고 공부한 내용을 정리하여 기록

<br>

### Forwrading
- 서블릿 또는 JSP에서 요청을 받은 후 다른 컴포넌트로 요청을 위임할 수 있다.
- 위임을 하는 2가지 방법에는 RequestDispatcher 클래스, HttpServletRespnse 클래스가 있다.
<br>

### RequestDispatcher 클래스
![](https://github.com/qlalzl9/TIL/blob/master/Servlet_JSP/img/Forwarding_1.png)
- 요청받은 요청 객체(request)를 위임하는 컴포넌트에 동일하게 전달할 수 있다.
<br>

### HttpServletResponse 클래스
![](https://github.com/qlalzl9/TIL/blob/master/Servlet_JSP/img/Forwarding_2.png)
- RequestDispatcher 클래스와는 다르게 요청받은 요청 객체를 위임받은 컴포넌트에 전달하는 것이 아닌, 새로운 요청 객체를 생성한다.
