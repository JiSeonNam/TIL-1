> [신입 프로그래머를 위한 실전 JSP 강좌 강의](https://www.inflearn.com/course/%EC%8B%A4%EC%A0%84-jsp-%EA%B0%95%EC%A2%8C/dashboard)를 듣고 공부한 내용을 정리하여 기록

<br>

- Servlet은 JAVA 언어를 이용하여 문서를 작성하고, 출력 객체(PrintWriter)를 이용하여 HTML 코드를 삽입한다.
- JSP는 Servlet과 반대로 HTML 코드에 JAVA언어를 삽입하여 동적 문서를 만들 수 있다.
<br>

### JSP 태그 종류
|태그|문법|설명|
|:-:|:-:|:-:|
|지시자|<%@    %>|페이지의 속성을 나타낸다.|
|주석|<%--    --%>|변수, 메서드 선언|
|선언|<%!    %>|	 
|표현식|<%=    %>|결과값 출력|
|스크립트릿|<%    %>|JAVA 코드를 넣을 때 사용하며 가장 많이 쓴다.|
|액션 태그|< jsp:action ></ jsp:acton >|자바빈 연결|
<br>

### JSP 동작 원리
![](https://github.com/qlalzl9/TIL/blob/master/Servlet_JSP/img/JSP_1_1.png)
- 클라이언트가 웹브라우저로 helloWorld.jsp를 요청하게 되면 JSP컨테이너가 JSP파일을 Servlet파일(.java)로 변환한다.
- Servlet파일은 컴파일 된후 클래스 파일(.class)로 변환되고, 요청한 클라이언트에게 html 파일 형태로 응답된다.
<br>

### JSP 내부 객체
- 내부 객체는 개발자가 객체를 생성하지 않아도 바로 사용할 수 있는 객체이다.
- JSP 컨테이너에 의해 Servlet으로 변환될 때 자동으로 객체가 생성된다.
- 내부 객체의 종류
	* 입출력 객체 : request, response
	* 서블릿 객체 : page, config
	* 세션 객체 : session
	* 예외 객체 : exception
