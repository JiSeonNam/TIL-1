# EL(Expression Laguage)
- 표현 언어(Expression Language)는 값을 표현하는 데 사용되는 스크립트 언어로서 JSP의 기본 문법을 보완하는 역할을 한다.
- 표현식 또는 액션 태그를 대신해서 값을 표현하는 언어이다.
- 표현식 <%= value %> → EL ${ value }
<br>

## EL의 표현방법과 기본 객체
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/EL_1.jpg"></p>
<br>

## 표현 언어의 기본 객체 사용 예
```html
<%@ page contentType = "text/html; charset=utf-8" %>
<%
    request.setAttribute("name", "홍길동");
%>
<html>
<head><title>EL Object</title></head>
<body>

요청 URI:${pageContext.request.requestURI} <br>
request의 name 속성 : ${requestScope.name} <br>
code 파라미터 : ${param.code}

</body>
</html>
```