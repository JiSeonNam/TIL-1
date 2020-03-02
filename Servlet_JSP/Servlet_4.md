> [신입 프로그래머를 위한 실전 JSP 강좌 강의](https://www.inflearn.com/course/%EC%8B%A4%EC%A0%84-jsp-%EA%B0%95%EC%A2%8C/dashboard)를 듣고 공부한 내용을 정리하여 기록

<br>
### 서블릿 초기화 파라미터 (ServletConfig)
- 특정 Servlet이 생성될 때 초기에 필요한 데이터들을 초기화 파라미터(initialization Parameter)라고 한다.
- 초기화 파라미터를 web.xml 파일에 설정하고 Servlet 파일에서는 ServletConfig 클래스를 이용해서 접근(사용)한다.
	* Servlet 클래스 생성
	* web.xml 파일에 초기화 파라미터 기술
	* ServletConfig 메서드를 이용해서 데이터 불러오기
```html
<servlet>
<servlet-name>ServletInitParam</servlet-name>
<servlet-class>com.javalec.ex.ServletInitParam</servlet-class>
```
```html
<init-param>
<param-name>id</param-name>
<param-value>abcd</param-value>
</init-param>
<init-param>
<param-name>pw</param-name>
<param-value>1234</param-value>
</init-param>
</servlet>
<servlet-mapping>
<servlet-name>ServletInitParam</servlet-name>
<url-pattern>/initParam</url-pattern>
</servlet-mapping>
```
```java
String id = request.getInitParameter("id");
String pw = request.getInitParameter("pw");
```
<br>

- web.xml에 설정하지 않고 Servlet 파일에 직접 기술할 수도 있다.
	* Servlet 클래스 생성
	* @WebInitParam 에 초기화 파라미터 기술
	* ServletConfig 메서드를 이용해서 데이터 불러오기
<br>
```css
@WebServlet(urlPatterns={"/ServletInitParam"}, initParams={@WebInitParam(name="id", value="abc")
			 , @WebInitParam(....), ...}
```
```java
String id = request.getInitParameter("id");
String pw = request.getInitParameter("pw");
```
<br>

### 데이터 공유 (servlet Context)
- 여러 Servlet에서 특정 데이터를 공유해야 할 경우 context parameter를 이용해서 web.xml에 데이터를 기술하고 Servlet에서 공유하면서 사용할 수 있다.
	* Servlet 클래스 생성
	* web.xml 파일에 context parameter 기술
	* ServletContext 메서드 이용해서 데이터 불러오기
```html
<context-param>
  <param-name>id</param-name>
  <param-value>abcd</param-value>
</context-param>
<context-param>
  <param-name>pw</param-name>
  <param-value>1234</param-value>
</context-param>
```
```java
String id = getServletContext().getInitParameter("id");
String pw = getServletContext().getInitParameter("pw");
```
<br>

### 웹어플리케이션 감시 (servletContextListener)
- 웹어플리케이션 생명주기(LifeCycle)를 감시하는 Listener가 ServletContextListener이다.
- Listener의 contextInitialized( )와 contextDestroyed( )가 웹 어플리케이션의 시작과 종료 시 호출된다.
- Listener 클래스를 새로 생성하고 web.xml 파일에 기술함으로써 만들 수 있다.
```java
@WebListener
public class ContextListenerEx implements ServletContextListener{

	public ContextListenerEx() {
		// TODO Auto-generated constructor stub
	}

	@Override
	public void contextDestroyed(ServletContextEvent arg0) {
		// TODO Auto-generated method stub
		System.out.println("contextDestroyed");
	}

	@Override
	public void contextInitialized(ServletContextEvent arg0) {
		// TODO Auto-generated method stub
		System.out.println("contextInitialized");
	}
}
```
```html
  <listener>
	<listener-class>com.javalec.ex.ContextListenerEx</listener-class>
  </listener>
```
