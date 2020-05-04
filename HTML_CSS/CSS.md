### 웹페이지에 CSS를 사용하는 방법
![](https://github.com/qlalzl9/TIL/blob/master/HTML_CSS/img/CSS_1.png)
#### 1. inline
- HTML 태그 안에 style을 적용
- 다른 CSS파일에 적용한 것보다 먼저 작용한다.
```html
<p style="border:1px solid gray;color:red;font-size:2em;">
```
#### 2. internal
- style 태그로 따로 지정한다.
- 구조와 스타일이 섞이게 되어 유지보수가 어렵다.
- 별도의 CSS파일을 관리하지 않아도 되며 서버에 CSS파일을 부르기 위해 별도의 브라우저가 요청을 보낼 필요가 없다.
```html
<head>
<style>
p  {
  font-size : 2em;
  border:1px solid gray;
  color: red;
}
</style>
</head>

<body>
<div>...</div>
</body>
```

#### 3. external
- 외부파일(.css)로 지정하는 방식
- CSS 코드가 짧지 않다면 가급적 external 방법을 사용하는 것이 좋다.
- 별도의 .css 파일을 작성하고 아래 코드와 같이 link 태그로 추가한다.
```html
<html>
	<head>
		<link rel="stylesheet" href="style.css">
	</head>
	<body>
		<div>
			<p>
				<ul>
					<li></li>
					<li></li>
					<li></li>
					<li></li>
				</ul>
			</p>
		</div>
	</body>
</html>
```
#### 용어
- Selector(선택자) : 웹페이지에 있는 모든 해당 태그를 '선택'의 의미.
- Declaration(선언) : 선택자가 지정한 태그들에서 어떤 '효과'를 줄 것인가를 의미.
- Property(속성), Value(값) : 효과의 내용.


### CSS Selector
- 우선순위 : `inline → id(#) → class(.) → tag`
- 같은 선택자들 일 경우 가장 마지막에 오는 선택자가 적용된다.
- id 선택자가 한번만 등장하도록 권장된다.
<br>

### 스타일 상속
  * 웹 문서에서 사용하는 여러 태그들은 서로 포함 관계가 있다.
  * 스타일 시트에서 자식 요소에서 별도로 스타일을 지정하지 않으면 부모 요소에 있는 스타일 속성들이 자식 요소로 전달된다.<br>
  (ex `body` 태그 스타일을 바꾸면 `h1`, `h2` 태그에도 적용된다.)
  * 스타일의 모든 속성이 부모 요소에서 자식 요소로 상속되는 것은 아니다.<br>
  (글자 색은 상속되지만 배경 이미지나 배경색은 상속되지 않는다.)
<br>

### 텍스트 관련 스타일

#### 글꼴 관련 스타일

- font-family
  * `font-family: 굴림`
  * 글꼴을 지정
<br>

- font-size
  * `font-size:3em | 15px | 100%`
  * 글자 크기를 조절
<br>

- font-weight
  * `font-weight: bold | 100 | 700`
  * 글자 굵기 지정
<br>

- font-style
  * `font-style: normal | inalic | oblique`
  * 글자 스타일 지정
<br>

#### 텍스트 스타일

- color
  * `color: rgb(0, 200, 0) | blue | #ff0000`
  * 글자 색 지정
<br>

- text-decoration
  * `text-decoration: none | underline| overline| line-through`
  * 텍스트에 줄 표시, 없애기
<br>

- text-transform
  * `text-transform: none | uppercase | lowercase`
  * 텍스트 대-소문자 변환
<br>

- text-shadow
  * `text-shadow: none | <가로거리> <세로 거리> <번짐 정도> <색상>`
  * 텍스트에 그림자 효과 추가
<br>

#### 문단 스타일

- text-align
  * `text-align: start | end | left | right | center | justify`
  * 텍스트 정렬
<br>

- line-height
  * `line-height: normal | 30px | 2.0 | 200%`
  * 줄간격 조절
<br>

#### 목록 스타일

- list-style-type
  * `list-height: none | disc | circle | square`
  * 목록의 불릿과 번호 스타일 지정
<br>

### 색상과 배경 스타일

#### 색상 표현
- 16진수 표기법
  * #RRGGBB 형식으로 표현한다. 각각 빨간색, 초록색, 파란색의 양을 표시
  * #000000(검은색) ~ #ffffff(흰색)
  * 두 자리씩 중복 될 경우 줄여서 쓰기 가능 (#000, #fff)

- rgb, rgba 표기법
  * rgb(255, 0, 0) 또는 rgba(255, 0, 0, 0.5)와 같이 표기하는 방법
  * 10진수로 표현하며 0 ~ 255까지 가능
  * rgba 중 a는 투명도로 0 ~ 1까지 표현 가능 (1 : 불투명, 0 : 투명)

- hsl, hsla 표기법
  * hsl은 차례로 hue(색상), saturation(채도), lightness(밝기)를 나타낸다.

- 색상 이름 표기법
  * red, yellow, black 과 같이 알려진 색상 이름으로 표시

\* 색상 추출 사이트 활용<br>
  * [Color Picker](www.colorpicker.com) 과 같은 사이트를 이용하면 사용할 때 마다 원하는 색상의 정확한 값을 얻을 수 있다.
<br>

#### 배경색과 배경 이미지
- backgroud-color 속성
  * 배경 색 지정을 할 수 있다.
  * background-color 값은 상속되지 않는다.
<br>

- background-clip 속성
  * ` background-clip: border-box | padding-box | content-box`
  * 배경 적용 범위 조절
  * 박스 모델 관점에서 배경 적용 범위를 조절할 수 있다.
<br>

- background-image 속성
  * `backgroud-image: url('bg1.png')`
  * 웹 요소에 배경 이미지 넣기
  * 현재 웹 문서를 기준으로 상대 경로와 'http://'로 시작하는 절대 경로 둘 다 사용 가능
  * 만약 이미지가 채우려는 요소 크기보다 작을 경우 가득 채울 정도로 가로와 세로가 반복된다.
<br>

- background-size 속성
  * `background-size: auto | contain | cover | <크기 값> | <백분율>`
  * 배경 이미지 크기 조절
<br>

- background-position 속성
  * `background-position: <수평 위치> <수직 위치>`
  * 배경 이미지의 위치를 조절해서 한쪽에 이미지를 표시할 수 있다.
<br>

- background-attachment 속성
  * `background-attachment : scroll | fixed`
  * 속성 값을 fixed로 하면 스크롤을 해도 배경 이미지는 그대로 유지된다.
<br>

\* background 속성을 하나로 쓸 수 있다. <br>
속성 값이 다르므로 순서 상관없이 나열하고 나열되지 않은 속성은 기본값으로 읽는다.
<br>

### CSS box model
![](https://github.com/qlalzl9/TIL/blob/master/HTML_CSS/img/CSS_2.png)
- Margin, Padding 을 top, right, bottom, left 등으로 조절할 수 있다.
- block level element : 화면 전체를 크기로 가지는 태그<br>
ex) `<p>`, `<hn>`, `<ul>`, `<ol>`, `<div>`, `<blockquote>`, `<form>`, `<hr>`, `<table>`, `<fieldset>`, `<address>` 등
- inline element : 자신의 크기만을 크기로 가지는 태그<br>
ex) `<img>`, `<object>`, `<br>`, `<sub>`, `<span>`, `<input>`, `<textare>`, `<label>`, `<button>` 등
<br>

- width, height 속성
  * `width or height : <크기> | <백분율>`
  * 박스 모델에서 콘텐츠 영역의 크기를 지정
<br>

- display 속성
  * `display : none | block | inline | inline-block`
  * 화면 배치 방법 결정
<br>

#### 테두리 관련 속성
- border-style 속성
  * `border-style : none | solid | dashed | dotted`
  * 테두리 스타일 지정(기본값 : none)
<br>

- border-width 속성
  * `border-width: <크기> | thin | medium | thick`
  * 테두리 두께 지정
<br>

- border-color 속성
  * `border-color: <색상>`
  * 테두리 색상 지정
<br>

\* 색상 추출 사이트 활용<br>
  * ex) `border: 3px solid #ccc`
  * 테두리의 스타일, 두께, 색상 묶어서 표현할 수 있다.
<br>

- border-radius 속성
  * `border-radius: <크기> | <백분율>`
  *  박스 모서리 둥글게 만들기
  * top, left, right, bottom으로 4방향 각각 적용 가능하다.
<br>

- border-shadow 속성
  * `border-shadow: none | <수평 거리> <수직 거리> <흐림 정도> <번짐 정도> <색상>`
  * 박스에 그림자 효과 내기
  * 수평 거리와 수직 거리는 반드시 지정해야하며 나머지는 옵션이다.
<br>

#### 여백 조절 속성
- margin 속성
  * `margin: <크기> | <백분율> | auto`
  * 요소 주변 여백 설정
  * top, left, right, bottom으로 4방향 각각 적용 가능하다.
<br>

- margin 중첩 현상
  * 여러개의 박스가 세로로 배치되어 있을 경우 요소 사이의 마진이 너무 커지는 것을 방지하기 위해 큰 margin 값으로 합쳐진다.
  * 가로로 배치할 경우 중첩 현상은 생기지 않는다.
<br>

- padding 속성
  * `padding : <크기> | <백분율> | auto`
  * 콘텐츠 영역과 테두리 사이 여백 설정
  * margin과 마찬가지로 top, left, right, bottom으로 4방향 각각 적용 가능하다.
<br>

### CSS 레이아웃

- box-sizing 속성
  * `box-sizing: content-box | border-box`
  * 박스 너비 기준을 정하는 속성(기본 값은 content-box)
  * content-box : 콘텐츠 영역 너비 값으로 width 속성을 사용<br>
  border-box : 콘텐츠 영역에 테두리까지 포함한 박스 모델 전체 너비 값으로 width 속성을 사용
<br>

- float 속성
  * `float: left | right | none`
  * 웹 요소를 문서 위에 떠 있게 만든다. 왼쪽이나 오른쪽으로 배치할 때 주로 사용한다.
<br>

- clear 속성
  * `clear: none | left | right | both`
  * float 속성 해제하기
  * float 속성을 사용하면 그 다음 요소들에도 똑같은 속성이 전달하기 때문에 다음 요소의 적절한 배치를 위해 clear 속성을 사용한다.
<br>

- position 속성
  * `position: static | relative | absolute | fixed`
  * 웹 문서 안의 요소들을 자유자재로 배치해주는 배치 방법을 지정하는 속성
  * static을 제외한 나머지 속성 값에서는 좌표를 이용해 top, bottom, left, right로 지정할 수 있다.
  * static : 문서의 흐름대로 배치<br>
  relative : 문서 흐름 따라 위치 지정<br>
  absolute : 원하는 위치에 배치(기준이 되는 위치는 가장 가까운 부모 요소 중 position이 relative인 요소)<br>
  fixed : 브라우저 창 기준으로 고정시켜 배치
<br>

- visibility 속성
  * `visivility: visible | hidden | collapse`
  * 요소를 보이게 하거나 보이게 않게 하기
  * 요소를 사용자에게만 보이지 않게 하여 배치를 적절하게 가능
<br>

- z-index 속성
  * `z-index: <숫자>`
  * 요소 위에 다른 요소를 쌓는 속성
  * z-index 값이 작을수록 아래에 쌓인다.
<br>

#### 표 스타일

- caption-side 속성
  * `caption-side: top | bottom`
  * 표 제목 위치 정하기
<br>

- border 속성
  * ex) `border: 1px solid black`
  * 표 테두리 스타일 설정
<br>

- border-collapse 속성
  * `border-collapse: collapse | separate`
  * 테두리 통합, 분리하기
<br>

- border-spacing 속성
  * `border-spacing: <크기>`
  * 인접한 셀 테두리 사이 거리 지정
<br>

- empty-cells 속성
  * `empty-cells: show | hide`
  * 빈 셀의 표시 여부 지정
<br>

- width, height 속성
  * ex) `table {width: 300px; height: 200px;}`
  * 표 너비와 높이 지정
<br>

- table-layout 속성
  * `table-layout: fixed | auto`
  * 콘텐츠에 맞게 셀 너비 지정
<br>

- text-align 속성
  * `text-align: left | right | center`
  * 셀 안에서 수평 정렬하기
<br>

- vertical-align 속성
  * `vertical-align: baseline | top | bottom | middle | ... | <길이 값> | <백분율 값>`
  * 셀 안에서 수직 정렬하기
<br>

### 기타 CSS 선택자

#### 연결 선택자
- 선택자와 선택자를 연결해 적용 대상을 한정시키는 선택자
- combination selector라고도 한다.

- 하위 선택자(descendant selector) 
  * `selector selector { 속성: 속성값; }`
  * 자식 태그 뿐만 아니라 자식의 자식 태그까지 모두 스타일이 적용된다.
<br>

- 자식 선택자(child selector)
  * `selector > selector { 속성: 속성값; }`
  * 바로 아래 태그, 즉 자식 태그에만 스타일이 적용된다.
<br>

- 인접 형제 선택자(adjacent selector)
  * `selector + selector { 속성: 속성값; }`
  * selector 이후 맨 먼저오는 selector에 스타일을 적용한다.
  * ex) `h1 + p { text-decoration: underline; }`으로 지정하면 h1 태그 다음에 오는 p 태그 중 첫번째 p 태그에만 밑줄을 긋는다.
<br>

- 형제 선택자(sibling selector)
  * `selector ~ selector { 속성: 속성값; }`
  * ex) `h1 ~ p { text-decoration: underline; }`으로 지정하면 h1 태그 다음에 오는 모든 p 태그에 밑줄을 긋는다.
<br>

#### 속성 선택자

- [속성] 선택자
  * `[속성] { 속성: 속성값; }`
  * 지정한 속성을 가진 요소를 찾아 스타일을 적용한다.
  * ex) `a[href]`로 지정하면 a 태그 중 href 속성이 있는 요소에만 스타일을 지정한다.
<br>

- [속성 = 값] 선택자
  * `[속성=속성값] { 속성: 속성값; }`
  * ex) `a[targer="_blank"]로 지정하면 a 태그 중 target 속성의 값이 _blank인 요소에만 스타일을 지정한다.
<br>

- [속성 ~= 값] 선택자
  * `[속성~=속성값] { 속성: 속성값; }`
  * ex) `[class ~="button"]`로 지정하면 `class="button"`처럼 값이 정확히 일치하거나 `class="flat button"`처럼 `"button"` 값이 포함되어 있을 경우 스타일을 적용한다.
  * `class="flat-button"`처럼 하이픈으로 연결되어 있거나 `class="buttons"`처럼 일부만 일치하는 경우에는 스타일이 적용되지 않는다.
<br>

- [속성 |= 값] 선택자
  * `[속성 |=속성값] { 속성: 속성값; }`
  * 특정 값이 포함된 속성에 스타일을 적용한다.
  * ~=와 달리 하이픈으로 연결한 단어여도 값으로 시작하는 값- 의 형태면 스타일을 적용한다.
<br>

- [속성 ^= 값] 선택자
  * `[속성 ^=속성값] { 속성: 속성값; }`
  * 특정 값으로 시작하는 속성에 스타일을 적용한다.
  * 값으로 시작하는 값~ 의 형태면 스타일을 적용한다.
<br>

- [속성 $= 값] 선택자
  * `[속성 $=속성값] { 속성: 속성값; }`
  * 특정 값으로 끝나는 속성에 스타일을 적용한다.
  * 값으로 끝나는 ~값 의 형태면 스타일을 적용한다.
<br>

- [속성 *= 값] 선택자
  * `[속성 *=속성값] { 속성: 속성값; }`
  * 값의 일부가 일치하는 속성에 스타일을 적용한다.
  * 값이 어느 위치에든 일치하기만 하면 스타일이 적용된다.
<br>

#### 가상 클래스 선택자

- `:link` 선택자
  * 문서의 링크 중 방문하지 않은 링크에 스타일 적용
  * 텍스트 링크는 기본적으로 파란색 글자와 밑줄로 표시되는데 링크의 밑줄을 없애거나 색상을 바꾸려고 할 때 :link 선택자를 사용한다.
<br>

- `:visited` 선택자
  * 문서의 링크 중 이미 방문한 링크에 스타일을 적용한다.
<br>

- `:hover` 선택자
  * 웹 요소 위로 마우스 커서를 올려놓을 때의 스타일을 지정한다.
  * 응용하면 이미지 위로 커서를 올려놓으면 다른 이미지로 바뀌었다가 커서를 치우면 원래 이미지로 돌아오는 롤오버(rollover) 효과를 만들 수 있다.
<br>

- `:active` 선택자
  * 링크나 이미지 등 웹 요소를 활성화했을 때(누르고 있을 떄)의 스타일을 지정한다.
<br>

* `:focus` 선택자
  * 웹 요소에 초점이 맞추어졌을 때의 스타일을 지정한다.
  * 초점이 맞추어졌다는 것은 텍스트 필드 안에 마우스 커서를 갖다 놓거나 Tab을 눌러 초점을 이동했을 때를 말한다.
<br>

- `:enabled` 선택자와 `:disabled` 선택자
  * `:enabled` : 해당 요소가 사용 가능한 상태일 때의 스타일을 지정한다. 
  * `:disabled` : 해당 요소가 사용 불가능한 상태일 때의 스타일을 지정한다.
  * 예를 들어 텍스트를 입력해야 할 텍스트 영역 필드는 enabled 상태로, 회원 약관 등 사용자가 내용을 보기만 해야할 경우 disabled 상태로 만들어야 한다.
<br>

- `:checked` 선택자
  * 라디오 박스나 체크 박스에서 사용자가 해당 항목을 선택(체크)했을 때의 스타일을 지정한다.
<br>

- `:root` 선택자
  * 문서 안의 root 요소에 스타일을 적용한다.
  * HTML 문서의 경우 root 요소가 HTML이므로 HTML 요소에 스타일이 적용된다.
<br>

- `:first-child` 선택자와 `:last-child` 선택자
  * `:first-child` : 첫 번째 자식 요소를 선택해 스타일을 적용한다.
  * `:last-child` : 마지막 자식 요소를 선택해 스타일을 적용한다.
<br>

- `:not` 선택자
  * 특정 요소가 아닐 때 스타일을 적용한다.
  * ex) `p:not(#ex) { color: blue; }`는 #ex가 아닌 모든 p 태그에 글자 색을 blue로 바꾼다.
<br>

- `::before`와 `::after`
  * 특정 요소의 내용 앞(`::before`) 또는 뒤(`::after`)에 지정한 내용을 넣을 수 있다.
<br>

### 반응형 디자인

- 뷰포트 지정하기
  * `<meta nmae="viewport" content="<속성1=값>, <속성2=값2>, ... ">`
  * 뷰포트 : 스마트폰 화면에서 실제 내용이 표시되는 영역
  * ex) `<meta name="viewport" content="width=device-width, initial-scale=1">`
  * 뷰포트의 너비를 스마트폰 화면 너비에 맞추고 초기화면 배율을 1로 지정한 것으로 가장 많이 사용하는 형태의 예시
<br>

#### 가변 그리드

- 가변 그리드 레이아웃
  * px을 이용해 크기를 고정시키는 것이 아닌 문서의 맨 바깥 부분을 묶어 전체 px을 정한 다음, 그 하위의 요소들을 각각 백분율(%)로 배치한다.
<br>

- 텍스트
  * 텍스트의 크기를 px 단위로 하면 크기가 고정되기 때문에 화면 크기가 작은 기기에서 매우 작게 표시된다.
  * 따라서 em과 rem 단위를 사용한다.
  * em : 부모 요소에서 지정한 폰트의 너비를 1em으로 지정한 것(1em = 16px)
  * rem : em은 부모 요소의 폰트 크기에 영향을 받기 때문에 처음부터 기본 크기를 정하여 기준을 정한다.
<br>

- 이미지
  * 텍스트와 마찬가지로 px을 사용하지 않고 백분율(%)을 사용한다. 
  * 고해상도 이미지를 크기만 줄여 모바일에 표시하면 단점이 많아 `<img>`태그와 `srcset`속성을 사용하여 화면 너비 값이나 픽셀 밀도에 따라 고해상도의 이미지 파일을 지정할 수 있다.
  * `<img src="<이미지>" srcset="<이미지1>[, <이미지2>, <이미지3>, ...]">`
<br>

- 비디오
  * 이미지와 마찬가지로 백분율(%)을 이용하여 크기를 적절하게 조절할 수 있다.
<br>

### grid
```html5
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title></title>
    <style>
      #grid {
        border:5px solid pink;
        display:grid;
        grid-template-columns: 150px 1fr;
      }
      div {
        border:5px solid gray;
      }
    </style>
  </head>
  <body>
    <div id="grid">
      <div>NAVIFATION</div>
      <div>ARTICLE</div>
    </div>
  </body>
</html>
```
![](https://github.com/qlalzl9/TIL/blob/master/HTML_CSS/img/CSS_3.png)
- grid layout을 사용하여 선택자들 간의 배치를 다양하게 할 수 있다.
- 위의 코드에서 NAVIGATION은 150px을 가지고 나머지를 ARTICLE이 채운다.
<br>

### 반응형 디자인
```html
<style>
      div{
        border:10px solid green;
        font-size:60px;
      }
      @media(max-width:800px) {
        div{
          display:none;
        }
      }
</style>
```

- mediaquery를 사용하면 창의 크기에 따라 반응하는 웹 페이지를 만들 수 있다.
- 예제 코드에서는 스크린의 크기가 800px보다 작아지면 div의 내용이 사라지게 만들었다.
<br>