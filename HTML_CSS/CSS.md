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
- Selector(선택자) : 웹페이지에 있는 모든 a 태그를 '선택'의 의미.
- Declaration(선언) : 선택자가 지정한 태그들에서 어떤 '효과'를 줄 것인가를 의미.
- Property(속성), Value(값) : 효과의 내용.


### CSS Selector
- 우선순위 : inline → id(#) → class(.) → tag
- 같은 선택자들 일 경우 가장 마지막에 오는 선택자가 적용된다.
- id 선택자가 한번만 등장하도록 권장된다.
<br>

### 스타일 상속
  * 웹 문서에서 사용하는 여러 태그들은 서로 포함 관계가 있다.
  * 스타일 시트에서 자식 요소에서 별도로 스타일을 지정하지 않으면 부모 요소에 있는 스타일 속성들이 자식 요소로 전달된다.<br>
  (ex body 태그 스타일을 바꾸면 h1, h2 태그에도 적용된다.)
  * 스타일의 모든 속성이 부모 요소에서 자식 요소로 상속되는 것은 아니다.<br>
  (글자 색은 상속되지만 배경 이미지나 배경색은 상속되지 않는다.)
<br>

### 텍스트 관련 스타일

#### 글꼴 관련 스타일

- font-family
  * `selector {font-family:굴림;}`
  * 글꼴을 지정 
<br>

- font-size
  * `selector {font-size:3em | 15px | 100%;}`
  * 글자 크기를 조절
<br>

- font-weight
  * `selector {font-weight: bold | 100 | 700;}` 
  * 글자 굵기 지정
<br>

- font-style
  * `selector {font-style: normal | inalic | oblique;}`
  * 글자 스타일 지정
<br>

#### 텍스트 스타일

- color
  * `selector {color:rgb(0, 200, 0) | blue | #ff0000;}`
  * 글자 색 지정
<br>

- text-decoration
  * `selector {text-decoration:none | underline| overline| line-through;}`
  * 텍스트에 줄 표시, 없애기
<br>

- text-transform
  * `selector {text-transform: none | uppercase | lowercase;}`
  * 텍스트 대-소문자 변환
<br>

- text-shadow
  * `selector {text-shadow: none | <가로거리> <세로 거리> <번짐 정도> <색상>;}`
  * 텍스트에 그림자 효과 추가
<br>

#### 문단 스타일

- text-align
  * `selector {text-align: start | end | left | right | center | justify;}`
  * 텍스트 정렬
<br>

- line-height
  * `selector {line-height: normal | 30px | 2.0 | 200%;}`
  * 줄간격 조절
<br>

#### 목록 스타일

- list-style-type
  * `selector {list-height: none | disc | circle | square;}`
  * 목록의 불릿과 번호 스타일 지정
<br>

### CSS box model
![](https://github.com/qlalzl9/TIL/blob/master/HTML_CSS/img/CSS_2.png)
- Margin, Padding 을 top, right, bottom, left 등으로 조절할 수 있다.
```css
div {
  padding-top: 50px;
  padding-right: 30px;
  padding-bottom: 50px;
  padding-left: 80px;
}
```
- border 속성을 활용하여 테두리를 표현할 수 있다.
```css
div {
  border-style: solid;
  border-width: 5px;
  border-right: none;
  border-left : 10px;
}
```
- block level element : 화면 전체를 크기로 가지는 태그
- inline element : 자신의 크기만을 크기로 가지는 태그
```css
div {
    display: none;    //화면에 사라지게 함
    display: block;
    display: inline;
}
```
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

### CSS 코드의 재사용
```html
<link rel="stylesheet" href="style.css">
```
- 지금까지의 내용을 하나의 .css파일로 만든 뒤 파일이 .css파일을 참고하게 해서 사용하면 코드의 재사용성을 높일 수 있다.
