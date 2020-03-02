> [신입 프로그래머를 위한 실전 JSP 강좌 강의](https://www.inflearn.com/course/%EC%8B%A4%EC%A0%84-jsp-%EA%B0%95%EC%A2%8C/dashboard)를 듣고 공부한 내용을 정리하여 기록

<br>

### 액션 태그
- JSP 페이지 내에서 어떤 동작을 하도록 지시하는 태그
- 페이지 이동, 페이지 include 등이 있다.
<br>

### forward
- 현재 페이지에서 다른 특정 페이지로 전환할 때 사용
```html
<%@ page language="java" contentType="text/html; charset=EUC-KR"
    pageEncoding="EUC-KR"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=EUC-KR">
<title>Insert title here</title>
</head>
<body>

	<h1>main.jsp 페이지 입니다.</h1>

	<jsp:forward page="sub.jsp" />

<!--
실행하면 바로 sub.jsp로 페이지를 forwarding을 한다.
URL은 main.jsp로 뜨지만 페이지는 sub.jsp이고 코드도 sub.jsp로 뜬다.
-->
</body>
</html>
```
<br>

### include
- 현재 페이지에 다른 페이지를 삽입할 때 사용
```html
<%@ page language="java" contentType="text/html; charset=EUC-KR"
    pageEncoding="EUC-KR"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=EUC-KR">
<title>Insert title here</title>
</head>
<body>

	<h1> include01.jsp 페이지 입니다. </h1>
	<jsp:include page="include02.jsp" flush="true" />
	<h1> 다시 include01.jsp 페이지 입니다. </h1>

<!--
include01.jsp가 실행되다가 include02.jsp가 실행이 다 끝난 후 다시 include01.jsp 실행된다.
-->
</body>
</html>
```
<br>

### param
- forward 및 include 태그에 데이터 전달을 목적으로 사용된다.
- 이름과 값으로 이루어져 있다.
```html
<%@ page language="java" contentType="text/html; charset=EUC-KR"
    pageEncoding="EUC-KR"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=EUC-KR">
<title>Insert title here</title>
</head>
<body>

	<jsp:forward page="forward_param.jsp">
		<jsp:param name="id" value="abcdef" />
		<jsp:param name="pw" value="1234" />
	</jsp:forward>

</body>
</html>
```
<br>

- 이름과 값을 넘겨받은 page가 파라미터를 사용할 수 있다.
```html
<%@ page language="java" contentType="text/html; charset=EUC-KR"
    pageEncoding="EUC-KR"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=EUC-KR">
<title>Insert title here</title>
</head>
<body>

	<%!
		String id, pw;
	%>

	<%
		id = request.getParameter("id");
		pw = request.getParameter("pw");
	%>

	<h1>forward_param.jsp 입니다.</h1>
	아이디 : <%= id %><br />
	비밀번호 : <%= pw %>

</body>
</html>
```
