> [신입 프로그래머를 위한 실전 JSP 강좌 강의](https://www.inflearn.com/course/%EC%8B%A4%EC%A0%84-jsp-%EA%B0%95%EC%A2%8C/dashboard)를 듣고 공부한 내용을 정리하여 기록

<br>

### JSP 스크립(스크립트릿, 선언, 표현식)
- 스크립트릿(Scriptlet) : <%  java 코드  %>
	* JSP 페이지에서 JAVA언어를 사용하기 위한 것으로 가장 많이 사용된다.
- 선언(Declaration) : <%!  java 코드  %>
	* JSP 페이지에서 사용되는 변수 또는 메서드를 선언할 때 사용된다.
	* 선언된 변수 및 메서드는 전역의 의미로 사용된다.
- 표현식(Expression) : <%=  java 코드  %>
	* JSP 페이지에서 사용되는 변수의 값 또는 메서드 호출 결과값을 출력하기 위해 사용된다.
	* 결과값은 String이며, ';'을 사용할 수 없다.

**※ JSP는 서버단에서 처리되기 때문에 html문서(소스보기)에서는 html만 나오고 java코드는 보이지 않는다.**
<br>

### 지시자
- JSP 페이지에서 전체적인 속성을 지정할 때 사용하며 <%@  속성  %> 형태로 사용된다.
- 종류
	* page : 해당 페이지의 전체적인 속성 지정
	* inlcude : 별도의 페이지를 현재 페이지에 삽입
	* taglib : 태그 라이브러리의 태그 사용
<br>

##### page 지시자 : <@ page %>
- 주로 사용되는 언어 지정 및 import문에 사용한다.
```java
<%@ page import="java.util.Arrays"%>
<%@ page language="java" contentType="text/html; charset=EUC-KR" pageEncoding="EUC-KR"%>
```

##### include 지시자 : <%@ include %>
- file 속성을 이용한다.
```java
<h1> include.jsp 페이지 입니다. </h><br>
<%@ include file="include01.jsp"%>
<h1> 다시 include.jsp 페이지 입니다. </h><br>
```

##### taglib 지시자
- 사용자가 만든 tag들을 태그라이브러리라고 한다. 이때 이 태그 라이브러리를 사용하기 위해 taglib 지시자를 사용한다.
- uri 및 prefix 속성이 있으며, uri는 태그라이브러리의 위치 값을 가지고, prefix는 태그를 가리키는 이름값을 가진다.
- 자세한 내용은 JSTL 때 다시 배울 것이다.
<br>

### 주석
- 실제 프로그램에는 영향이 없고, 프로그램 설명의 목적으로 사용되는 태그이다.
- HTML 과 JSP 주석이 별도로 존재한다.
	* HTML : <!- comments -->**
	* JSP : <@-- comments -->**

**※ Java의 주석도 그대로 사용된다. (//, /* */)**
