>생활코딩의 [WEBn 강의](https://opentutorials.org/course/3083)를 듣고 중요 내용을 정리합니다.

### 1. 웹페이지에 CSS를 사용하는 방법
![](https://github.com/qlalzl9/TIL/blob/master/HTML_CSS/img/CSS_1.png)
- Selector(선택자) : 웹페이지에 있는 모든 a 태그를 '선택'의 의미.
- Declaration(선언) : 선택자가 지정한 태그들에서 어떤 '효과'를 줄 것인가를 의미.
- Property(속성), Value(값) : 효과의 내용.

### 2. CSS 속성
- 글자크기 - font-size : 70px;    // 글자 크기를 70px로 변경
- 정렬 - text-align : center;    // 가운데 정렬     (left, right, justify)
- 글자 색상 - color : orange;    // 글자 색상을 주황색으로 변경
- "red" , "#ff0000" , "rgb(255,0,0)" 등으로 표현 가능하다.
```css
div {
    font-size:70px;
    text-align:center;
    color:orange;
}
```

### 3. CSS Selector
- 우선순위 : id(#) → class(.) → tag
- 같은 선택자들 일 경우 가장 마지막에 오는 선택자가 적용된다.
- id 선택자가 한번만 등장하도록 권장된다.

### 4. CSS box model
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

### 5. grid
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

### 6. 반응형 디자인
```css
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

### 7. CSS 코드의 재사용
```css
<link rel="stylesheet" href="style.css">
```
- 지금까지의 내용을 하나의 .css파일로 만든 뒤 파일이 .css파일을 참고하게 해서 사용하면 코드의 재사용성을 높일 수 있다.


##### WEB2 - CSS 강의를 듣고 만든 페이지
[Jerry](https://qlalzl9.github.io/Jerry/)

![](https://github.com/qlalzl9/TIL/blob/master/HTML_CSS/img/CSS_4.png)

##### 코드
https://github.com/qlalzl9/Jerry
