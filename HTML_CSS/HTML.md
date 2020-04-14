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
	* Inline level tag이기 때문에 줄바꿈 없이 다른 내용과 함께 한 줄로 표시되고 인용 내용6을 구별할 수 있도록 인용 내용에 따옴표를 붙여 표시한다.
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
	* 표의 제목을 붙이는 태그로 < caption > 과는 달리 중앙에 정렬되지 않는다.
	* < table > 보다 앞에 사용하면 표 위에, </ table > 태그 뒤에 사용하면 표 아래에 제목이 표시된다.
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
	* `<figure>`로 요소를 묶고 `<figcaptio>`으로 설명 글을 붙인다.
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

### 기타 태그
- `<u></u>`
	* 밑줄을 만든다.
	<br>

- `<img src="coding.jpg" width="100%">`
	* coding.jpg라는 이미지를 첨부하고 크기 조절까지 가능
	<br>

- `<title></title>`
	* 웹페이지의 제목 설정. 검색엔진은 title을 기준으로 하기 때문에 title을 꼭 정해주어야 한다. title을 안정하면 굉장히 손해다.
	<br>

- `<meta charset="utf-8">`
	* 브라우저에게 utf-8이란 방법으로 문서를 읽어라.라는 뜻으로 한글이 깨지지 않게 해 준다.
	<br>

- `<head></head>`
	* 본문을 설명하는 것을 묶어서 head라고 한다.
	<br>

- `<body></body>`
	* 본문을 body라고 한다. 따라서 html의 모든 태그는 head 또는 body 아래에 놓이게 된다.
	<br>

- `<html></html>`
	* head와 body를 감싸는 최고위층 태그이다.
	<br>

- `<!doctype html>`
	* 이 문서는 html 문서다.라는 뜻으로 관용적으로 html 태그 위에 쓰인다.
	<br>

- `<a href= "http://www.w3.org/TR/2011/WD-html5-20110405/" target="_blank" title=“html5 specification“>Hypertext Markup Language(HTML)</a>`
	* 이때 `<a></a>`는 링크 태그이고 `<a href=“주소”`하면 주소로 하이퍼텍스트를 해주고 target을 “_blank”로 하면 새 탭에서 열린다. 그리고 title=“툴팁 내용”하면 주소를 올렸을 때 툴팁이 표시된다.