> [신입 프로그래머를 위한 실전 JSP 강좌 강의](https://www.inflearn.com/course/%EC%8B%A4%EC%A0%84-jsp-%EA%B0%95%EC%A2%8C/dashboard)를 듣고 공부한 내용을 정리하여 기록

<br> 
### JSP(Java Server Page)
- 동적 웹 애플리케이션 컴포넌트
- 확장자 : .jsp
- 클라이언트의 요청에 동적으로 작동하고, 응답은 html을 이용한다.
- jsp는 서블릿으로 변환되어 실행된다.
- MVC 패턴에서 View로 이용된다.
img < JSP 실행 흐름 >

- xxx.jsp 파일은 실행될 때 xxx_jsp.java로 변환되고, xxx_jsp.class로 컴파일되어 html로 응답한다.
- .java파일과 .class파일은 tomcat설치폴더/work/catalina/localhost/프로젝트명 폴더에 저장된다.

### Servlet(Server Applet)
- 동적 웹 애플리케이션 컴포넌트
- 확장자 : .java
- 클라이언트의 요청에 동적으로 작동하고, 응답은 html을 이용한다.
- java thread를 이용하여 동작한다.
- MVC 패턴에서 Controller로 이용된다.


#### Mapping
- 기존의 url은 길고 보안에 쉽게 노출되어 있다. 이러한 경로를 간단하게 표현하는 것
- ex) http ://localhost:8080/helloworld/servlet/com.javalec.ex.HelloWorld  -> http ://localhost:8080/helloWorld/hw

##### (1) web.xml에 서블릿 맵핑
```html
<servlet>
    <servlet-name>helloworld</servlet-name>
    <servlet-class>com.javalec.ex.HelloWorld</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>helloworld</servlet-name>
    <url-pattern>/hw</url-pattern>
</servlet-mapping>
```
- web.xml파일에 직접 입력하는 방법이다.
- servlet-name은 임의로 지정하지만 servlet-class는 실제 패키지명과 클래스 파일명으로 입력해야 한다.


##### (2) annotation을 이용한 서블릿 맵핑
```java
/**
 * Servlet implementation class HelloWorld
 */
@WebServlet("/hw")
public class HelloWorld extends HttpServlet {
	private static final long serialVersionUID = 1L;
  		  ...
```
- 맵핑명을 실제 .java 소스에 입력하는 방법이다.
