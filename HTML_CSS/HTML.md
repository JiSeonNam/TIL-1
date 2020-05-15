## 텍스트 관련 태그

### Block level tags
- `<hn></hn>`
	* 제목 태그이다. 글씨가 두껍고 커지고 줄 바꿈이 자동 실행된다. h1에서 h6까지 있고 6으로 갈수록 작아진다.
	<br>

- `<p></p>`
	* 문단을 구분하는 역할. br과 비슷하지만 중복해서 써도 1번만 줄 바꿈이 일어난다. 둘 중에 의미에 맞는 태그를 사용하는 게 알맞다.
	<br>

- `<br>`
	* 강제로 줄을 바꾸는 역할. 닫는 태그는 없다.
	* 중복 사용이 가능하며. 3번 쓰면 3번 줄 바꿈이 일어난다.
	<br>

- `<hr>`
	* 수평 줄을 삽입할 때 사용한다. 단락의 주제가 바뀔 때 분위기 전환용이다. 닫는 태그는 없다.
	<br>

- `<blockquote></blockquote>`
	* 글을 인용할 때 사용한다.
	* block level tag이기 때문에 인용 내용이 줄이 바뀌어 나타나고 다른 내용과 구별 되도록 안으로 들여 써진다.
	<br>

- `<pre></pre>`
	* 입력하는 그대로를 화면에 표시한다.
	* 화면 낭독기가 인식하지 못함
	<br>

### Inline level tags
- `<strong></strong>`
	* 글씨를 진하게 한다.
	* 일반적으로 경고나 주의 사항처럼 중요한 내용일 때 사용한다.
	<br>

- `<b></b>`
	* 글씨를 진하게 한다.
	* 문서의 키워드처럼 단순히 진하게 표시하고 싶을 때 사용
	* 화면 낭독기가 인식하지 못함
	<br>

- `<em></em>`
	* 텍스트를 비스듬히 표시한다.
	* 문장에서 흐름상 특정 부분을 강조하고 싶을 때 사용
	<br>

- `<i></i>`
	* 텍스트를 비스듬히 표시한다.
	* 마음 속의 생각이나 꿈, 기술적인 용어, 다른 언어의 관용구 등에 사용한다.
	<br>

- `<q></q>`
	* 글을 인용할 때 사용한다.
	* Inline level tag이기 때문에 줄바꿈 없이 다른 내용과 함께 한 줄로 표시되고 인용 내용을 구별할 수 있도록 인용 내용에 따옴표를 붙여 표시한다.
	<br>

- `<mark></mark>`
	* 노란색으로 형관펜 효과를 낸다.
	* CSS의 background-color 속성을 사용하면 색 변경이 가능하다.
	<br>

- `<span></span>`
	* 태그 자체로는 아무 의미 없지만 텍스트 단락 안에서 줄바꿈 없이 일부 텍스트만 묶어 스타일을 적용하려고 할 떄 주로 사용한다.
	<br>

- `<ruby></ruby>`
	* 동아시아 국가들의 주석을 함께 표기하기 위한 용도로 사용되며 주석으로 표시할 내용을 < ruby > 태그안에 < rt >로 표시한다.

### 목록 태그
- `<li></li>`
	* 목차를 만들어준다.
	<br>

- `<ul></ul>` (Unordered List)
	* 목차끼리 순서 없이 구분하는 역할. ul과 li는 부모-자식 태그이다.
	<br>

- `<ol></ol>` (Ordered List)
	* ul을 ol로 바꿔주면 숫자순서가 자동으로 매겨진다. ol과 li는 부모-자식 태그이다.
	<br>

### 표를 만드는 태그
- `<table></table>`
	* 표 자리를 만든다.
	<br>

- `<tr></tr>`
	* 행을 만든다.
	<br>

- `<td></td>`
	* 열을 만든다.
	<br>

- `<th></th>`
	* 표에 제목 셀을 만든다.
	* 셀의 중앙에 배치하고 굵게 표시한다.
	<br>

- colspan, rowspan 속성 - 행 또는 열 합치기
	* colspan 속성을 사용하면 여러 열을 하나의 열로 합칠 수 있다.
	* rowspan 속성을 사용하면 여러 행을 하나의 행으로 합칠 수 있다.
	<br>

- `<caption></caption>`
	* 표의 제목을 붙이는 태그로  < table > 태그 바로 다음에 사용한다.
	* 표 제목은 표의 위쪽 중앙에 표시된다.
	* 태그 안에 다른 태그를 사용해 제목을 여러 줄로 표시하거나 텍스트를 꾸밀 수 있다.
	<br>

- `<figcaption></figcaption>`
	* 표의 제목을 붙이는 태그로 `<caption>` 과는 달리 중앙에 정렬되지 않는다.
	* `<table>` 보다 앞에 사용하면 표 위에, `</table>` 태그 뒤에 사용하면 표 아래에 제목이 표시된다.
	<br>

### 이미지와 하이퍼링크
<br>

#### 이미지
- `<img>`
	* 웹 문서에 이미지를 삽입 할 때 사용
	* src 속성을 사용해 파일이 있는 경로를 알려 주어야 화면에 이미지를 표시할 수 있다.
<br>

- src 속성
	* 이미지 파일 경로를 지정
	* ex) `<img src="images/ex.jpg">`
<br>

- alt 속성
	* 이미지를 설명하는 대체 텍스트를 삽입할 때 사용
	* ex) `<img src="images/ex.jpg" alt="예시입니다">`
<br>

- width, height 속성
	* 이미지의 너비와 높이를 설정할 때 사용
	* 설정하지 않으면 원본 크기로 삽입된다.
	* ex) `<img src="images/ex.jpg" alt="예시입니다" width="250" height="250">`
<br>

- `<figure>`, `<figcaption>` 속성
	* 이미지에 설명 글을 삽입할 때 사용
	* `<figure>`로 요소를 묶고 `<figcaption>`으로 설명 글을 붙인다.
	* 이미지만 삽입한다면 `<figure>`태그를 사용하지 않아도 된다.
	```html
	<!-- 예시코드 -->
	<figure>
		<img src="images/ex.jpg" alt="예시입니다" width="250px" height="250px">
		<figcaption>이미지 설명 글입니다.</figcaption>
	</figure>
	```
<br>

#### 하이퍼링크

- `<a>`태그, href 속성
	* 링크를 만들 때 사용
	* ex) `<a href="링크 주소">텍스트</a>`
	* 이미지 링크 만들기 : `<a href="링크 주소"><img src="이미지 파일 경로"></a>`
<br>

- target 속성
	* 현재 화면 뿐만 아니라 새로운 화면에서도 링크를 열 수 있게 할 때 사용
	* ex) `<a href="링크 주소" target="_blank">텍스트</a>`
<br>

- 앵커 만들기
	* 앵커(anchor)는 페이지가 긴 웹 문서에서 특정 요소를 클릭하면 페이지 내에서 그 위치로 한 번에 이동시켜 준다.
	* `<태그 id="앵커 이름">텍스트 또는 이미지</태그>`<br>
	`<a href="#앵커 이름"> 텍스트 또는 이미지</a>`
	```html
	<ul id="Contents">
		<li><a href="#content1">목차1</a></li>
		<li><a href="#content2">목차2</a></li>
		<li><a href="#content3">목차3</a></li>
	</ul>

	<h2 id="content1">내용1</h2>
	<p>내용1</p>
	<p><a href="#Contents">[목차로]</a></p>

	<h2 id="content2">내용2</h2>
	<p>내용2</p>
	<p><a href="#contents">[목차로]</a></p>

	<h2 id="content3">내용3</h2>
	<p>내용3</p>
	<p><a href="#Contents">[목차로]</a></p>
	```
<br>

### 폼 관련 태그

- `<form></form>`
	* 특정 항목에 사용자가 뭔가 입력하는 형태로 사용하고자 할때 사용
	* 속성들을 이용해 사용자가 입력한 자료들을 서버로 어떤 방식으로 넘길 것인지, 어떤 프로그램을 이용해 처리할 것인지 정한다.
<br>

- form 태그의 속성
	* method : 사용자가 입력한 내용들을 서버 쪽 프로그램으로 어떻게 넘겨줄지 지정한다.
		- 속성 값 : get, post
	* name : 폼의 이름을 지정한다. 한 문서 안에 여러 개의 form 태그가 있을 경우, 그들을 구분하기 위해 사용
	* action : form 태그 안의 내용들을 처리해 줄 서버 상의 프로그램을 지정한다.
	* target : action 태그에서 지정한 스크립트 파일을 현재 창이 아닌 다른 위치에 열도록 조정한다.
	* autocomplete : 이미 입력했던 내용이 아래에 표시되도록 자동완성 기능을 제공한다. <br>
	기본값이 "on"이므로 개인정보를 입력할 경우 autocomplete 속성을 "off"로 하여 끌 수 있다.
```html
<!-- 예시코드 -->
<form action="login.jsp" method="post">
	<input type="text" title="아이디">
	<input type="text" title="비밀번호">
	<input type="submit" title="로그인">
</form>
```

- `<label></label>`
	* 폼 요소에 레이블을 붙이는 태그이다.
	* 레이블이란 입력 창 옆에 '아이디', '비밀번호' 와 같이 입력 창 옆에 붙여놓은 텍스트를 말한다.
	* ex) `<label>아이디(6자 이상)<input type="text" title="아이디"></label>` 또는<br>
	`<label for="user-id">아이디(6자 이상)</label>`<br>
	`<input type="text" id="user-id">`
	* 라디오 버튼과 체크박스에 사용되면 사용자가 버튼이나 박스 부분을 정확하게 클릭할 필요없이 연결된 텍스트만 클릭해도 체크된다.
<br>

- `<fieldset>`, `<legend>`
	* 폼 요소를 그룹으로 묶을 때 사용하며 하나의 폼 안에서 여러 구역을 나누어 표시함으로써 정보를 입력하기도 편하고 화면도 깔끔하게 정리할 수 있다.
	* `<fieldset>`태그는 `<fieldset></fieldset>` 태그 사이의 폼들을 하나의 영역으로 묶어 외곽선을 그려 주고
	* `<legend>` 태그는 `<fieldset>` 태그로 묶은 그룹에 제목을 붙여준다.
<br>

- `<input>` 태그
	* 사용자가 입력할 부분을 넣는 태그
	* type으로 구분하여 text, checkbox, radio 버튼, submit 버튼 등이 사용될 수 있다.
<br>

- `<input>` 태그의 type들
	* input 태그의 type들은 종류가 굉장히 많고 중요하기 때문에 자유롭게 다룰 줄 알아야한다.
	* [input 태그의 type들(W3schools)](https://www.w3schools.com/tags/att_input_type.asp)
<br>

- autofocus 속성
	* 페이지를 불러오자마자 폼의 요소 중에서 원하는 요소에 마우스 커서를 표시할 수 있다.
	* ex) `<input type="text" id="uname" autofocus required>`
<br>

- placeholder 속성
	* 입력란에 힌트 내용을 표시하고 있다가 클릭하면 사라지도록 만든다.
	* ex) `<input type="text" id="uid" placeholder="하이픈없이 입력">`
<br>

- readonly 속성
	* 입력란에 사용자가 입력하지 못하고 읽기만 가능하게 만든다.
	* true 또는 false 값만 가지며 `readonly`만 쓰거나 `readonly="readonly"`라고 써도 `readonly="true"`로 인식한다.
<br>

- required 속성
	* submit 버튼을 눌렀을 때 필요한 내용이 모두 채워졌는지 검사하는 속성
	* `required="required"` 또는 `required`
<br>

- size, minlength, maxlength 속성
	* size : 텍스트 필드에 보일 글자 수
	* minlength : 최소 입력 글자 수
	* maxlength : 최대 입력 글자 수
<br>

- `<select>`, `<option>`, `<optgroup>` 태그
	* 사용자가 내용을 입력하는 것이 아니라 여러 옵션 중에서 선택하도록 드롭다운 목록을 제공한다.
	* `<select>` 태그로 드롭다운 목록의 시작과 끝을 표시하고 그 안에 `<option>` 태그를 사용해 원하는 항목을 추가한다.
	* `<option>`태그에 value 속성을 이용해 서버로 넘겨주기 위한 값을 지정한다.
	* `<optgroup>`태그를 사용하면 드롭다운 목록에서 여러 항목들을 그룹을 묶을 수 있다.
<br>

- `<datalist>`, `<option>` 태그
	* `<select>`태그와 비슷하지만 목록에서 제시한 값을 선택하면 그 값을 자동으로 입력해준다.
<br>

- `<textarea>` 태그
	* 텍스트 영역을 만들어 한 줄 이상의 문장을 입력할 때 사용한다.
	* cols 속성으로 너비 크기, rows 속성으로 화면에 표시할 줄 수를 지정할 수 있다.
<br>

### HTML 시맨틱 태그

- `<header>` 태그
	* 주로 페이지 맨 위쪽이나 왼쪽에 삽입하며 헤더의 내용으로는 주로 `<form>` 태그를 사용해 검색 창을 넣거나 `<nav>`태그를 사용해 사이트 메뉴를 넣는다.
<br>

- `<nav>` 태그
  * 문서를 연결하는 네비게이션 태그
  * 동일한 사이트 안의 문서나 다른 사이트의 문서로 연결하는 링크 모음을 나타낸다.
  * `<header>`나 `<footer>`, `<aside>`태그 안에 포함시킬 수도 있고 독립해 사용할 수도 있다.
  * 문서 안의 여러 개의 `<nav>`태그를 사용할 경우, ID를 따로 지정해 주면 스타일 시트에서 각 네비게이션에 맞게 스타일을 지정할 수 있다.
<br>

- `<section>` 태그
  * 주제별 콘텐츠 영역 나타내기
  * 문맥 흐름 중에서 콘텐츠를 주제별로 묶을 때 사용하며 주로 제목을 나타내는 `<hn>`태그가 함께 사용된다.
  * `<section>` 태그 안에 또 다른 `<section>` 태그를 넣을 수도 있다.
<br>

- `<article>` 태그
  * 콘텐츠 내용 넣기
  * 태그를 적용한 부분을 떼어 내 독립적으로 배포하거나 재사용하더라도 완전히 하나의 콘텐츠가 될 때 `<article>` 태그를 쓴다.
  * `<section>` 태그와 비슷하지만 `<section>` 태그는 문맥 흐름 중에서 콘텐츠를 묶을 떄 사용한다.
  * `<article>` 태그 안에 `<section>` 태그를 넣을 수 있다.
<br>

- `<aside>` 태그
  * 본문 이외의 내용 표시하기
  * 필수 요소가 아닌 사이드바에 들어가는 내용을 넣을 때 사용한다.
  * CSS의 속성을 이용해 사이드바의 배치를 조정할 수 있다.
<br>

- `<iframe>` 태그
  * 외부 문서 삽입하기
  * 웹 문서 안에 다른 웹 문서를 가져와 표시하는 것을 inline frame이라고 한다. <br> inline frame을 삽입하는 태그가 `<iframe>` 태그이다.
<br>

- `<footer>` 태그
  * 제작 정보와 저작권 정보 표시하기
  * 일반적으로 웹 문서 끝자락에 들어가며 `<footer>` 태그 안에 `<header>`, `<section>`, `<article>` 등 다른 태그를 모두 사용할 수 있다.
<br>

- `<address>` 태그
  * 사이트 제작자 정보, 연락처 정보 나타내기
  * 주로 `<footer>` 태그 안에 사용되며 웹 사이트 관련 우편 주소도 `<address>` 태그 안에 포함시킨다.
<br>

- 시맨틱 태그가 나온 뒤 `<div>` 태그의 사용
  * 주로 콘텐츠를 묶어 시각적 효과를 적용할 때 즉 CSS를 적용할 때 `<div>` 태그를 사용한다.
<br>

### HTML과 멀티미디어

- `<object>` 태그와 `<embed>` 태그
	* `<object data="경로" type="유형" [name="이름" width="너비" height="높이"]></object>`
	* `<embed src="경로" type="유형" width="너비" height="높이">`
	* 웹 브라우저에서 처리할 수 없는 작업을 위해 플러그인을 사용하는 태그
	* `<object>`태그를 사용할 경우 특정 외부 파일을 가져와 표시할 것인지 여부를 알려 주기 위해 최소한 data나 type속성 둘 중 하나를 반드시 사용해야 한다.
	* `<object>`태그와는 달리 `<embed>`태그는 닫는 태그가 없으며 `<embed>`태그는 주로 `<object>`태그를 지원하지 않는 이전 브라우저에서 사용된다.
<br>

- `<audio>` 태그
	* `<audio src="오디오 파일 경로" [속성] [속성="속성 값"]></audio>`
	* HTML5에서 배경 음악이나 효과음 등 오디오 파일 삽입 태그
<br>

- `<video>` 태그
	* `<video src="비디오 파일 경로" [속성] [속성="속성 값"]></video>`
	* 비디오 파일 삽입 태그
<br>

- `<source>` 태그
	* ex) `<source src="video.ogv" type="video/ogg; codecs="'theora,vorbis'">`
	* 여러 미디어 파일 한꺼번에 지정하는 태그
<br>

- `<track>` 태그
	* `<track kind="자막종류" src="경로" srclang="언어" label="제목" default>`
	* 비디오 화면에 자막 추가하기
	* `<track>` 태그의 속성
		- kind : 자막의 종류를 지정
		- src : 자막 텍스트의 파일 경로 지정
		- srclang : 사용한 언어 지정(kind 속성 값이 subtitle이라면 반드시 지정해야 한다)
		- label : 자막이 여러 개일 경우, 자막을 식별할 수 있도록 제목을 달아준다.
		- default : 자막 파일이 여러 개일 경우, 기본으로 사용할 자막을 default로 지정
<br>
