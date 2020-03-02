> [신입 프로그래머를 위한 실전 JSP 강좌 강의](https://www.inflearn.com/course/%EC%8B%A4%EC%A0%84-jsp-%EA%B0%95%EC%A2%8C/dashboard)를 듣고 공부한 내용을 정리하여 기록

<br>

### request 객체
- 웹브라우저를 통해 서버에 정보를 요청하는 것을 request라고 한다. 이러한 요청 정보는 request 객체가 관리한다.
- Request 객체 관련 메서드
	* getContextPath( ) : 웹어플리케이션의 Context path를 얻는다.
	* getServerName( ) : 서버 이름을 얻는다.
	* getServerPort( ) : 포트 번호를 얻는다.
	* getMethod( ) : get방식과 post방식을 구분한다.
	* getSession( ) : 세션 객체를 얻는다.
	* getProtocol( ) : 해당 프로토콜을 얻는다.
	* getRequestURL( ) : 요청 URL를 얻는다.
	* getRequestURI( ) : 요청 URI를 얻는다.
	* getQueryString( ) : 쿼리 문자열을 얻는다.
```java
<%
		out.println("서버 : " + request.getServerName() + "<br/>");
		out.println("포트 번호 : " + request.getServerPort() + "<br/>");
		out.println("요청 방식 : " + request.getMethod() + "<br/>");
		out.println("프로토콜 : " + request.getProtocol() + "<br/>");
		out.println("URL : " + request.getRequestURL() + "<br/>");
		out.println("URI : " + request.getRequestURI() + "<br/>");
%>
```

- Parameter 메서드
	* 요청 관련메서드 보다 실제로 더 많이 쓰인다.
	* JSP 페이지를 제작하는 목적이 데이터 값을 전송하기 위해서이므로, parameter 메서드가 더 중요하다.
	* 종류
		- getparameter(String name) : name에 해당하는 parameter 값을 얻는다.
		- getParameterNames ( ) : 모든 parameter 이름을 얻는다.
		- getParamerterValues(String name) : name에 해당하는 2개 이상의 parameter 값들을 배열로 얻는다.
```java
<body>
<%!
  String name, id, pw;
  String[] hobbys;
%>
<%
  request.setCharacterEncoding("UTF-8");

  name = request.getParameter("name");
  id = request.getParameter("id");
  pw = request.getParameter("pw");

  hobbys= request.getParameterValues("hobby");
%>

이름 : <%= name %> <br/>
아이디 : <%= id %> <br/>
비밀번호 : <%= pw %> <br/>
취미 : <%= Arrays.toString(hobbys) %> <br/>
</body>
```
<br>

### response 객체
- 웹브라우저의 요청에 응답하는 것을 response라고 하며, 이러한 response 정보를 가지고 있는 객체를 response 객체라고 한다.
- Response 객체 관련 메서드
	* getCharacterEncoding( ) : 응답할 때의 문자 인코딩 형태를 얻는다.
	* addCookie(Cookie) : 쿠키를 지정한다.
	* sendRedirect(URL) : 지정한 URL로 이동한다.
```java
<body>
<%!
  int age;
%>
<%
  String str = request.getParameter("age");
  age = Integer.parseInt(str);

  if(age >= 20)
    response.sendRedirect("pass.jsp?age=" +age);	// age >= 20이면 pass.jsp로 이동
  else
    response.sendRedirect("ng.jsp?age=" +age);		// age < 20이면 ng.jsp로 이동
%>
<%= age %>
</body>
```
