> [신입 프로그래머를 위한 실전 JSP 강좌 강의](https://www.inflearn.com/course/%EC%8B%A4%EC%A0%84-jsp-%EA%B0%95%EC%A2%8C/dashboard)를 듣고 공부한 내용을 정리하여 기록

### Servlet Parameter
img
- Form태그의 submit 버튼을 클릭하여 데이터를 서버로 전송하면, 해당 파일(Servlet)에서는 HttpServletRequest 객체를 이용하여 Parameter값을 얻을 수 있다.
- getParameter(name) : Form태그에 있는 value 값을 반환
- getParameterValues(name) : 2개 이상의 value 값을 반환
- getParameterNames( ) : 넘어온 데이터들의 이름들을 반환
<br>

### 한글처리
- 기본 문자 처리 방식의 한글이 깨져 보일 경우 Get방식과 Post방식 2가지 방법으로 한글 처리가 가능하다.
- Get방식은 server.xml 파일의 URIEncoding을 "EUC-KR" 또는 "UTF-8"로 설정한다.
```java
<Connector URIEncoding="EUC-KR" port="8090" protocol="HTTP/1.1"...중략...>
```
- Post방식은 setCharacterEncoding( )을 사용하여 "EUC-KR" 또는 "UTF-8"로 설정한다.
```java
protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		// TODO Auto-generated method stub
		request.setCharacterEncoding("EUC-KR");
        ...
}
```
