---
title: "[Front End] 기초부터 완성까지 - 02장. HTML과 CSS (1)"
last_modified_at: 2022-03-29T03:00:00+09:00
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

기초부터 완성까지, 프런트엔드 개발 도서 관련 : 02장 HTML과 CSS에 대해 정리한다. (1)에서는 HTML에 대해서만 다루고 (2)에서 CSS에 대한 글을 작성하겠다.
{: .notice--info}

# HTML과 CSS

집 짓기에 비유해보자면,<br>
**HTML**은 기둥, **CSS**는 꾸미는 역할, **자바스크립트**는 여러 시설을 동작하게 하는 것.

# HTML
: Hyper Text Markup Language

- HTML 표준은 W3C, WHATWG가 주도적으로 작성한다.
  - WHATWG
    : 애플, 구글, 모질라, 마이크로소프트 등 각 브라우저 개발 벤더사들이 모여 웹 표준을 만드는 조직)
  - 과거와 달리 살아 있는 표준 형태로 발전중.
    : 웹 개발자, 벤더사, 이해 당사자 등 피드백을 받으며 지속적으로 업데이트되는 표준

## HTML 요소 구성 살펴보기

HTML은 느슨한 문법을 가지고 있다. 그래서 구성에 맞지 않게 요소를 작성해도 에러 없이 제대로 렌더링 된다.<br>
그렇다 보니 표준을 준수하지 않고 작성하다 보면 예상치 못한 문제가 발생하기도 한다. 주의하자.

```html
<h1>Hello world</h1>
```

요소는 크게 3 가지로 나뉜다.

**Element(요소)**
1. Content(내용)
2. Start Tag(시작 태그)
3. End Tag(종료 태그)

### Void 요소

- Start Tag 또는 Self-closing 형태로 돼있는 경우
  : Start Tag 내에 / 를 넣는 경우(ex. `<input/>`) Self-closing 이라고 함

### Attributes(속성)

태그의 동작을 제어하는 역할.



## 콘텐츠 모델(Contents Model)
![img](https://www.w3.org/TR/2011/WD-html5-20110525/content-venn.png){: .align-center}

HTML5 에서는 요소가 어떤 콘텐츠를 표현할 수 있는지, 어떤 하위 요소를 가지는지에 따라 Contents Model 로 분류한다.

### Metadata Content

나머지 콘텐츠의 표시나 동작을 설정하거나 문서와 다른 문서와의 관계를 설정하는 요소
```html
<base>, <link>, <meta>, <nonscript>, <script>, <style>, <template>, <title> 
```


### Flow Content

본문에 사용되는 대부분의 요소

### Sectioning Content

아웃라인을 정의하며 각 Heading 요소와 Footer 요소의 범위를 정함
- 아웃라인
  : 아웃라인은 HTML5에서 등장한 개념으로 웹 사이트의 내용을 명확하게 구분하도록 추가된 문서의 구조를 정의하는 알고리즘

```html
<article>, <aside>, <nav>, <section> 
```

### Phrasing Content

문서 내의 텍스트를 의미하는 단락을 형성하는 요소

### Heading Content

섹션의 헤더를 정의하는 요소

```html
<h1> ~ <h6> 
```

### Embedded Content

외부의 자원을 가져오거나 삽입할 때 사용하는 요소

```html
<audio>, <canvas>, <embed>, <iframe>, <img>, <mth>, <svg>, <video> 
```

### Interactive Content

유저와 상호작용을 위해 특별히 설계된 요소

```html
<audio>, <canvas>, <embed>, <iframe>, <img>, <mth>, <svg>, <video> 
```

## HTML 문서 골격 만들기

골격 작성은 문서의 **성격**과 **정보**를 정확하게 나타내며 **정상적인 페이지 노출에 중요한 역할**을 한다.

### DOCTYPE

문서의 타입을 지정함.

DOCTYPE을 html로 하면 HTML5로 작성된 문서임을 나타냄.<br>
(모던 브라우저에서는 명시하지 않아도 자동인식하지만, IE9 이전 버전의 브라우저는 다른 형태의 문서로 인식함...)

```html
<!DOCTYPE html>
```

### html

`<html>` 태그는 문서의 루트 지점을 명시하는 태그이다.

lang 속성을 해당 서비스 국가에 맞게 넣는 것이 중요.
: 다국어 지원 서비스라면 언어가 바뀔때마다 해당 속성 또한 함께 변경해주어야 한다.

```html
<!DOCTYPE html>
<html lang ="ko">
</html>
```

### head

`<head>`태그는 문서의 제목과 문서의 인코딩 형식 등을 지정한다.

- `<title>` : 문서의 이름(제목)을 나타냄. 문서당 한 개의 제목 요소가 있어야 하며 북마크나 검색 엔진에서 식별 기준이 됨.
- `<meta>` : 메타 데이터를 다룸. 메타 데이터란 `데이터에 대한 데이터, 어떤 목적으로 만들어진 데이터`를 의미하며, 주로 기계가 읽고 이해하는 정보이다.
  - charset : 문서의 인코딩 방식 지정.(안하면 문자 깨짐 현상 발생)

```html
<!DOCTYPE html>
<html lang="ko">
    <head>
        <title>문서의 제목입니다.</title>
        <meta charset="UTF-8"/>
    </head>
</html>
```

### body

문서의 내용이 들어가며 일반적으로 `<head>` 태그 다음에 작성 됨.

```html
<!DOCTYPE html>
<html lang="ko">
    <head>
        <title>문서의 제목입니다.</title>
        <meta charset="UTF-8"/>
    </head>
    <body>
    <!-- 문서 내용 -->
    </body>
</html>
```

## 시맨틱(Semantic)

HTML 요소를 어떻게 작서하는 것이 중요할까???

여러가지 있겠지만 제일 중요한 것은 시멘틱하게 작성하는 것이다.

### 시멘틱이란?

Semantic
: 의미의, 의미론적인

즉, 의미에 맞는 태그를 사용해 문서를 작성하는 것을 말한다.

시멘틱하게 HTML을 작성하면 **요소에 쉽게 의미를 부여**할 수 있다.<br>
**개발자 입장**에서는 **가독성이 증가**하고, **유지보수가 쉬워**진다. 또한 장치에서 콘텐츠의 **계층 구조를 더 쉽게 파악**할 수 있다.

**검색 엔진**은 HTML 계층 구조에 따라 키워드들의 중요도를 파악하기 때문에 **크롤러에 더 구체화된 정보를 제공**할 수 있게 된다.

### <h1> ~ <h6>

각 섹션의 제목을 나타냄.

### <header>

제목이나 대표 이미지가 들어가는 요소.
`<body>`의 하위로 작성될 경우 웹 페이지의 전체 헤더를 정의하는 영역이며,<br>
`<article> or <section>` 등 Sectioning Content 하위로 사용되면 해당 영역의 헤더를 의미

`<h1>, <h2>` ... 등의 요소나 로고 등을 포함.

### <footer>

전체 문서 또는 Sectioning Content 의 바닥글로 사용됨.

작성자나 관련 문서 링크, 라이센스, 색인 등의 데이터가 들어감.

### <main>

페이지의 콘텐츠 영역을 의미

페이지당 한 번 사용하며 `<body>` 아래 직접 추가

다른 요소 내에 중첩되면 안된다.


### <article>

요소 자체가 하나의 의미 있는 콘텐츠 블록 영역

요소만으로 단일 게시물을 나타낼 때 사용. 독립적으로 배포하거나 재사용됨.

블로그 항목이나 게시물, 기사, 위젯 등에 사용됨

### <section>

`<article>`과 유사하지만 페이지의 단일 부분을 그룹화하는 데에 유용한 요소

기사의 헤드라인을 모으거나 각 블로그의 피드 정보가 나타나는 영역으로 사용됨.

요소의 콘텐츠를 함께 묶는 것이 합리적일 때 `<article>` 대신 `<section>` 요소를 사용하는 것이 더 좋음.

### <aside>

기본 콘텐츠와는 직접 관련이 없지만 간접적으로 관련된 추가 정보를 포함하는 요소

`<nav>` 요소나 광고, 인용처럼 분리된 콘텐츠를 나타낼 때 사용

### <nav>

다른 페이지 또는 내 문서의 특정 영역으로 이동시키는 링크를 나타냄.

## SEO(Search Engine Optimization)

우리의 사이트를 쉽게 찾도록 개선하는 여러 노력을 **검색 엔진 최적화**(SEO)라고 한다.

요즘 사이트 개발에서 **필수로 적용하는 방법**을 살펴보자.

### 시맨틱하게 작성

검색 엔진과 크롤러는 각 영역이 어떤 의미의 정보를 담는지 기계적으로 읽을 수 있다.

시맨틱하게 작성된 HTML은 검색 시 사이트가 노출되는 것에 도움을 준다.

### `<title>` 필수적으로 작성

`<title>`에 작성된 제목은 검색될 때 그 사이트의 이름이다.

자주 바꾸거나 길이가 너무 길어지면 검색 엔진에서 불이익을 받을 수 있다.

### `<meta name="description">`이용

`<meta name="description">`을 통해 페이지에 대한 간단한 설명을 작성해야 한다.

```html
<meta name="description" content="이 페이지에 대한 두세 문장 정도의 간단한 설명">
```

형태로 작성하며, 사용자의 흥미를 유발하여 검색이 되었을 때 완전히 노출되도록 적는 것이 좋다.

### `<meta charset="UTF-8"/>` 인코딩 방식 지정

`<meta>` 태그의 charset 속성으로 인코딩 방식을 지정하는 것이 좋다.

여러 브라우저에서 통일된 인코딩 방식으로 노출시키게 된다.

### open graph, twitter 태그 사용

open graph, twitter 태그를 통해 외부 사용자를 유인하자.

og 태그
: 페이스북에서 만든 프로토콜
: 여러 상황에서 동일한 메타 정보를 쉽게 표시하도록 만들어졌음

카카오나 네이버 또한 링크 공유 시 og 태그에 작성된 정보로 노출되며 트위터는 자체 twitter 프로토콜을 가진다.

```html
<meta property="og:title" content="페이지 이름"/>
<meta property="og:description" content="페이지에 대한 간략한 한두 줄 설명"/>
```

**og:** 가 붙으면 프로퍼티에 값을 지정한 뒤 content 프로퍼티에 값을 주어 작성한다.

요즘은 링크 공유를 통한 유입이 중요하기 때문에 og 태그를 작성하는 것은 필수다.

<br>

# 고찰

HTML에 대한 기본적인 지식을 공부해보았다. 처음에는 알고있는것은 제외하고 몰랐던 것들을 정리하려고 했는데, 생각보다 아에 밑바닥 지식 Base가 없이 개발을 시작해서 그런지 모든것이 새로운 지식이었다.
<br>
<br>
그동안 개발 하면서 의미도 모른체 사용하고 있던 것들 또는 놓치고 있던 것들에 대한 기본지식을 쌓을 수 있었고, FE 개발자라면 당연히 알아야 하는 것들이라고 생각이 든다.<br>
(물론, 난 BE 개발자다.)
{: .notice--primary}

<div style="
  display: flex;
  justify-content: space-between;
">
<a href="/front%20end/기초부터%20완성까지,%20프런트엔드/front-end-basic-00-overview/">⬅️ 기초부터 완성까지, 프런트엔드 목차보기</a>
<a href="/front%20end/기초부터%20완성까지,%20프런트엔드/front-end-basic-02-2/">2장. HTML과 CSS (2) 보러가기 ➡️️</a>
</div>