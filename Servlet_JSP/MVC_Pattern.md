> [신입 프로그래머를 위한 실전 JSP 강좌 강의](https://www.inflearn.com/course/%EC%8B%A4%EC%A0%84-jsp-%EA%B0%95%EC%A2%8C/dashboard)를 듣고 공부한 내용을 정리하여 기록

<br>

### MVC 패턴
- MVC란 Model, View, Controller를 뜻하는 용어로 개발 형태의 일종으로 유지보수에 유리하다.
<br>
- Model은 데이터베이스와의 관계를 담당한다. 클라이언트의 요청에서 필요한 자료를 데이터베이스로부터 추출하거나, 수정하여 Controller로 전달한다.
<br>
- View는 사용자한테 보여지는 UI화면이다. 주로 .jsp파일로 작성하며, Controller에서 어떤 View 컴포넌트를 보여줄지 결정한다.
<br>
- Controller는 클라이언트의 요청을 받고, 적절한 Model에 지시를 내리며, Model에서 전달된 데이터를 적절한 View에 전달한다.
<br>

### Model1
![](https://github.com/qlalzl9/TIL/blob/master/Servlet_JSP/img/MVC_1.png)
- MVC에서 View와 Controller가 같이 있는 형태이다.
- 규모가 작고 유지보수보다는 속도에 중점을 둔 프로젝트에 많이 사용한다.
<br>

###Model2
![](https://github.com/qlalzl9/TIL/blob/master/Servlet_JSP/img/MVC_2.png)
- MVC에서 Mdel, View, Controller가 모두 모듈화 되어 있는 형태이다.
- 규모가 크고 유지보수를 계속 해야하는 프로젝트에  많이 사용한다.
- 보통 Model2를 많이 사용한다.
