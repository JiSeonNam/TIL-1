> [신입 프로그래머를 위한 실전 JSP 강좌 강의](https://www.inflearn.com/course/%EC%8B%A4%EC%A0%84-jsp-%EA%B0%95%EC%A2%8C/dashboard)를 듣고 공부한 내용을 정리하여 기록

<br>

### EL(Expression Laguage)
- 표현식 또는 액션 태그를 대신해서 값을 표현하는 언어이다.
- 표현식 <%= value %> → EL ${ value }
<br>

### EL 내장 객체
- pageScopte : page 객체를 참조하는 객체
- requestScope : request 객체를 참조하는 객체
- sessionScope : session 객체를 참조하는 객체
- applicationScope : application 객체를 참조하는 객체
- param : 요청 파라미터를 참조하는 객체
- paramValues : 요청 파라미터(배열)를 참조하는 객체
- initParam : 초기화 파라미터를 참조하는 객체
- cookie : cookie 객체를 참조하는 객체
