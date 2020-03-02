> [신입 프로그래머를 위한 실전 JSP 강좌 강의](https://www.inflearn.com/course/%EC%8B%A4%EC%A0%84-jsp-%EA%B0%95%EC%A2%8C/dashboard)를 듣고 공부한 내용을 정리하여 기록


### Servlet 클래스
img
- Servlet클래스는 HttpServlet클래스를 상속받는다.
- 따라서 인터페이스 Servlet과 추상클래스 GenericServlet의 멤버들을 모두 사용 가능하다.
<br>
### doGet 방식과 doPost 방식
- HTML 코드에서 WAS(Web Application Server)로 데이터를 보낼 때(request) 사용하는 방식이다.
- 웹어플리케이션에 있는 Servlet이 정보를 받을 때 doGet( ) 또는 doPost( ) 메서드가 호출되면서 메서드가 수행된다.
- doGet( )과 doPost( )는 매개변수로 HttpServletRequest와 HttpServletResponse를 받는다.
- setContentType( )을 호출하여 응답방식을 결정한다.
- getWriter( )를 이용하여 출력 스트림을 얻는다.
<br>
##### doGet 방식
- HTML 내 form태그의 method속성이 get일 경우 호출된다.
- 웹브라우저의 주소창을 이용하여 servlet을 요청한 경우에도 호출된다.
- URL값으로 정보를 포함해서 전송되어 보안에 약하다.
<br>
##### doPost 방식
- HTML 내 form태그의 method속성이 post일 경우 호출된다.
- header를 이용해 정보가 전송되어 보안에 강하다.
<br>
### Context Path
- WAS에서 웹어플리케이션을 구분하기 위한 path이다.
- 이클립스 프로젝트를 생성하면, 자동으로 server.xml에 추가된다.
