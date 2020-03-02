> [신입 프로그래머를 위한 실전 JSP 강좌 강의](https://www.inflearn.com/course/%EC%8B%A4%EC%A0%84-jsp-%EA%B0%95%EC%A2%8C/dashboard)를 듣고 공부한 내용을 정리하여 기록

<br>

### 세션
- 서버와의 관계를 유지하기 위한 수단이다.
- 쿠키와 달리 클라이언트 특정 위치에 저장되는 것이 아니라, 서버 상에 객체로 존재한다.
- 세션은 서버에서만 접근 가능하여, 보안이 좋고, 저장할 수 있는 데이터에 한계가 없다.
- 서버 상에 객체로 존재하며 브라우저 당 하나의 세션 객체가 생성되고 각각 고유의 유니크한 id값이 있다.
<br>

### 세션 문법
- 세션은 클라이언트의 요청이 발생하면 서버 컨테이너에서 자동생성된다.
- session이라는 내부 객체를 지원하여 메서드를 통해 세션의 속성을 설정할 수 있다.
- session.setAttribute("mySessionName", "mySessionValue")로 세션에 데이터를 저장한다.
- session.getAttribute("mySssionName")으로 데이터를 가져오며 항상 Object 타입으로 받아 오기 때문에 사용할 때 원하는 타입으로 형 변환을 해서 사용해야 한다.
	* Object obj1 = session.getAttribute("mySessionName");
	* String mySessionName = (String)obj1;
- sesion.getAttributeNames( ) : 세션에 저장되어 있는 모든 Name을 다 얻어 온다.
- getId( ) : 자동 생성된 세션의 유니크한 아이디를 얻는다.
- isNew( ) : 세션이 최조 생성되었는지, 이전에 생성된 세션인지를 구분한다.
- getMaxInactiveInterval( ) : 세션의 유효시간을 얻는다. 가장 최근 요청시점을 기준으로 카운트된다.
	* 톰캣 설치 폴더의 conf/web.xml에 기본으로 30분으로 설정되어 있다. 수정하면 변경 가능하다.
	* 세션 유효시간을 설정하는 2가지 방법
		- getMaxInactiveInrerval( ) 메서드를 사용하여 변경
		- web.xml 파일의 <session-config> 태그를 사용하여 유효시간 변경
- removeAttribute( ) : 세션에서 특정 데이터를 제거한다.
- invalidate( ) : 세션의 모든 데이터를 삭제한다.
