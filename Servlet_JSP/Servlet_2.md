> [신입 프로그래머를 위한 실전 JSP 강좌 강의](https://www.inflearn.com/course/%EC%8B%A4%EC%A0%84-jsp-%EA%B0%95%EC%A2%8C/dashboard)를 듣고 공부한 내용을 정리하여 기록


### Servlet 작동 순서
![](https://github.com/qlalzl9/TIL/blob/master/Servlet_JSP/img/Servlet_2_1.png)
< Servlet 작동 순서 >
- 클라이언트에서 servlet요청이 들어오면 서버에서는 Servlet Container를 만들고 요청이 있을 때마다 쓰레드가 생성된다.
- 클라이언트 요청 -> 웹서버 -> WAS -> Servlet Container(쓰레드 생성, 서블릿객체 생성)
- Servlet은 요청이 들어오면 JVM 안에서 쓰레드를 생성하기 때문에 다른 CGI 언어들에 비해 서버 부하가 적게 발생한다. (다른 CGI는 요청 있을 때마다 객체 생성한다.)
<br>

### Servlet 라이프 사이클(생명 주기)
![](https://github.com/qlalzl9/TIL/blob/master/Servlet_JSP/img/Servlet_2_2.png)
< Servlet LifCycle >
- Servlet의 사용도가 높은 이유는 빠른 응답 속도때문이다.
- Servlet은 최초 요청 시 객체가 만들어져, 메모리에 로딩되고, 이후 요청 시에는 기존의 객체를 재활용하게 된다.
(따라서 속도가 빠르다.)
