---
title: "[Front End] 기초부터 완성까지 - 02장. HTML과 CSS (2)"
last_modified_at: 2022-03-29T19:00:00+09:00
categories:
    - Front End
    - 기초부터 완성까지, 프런트엔드
tags:
    - Front End
    - 기초부터 완성까지, 프런트엔드
toc: true
toc_sticky: true
toc_label: "목차"
---

기초부터 완성까지, 프런트엔드 개발도서 관련 : 02장 HTML과 CSS에 대해 정리한다. (2)에서는 CSS 대해서만 다루고 (1)에서 HTML에 대한 글을 볼 수 있다.
{: .notice--info}

# HTML과 CSS

집 짓기에 비유해보자면,<br>
**HTML**은 기둥, **CSS**는 꾸미는 역할, **자바스크립트**는 여러 시설을 동작하게 하는 것.

# CSS
: Cascading Style Sheets

- 문서를 표시하는 방법(스타일이나 레이아웃)을 지정하는 역할
- 시각적으로 아름답게 표현한다.

## 작성방법

```css
selector {
    property: value;
}
```

`selector`의 `property`를 `value`로 하겠다.

한 선택자에 여러 개의 프로퍼티를 부여하고 싶을 때 중괄호를 사용한다.

```css
selector {
    property1: value1;
    property2: value2;
}
```

외부 CSS 파일에 작성된 스타일을 HTML 페이지에 연결할 때는 **link 태그** 이용

**href** 속성을 통해 CSS 파일의 위치를 지정<br>
**rel**로 관계를 표현 (CSS는 rel의 값으로 stylesheet을 넣는다.)

```html
<!DOCTYPE html>
<html lang="ko">
    <head>
        <!-- 외부 파일에 정의된 css 연결하기 -->
        <link rel="stylesheet" href="./main.css" type="text/css" />
    </head>
    <body>...</body>
</html>
```

CSS 프로퍼티는 항상 .css 확장자를 가진 파일에서만 작성 가능한것은 아니다.

HTML 내의 `<style>` 태그 안에 작성하고나 HTML 요소에 바로 작성 가능하다.

```html
<head>
    <style>
        h1 {
            color: blue;
        }
    </style>
</head>
```

```html
<h2 style=""color: skyblue; font-size: 14px;">Hello</h2>
```

## 상속

CSS 프로퍼티 중 상위요소에 적용되었지만, 자식 요소까지 상속되는 프로퍼티들이 있다.

반대로 상속되지 않는 프로퍼티도 존재하며, 상속되는 프로퍼티지만 HTML 요소의 종류에 따라 상속되지 않기도 한다.

1. 상속된 경우(font)
    ```html
    <div style="font-size: 20px;">
        parent
        <!-- child 또한 font-size가 20px로 노출 -->
        <p>child</p>
    </div>
    ```
2. 상속되지 않는 경우(width, height, margin, display, border, ...)
    ```html
    <div style="border: 1px solid #ccc;">
        parent
        <!-- child에는 border가 적용되지 않는다.-->
        <p>child</p>
    </div>
    ```
3. 상속되는 경우지만 상속이 안된 경우(button)
    ```html
    <div style="font-size: 20px">
        parent
        <!-- button 에는 font-size 가 적용 안 됨 -->
        <button>child</button>
    </div>
    ```
4. 종류에 영향받지 않고 부모 요소를 상속받는 경우(inherit)
    ```html
    <div style="font-size: 30px">
        parent
        <!-- button 에는 font-size 가 적용 됨 -->
        <button style="font-size: inherit">child</button>
    </div>
    ```
   
## 선택자

- 어느 요소에 스타일을 적용할지 대상을 뜻하는 패턴
- 선택자를 작성하는 방법은 여러 가지 존재. 
- 지정하는 방식에 따라 명시도나 우선순위가 다름

### 전체 선택자

- '*' 와일드 카드라 불림
- 전체 선택자 사용시 페이지에 해당하는 모든 요소가 영항을 받음
- 브라우저의 기본 스타일을 초기화하는 reset css나 normalize css 등의 스타일에서 주로 사용
  : reset & normalize css : 브라우저마다 기본 스타일을 가지고 있기 때문에 통일하기 위해 사용

```css
* {
    /*  모든 요소의 padding이 0이 됩니다.  */
    padding: 0;
}
```

### 타입 선택자

- HTML 요소 이름을 선택자로 사용
- 해당하는 모든 요소가 영향을 받음
- 요소 이름을 그대로 선택자 자리에 작성

```css
div {
    /*  모든 div 요소의 padding이 0이 됩니다.  */
    padding: 10px;
}
```

### id 선택자

- id 속성에 지정한 값을 이용해 스타일 지정
- id는 페이지당 하나만 가지는 단일값으로 정확하게 일치하는 단일 요소에만 지정하고 싶을 때 사용
- id 앞에 # 을 붙여 작성

```html
<div id="id-name">
    id가 id-name인 div 요소
</div>
<style>
    #id-name {
        padding: 10px;
    }
</style>
```

### class 선택자

- 요소의 스타일을 지정하는 가장 일반적인 방법
- class 속성은 id와 다르게 여러 요소에 같은 class 값을 지정할 수 있음
- class 명 앞에 '.'을 붙여 작성

```html
<div class="" class-name>
    class가 class-name인 div 요소
</div>
<p class="class-name">class가 class-name인 p 요소</p>
<style>
    .class-name {
        padding: 10px;
    }
</style>
```

### 속성(attribute) 선택자

- html 속성의 값들을 비교한 결과로 스타일을 지정
- 선택자는 요소 이름[속성명(연산자)값] 의 형태를 갖는다.
- 연산자에 따라 요소를 선택하는 기준을 다르게 적용

| 연산자 | 설명                                     |
|-----|----------------------------------------|
| 속성명 | 속성명만 작성할 때 해당 속성을 갖는 요소를 모두 선택         |
| =   | 값이 정확히 일치할 때 성택                        |
| ~=  | 값이 정확하게 일치할 때 + 띄어쓰기로 여러 값이 지정된 요소일 때 선택 |
| \|= | 값이 정확하게 일치할 때 + 값 뒤 -(하이픈)을 작성할 때 선택 |
| ^= | 접두사로 값을 가질 때 선택 |
| $= | 접미사로 값을 가질 때 선택 |
| *= | 값이 포함된 모든 요소 선택 |

```css
span[id] {
    color: red;
}

a[href="test.com"] {
    color: red;
}

span[class~="class1"] {
    color: red;
}

span[id|="hi"] {
    color: red;
}

span[id^="test"] {
    color: red;
}

span[id$="test"] {
    color: red;
}

span[id*="javascript"] {
    color: red;
}
```

### 여러 선택자를 동시에 지정하기

- 여러 선택자에 동일한 스타일 지정하고 싶을 때 ',' 이용하여 연결

```css
#id-name,
.class-name,
button {
    color: red;
}
```

## 우선순위와 명시도(Specificity)

- 각 스타일이 한 요소에 중첩되는 상황에서 어떤 스타일이 우선순위로 적용되는지 확인해보자.

### 마지막 작성된 스타일 적용

```css
div {
    color: blue;
    color: red;
}
```
이 경우 상단에 작성된 스타일은 상쇄 되며, 마지막 스타일이 적용됨.

```html
<link rel="stylesheet" href="./src/styles.css"/>
<link rel="stylesheet" href="./src/styles2.css"/>
```
여러 파일에 같은 선택자로 스타일이 지정된 경우, 나중에 연결된 파일의 스타일로 우선순위 적용됨.<br>
이를 피하기 위해 순서로 우선순위를 제어하기 보다, 별개의 클래스를 만들어 스타일을 완전 구분하는 것이 좋음.

### 명시도(Specificity)가 높은 선택자의 스타일 적용

- 명시도가 같은 경우 더욱 많이 작성된 선택자의 스타일이 적용
- 명시도가 다른 경우 무조건 명시도가 높은 선택자가 적용

이러한 우선순위 법칙을 무시하는 강력한 두 가지 방법이 있다.

1. 인라인 스타일 (style="")
   - id를 부여하는 방식보다 더 높은 명시도를 가짐
2. !important
   - 모든 특성을 무시하고 스타일을 적용하는 가장 강력한 방법

```html
<head>
    <style>
        div {
            color: blue !important;
        }
        
        #test-id {
            color: yellow;
            font-size: 20px;
        }
    </style>
</head>
<body>
    <!-- 이 경우 폰트의 색상은 파란색, font의 크기는 40px 입니다.-->
    <div id="test-id" style="font-size: 40px; color: red;">테스트</div>
</body>
```

**!important** 를 사용하는 것은 모든 종속성이나 규칙을 무시하고 이후에 디버깅을 어렵게 만들기 때문에 **안티 패턴으로 취급**된다.

## 박스 모델과 여백 상쇄

### 박스 모델

4가지 영역으로 이루어져 있음

![img](https://media.vlpt.us/images/gil0127/post/1b3fcbde-3863-4a5c-909a-8b58e74f73ac/%EB%B0%95%EC%8A%A4%EB%A5%BC%20%EA%B5%AC%EC%84%B1%ED%95%98%EB%8A%94%20%EC%9A%94%EC%86%8C.png)

1. content 영역
2. padding 영역
3. border 영역
4. margin 영역

**content 영역**은 텍스트나 하위 박스 모델 등을 나타내며,<br>
**padding, border, margin 영역**은 순서대로 content 영역을 둘러싸는 영역이다.

이는 CSS의 property로 너비를 지정할 수 있다.

CSS 프로퍼티 중 하나인 **box-sizing**는 요소의 너비와 높이를 계산하는 방식을 지정한다.<br>
- content-box : content 영역만을 기준으로 계산.
- border-box : padding, border, content 영역의 너비를 모두 합한 영역을 기준으로 계산.

### 여백 상쇄

인접한 같은 레벨의 블록 요소에 상하 여백이 겹치면 여백이 하나로 합쳐지는 현상을 여백 상쇄라고 한다.

이때 여백은 인접한 여백 중 큰 여백으로 상쇄된다.

여백 상쇄는 display 프로퍼티 값이 `flex, grid`일 때나 position 프로퍼티가 `absolute, float`인 상태일 때는 적용되지 않는다.


## flex를 이용한 레이아웃 만들기

모던 웹 브라우저가 등장하며 유연한 레이아웃 방식이 필요해졌고, CSS3가 등장하며 이에 맞는 flex와 grid 레이아웃 방식이 등장하였다.

현재 대부분의 모던 브라우저에서 flex와 grid 프로퍼티를 지원하기 때문에 레이아웃 배치하기 위해 이 두가지 방법을 가장 많이 사용한다.

### 기본 개념

flex는 디바이스나 디스플레이의 크기에 따라 컨테이너에 들어 있는 콘텐츠의 너비, 높이, 순서를 변경해 컨테이너의 공간을 가장 효율적으로 채우는 방법을 추구한다.

따라서 여유 공간을 채우도록 너비나 높이를 늘이거나 줄인다.

#### flex container와 flex items

flex container
: display 프로퍼티에 flex나 inline-flex를 갖는 HTML 요소

flex container는 flex-flow, flex-direction을 기준으로 배치된다.


#### 주축(main axis)과 교차 축(cross axis)

주축(main axis)
: flex items가 배치되는 기본축
: 항상 수평으로 진행되는 것은 아니며 flex container에서 flex-direction 프로퍼티로 진행 방향을 변경할 수 있음.
: 시작점을 main-start, 끝나는 점을 main-end, 이 크기를 main-size라고 함.
: 평행으로 진행한다면 main-size == flex container 너비

교차 축(cross axis)
: 주축에 수직인 방향
: cross-start, cross-end, cross-size 존재
: 주축의 방향이 수평이라면 교차 축의 방향은 수직 & corss-size == flex container 높이

[⬅️ 기초부터 완성까지, 프런트엔드 목차보기](https://tonyjev93.github.io/front%20end/기초부터%20완성까지,%20프런트엔드/front-end-basic-00-overview/)