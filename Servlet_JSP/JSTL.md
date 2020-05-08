# JSTL (JSP standard Tag Library)
- JSP의 경우 HTML 태그와 같이 사용되어 전체적인 코드의 가독성이 떨어진다.
- 이러한 단점을 보완하고자 만들어진 태그 라이브러리가 JSTL이다
- JSP 페이지에서 조건문 처리, 반복문 처리 등을 html tag형태로 작성할 수 있게 도와준다.
<br>
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Servlet_JSP/img/JSTL_2.jpg"></p>
<br>

### JSTL 라이브러리
- JSTL은 5가지 라이브러리를 제공한다.
![](https://github.com/qlalzl9/TIL/blob/master/Servlet_JSP/img/JSTL_1.png)

#### Core
- Core 라이브러리는 기본적인 라이브러리로 출력, 제어문, 반복문 같은 기능이 포함되어 있다.
- <%@ taglib uri=http//java.sun.com/jsp/jstl/core/ prefix="c" %>
- 출력 태그 : <c:out>
```java
<c:out value="출력값" default="기본값" escapeXml="true or false">
```
- 변수 설정 태그 : <c:set>
```java
<c:out value="출력값" default="기본값" escapeXml="true or false">
```

- 변수를 제거하는 태그 : <c:remove>
```java
<c:remove var="변수명" scope="범위">
```

- 예외 처리 태그 : <c:catch>
```java
<c:catch var="변수명">
```

- 제어문(if) 태그 : <c:if>
```java
<c:if test="조건" var="조건 처리 변수명" scope="범위">
```

- 제어문(swich) 태그 : <c:choose>
```java
<c:choose>
<c:when test="조건"> 처리 내용 </c:when>
<c:otherwise> 처리 내용 </c:otherwise>
</c:choose>
```

- 반복문(for) 태그 : <c:forEach>
```java
<c:forEach items="객체명" begin="시작 인덱스" end="끝 인덱스" step="증감식" var="변수명" varStatus="상태변수">
```
- 페이지 이동 태그 : <c:redirect>
```java
<c:redirect url="url">
```

- 파라미터 전달 태그 : <c:param>
```java
<c:param name="파라미터명" value="값">
```
