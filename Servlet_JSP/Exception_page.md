> [신입 프로그래머를 위한 실전 JSP 강좌 강의](https://www.inflearn.com/course/%EC%8B%A4%EC%A0%84-jsp-%EA%B0%95%EC%A2%8C/dashboard)를 듣고 공부한 내용을 정리하여 기록

<br>

### page 지시자를 이용한 예외 처리
- 에러가 발생할 수 있는 페이지에 page 지시자의 errorPage 속성을 사용하여 페이지 이동 명시
```java
<%@ page errorPage= "errorPage.jsp"%>
```
- 에러 페이지로 지정된 페이지에서는 page 지시자의 isErrorPage 속성을 사용하여 에러 페이지임을 명시
명시를 안해주면 기본값인 false이며 isErrorPage가 false이면 exception이라는 객체를 사용할 수 없다.
- 웹페이지에서 에러페이지인 500으로 처리하는 경우가 있어서 이것을 방지하기 위해 정상 페이지라는 의미의 response.setStatus(200); 을 해줘야 한다.
```java
<%@ page isErrorPage= "true"%>
<% reponse.setStatus(200); %>
	...
<body>
<%= exception.getMessage() %>
</body>
```
<br>

### 응답 상태 코드별 예외 처리
```html
<errorPage>
  <errorCode> 404 </errorCode>
  <location> /error404.jsp </location>		<!-- 404에러 발생 시 error404.jsp 페이지로 이동-->
</errorPage>
<errorPage>
  <errorCode> 500 </errorCode>
  <location> /error500.jsp </location>		<!-- 505에러 발생 시 error505.jsp 페이지로 이동-->
</errorPage>
```
<br>

### exception 타입별 예외 처리
```html
<errorPage>
  <exception-type> 404 </exception-type>
  <location> /error/errorNullPointer.jsp </location>	<!-- NullPointer 에러 발생 시시 errorNullPointer.jsp 페이지로 이동-->
</errorPage>
```
