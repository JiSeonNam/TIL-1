> [신입 프로그래머를 위한 실전 JSP 강좌 강의](https://www.inflearn.com/course/%EC%8B%A4%EC%A0%84-jsp-%EA%B0%95%EC%A2%8C/dashboard)를 듣고 공부한 내용을 정리하여 기록

### 빈
- JAVA의 데이터(속성)와 기능(메서드)으로 이루어진 클래스
- 반복적인 작업을 효율적으로 하기 위해 사용한다.
- jsp 페이지를 만들고 액션태그를 사용하여 빈을 사용한다.
<br>

### 빈 만들기
- 데이터 객체에는 데이터가 있어 그에 해당하는 getter와 setter가 있다.
- 빈을 만든다는 것은 데이터 객체를 만들기 위한 클래스를 만드는 것이다.
- JAVA에서 데이터 객체를 만드는 클래스와 동일하다. jsp에서는 Bean이라고 한다.
<br>

### 빈을 사용하는 방법 : 빈 관련 액션 태그 (useBean, getProperty, setProperty)
- 액션 태그 중에서 Bean 관련 태그가 있다.
- 주로 데이터를 업데이트하고, 얻어오는 역할을 한다.

#### useBean
- 특정 Bean을 사용한다고 명시할 때 사용
```html
<jsp:useBean id="student" class="com.javalec.ex.Student" scope="page"/>
<!--
id - java에서의 참조변수에 해당
class - 클래스 이름
scope - 스코프 범위
-->
```
<br>

**※ Scope**
- page : 생성된 페이지 내에서만 사용 가능
- request : 요청된 페이지 내에서만 사용 가능
- session : 웹브라우저의 생명주기와 동일하게 사용 가능
- application : 웹어플리케이션 내에서 사용 가능
<br>

#### setProperty
- 데이터 값을 설정할 때 사용
```html
<jsp:setProperty name="student" property="name" value="홍길동"/>
<!-- student라는 Bean에 name이라는 속성을 홍길동이라는 값으로 설정 -->
```
<br>

#### getProperty
- 데이터 값을 가져올 때 사용
```html
<jsp:getProperty name="student" property="name"/>
<!-- student라는 Bean의 name이라는 속성의 값을 가져온다 -->
```
<br>

<details>
  <summary>**예제코드 펼치기/접기**</summary>

```html
<%@ page language="java" contentType="text/html; charset=EUC-KR"
    pageEncoding="EUC-KR"%>
<jsp:useBean id="student" class="com.javalec.ex.Student" scope="page" />
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=EUC-KR">
<title>Insert title here</title>
</head>
<body>

	<jsp:setProperty name="student" property="name" value="홍길동"/>
	<jsp:setProperty name="student" property="age" value="13"/>
	<jsp:setProperty name="student" property="grade" value="6"/>
	<jsp:setProperty name="student" property="studentNum" value="7"/>

	이름 : <jsp:getProperty name="student" property="name" /><br />
	나이 : <jsp:getProperty name="student" property="age" /><br />
	학년 : <jsp:getProperty name="student" property="grade" /><br />
	번호 : <jsp:getProperty name="student" property="studentNum" /><br />

</body>
</html>
```
</details>
