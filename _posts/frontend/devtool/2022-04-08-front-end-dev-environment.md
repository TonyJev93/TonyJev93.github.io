---
title: "[Front End] 프론트엔드 개발환경의 이해와 실습(webpack, babel, eslint ..)"
last_modified_at: 2022-04-09T23:00:00+09:00
categories:
    - Front End
    - 프론트엔드 개발환경의 이해와 실습
tags:
    - Front End
    - 프론트엔드 개발환경의 이해와 실습
    - webpack
    - babel
    - eslint
toc: true
toc_sticky: true
toc_label: "목차"
---

프론트엔드 개발환경의 이해와 실습 : 인프런 강의인 "프론트엔드 개발환경의 이해와 실습 - 김정환님"를 수강하며 학습내용을 기록한 내용들
{: .notice--info}

# NPM

## 프론트엔드 Node.js 가 필요한 이유

- 최신 스펙으로 개발할 수 있다.
  - 브라우져에 비해 자바스크립트 스펙의 발전이 더욱 빠름. 이러한 차이를 매꾸어주는 징검다리 역할을 하는 babel, weback, NPM 은 Node.js 위에서 돌아가는 도구들
  - Typescript, SASS 같은 고수준 언어를 사용하기 위해 트랜스파일러가 필요한데 이것 또한 Node.js 환경
- 빌드 자동화
  - 과거에는 코딩 결과물을 바로 브라우져에 올리지 않고, 파일 압축, 난독화, 폴리필을 추가하는 등 개발 이외 작업을 거친 후 배포함.
  - 이러한 빌드 과정을 이해하는데 Node.js가 필요함.
  - 라이브러리 의존성, 각종 테스트 자동화하는데도 사용됨.
- 개발환경 커스터마이징
  - 프레임워크에서 제공하는 도구(CRA, vue-cli)를 개발환경에 따라 사용할 수 없는 경우가 있음.
  - 이런 경우 커스터마이징 하여 사용하기위해 Node.js 지식이 필요
  
## 패키지 설치

### CDN

- CDN(컨텐츠 전송 네트워크)으로 제공하는 라이브러리를 직접 가져 오는 방식
```html
<script src="https://unpkg.com/react@16/umd/react.development.js"></script>
```
- CDN 서버 장애로 이냏 외부 라이브러리를 사용할 수 없을 때가 있음.

### 직접 다운로드 방법

- `https://unpkg.com/react@16/umd/react.development.js`에 직접 접속해서 복사하여 사용
- 그러나 업데이트된 내용들을 자동으로 동기화 할 수 없음

### NPM 이용

- 의존성 자동 업데이트 가능
- `npm install {library}`
- 유의적 버전 (Sementic Version)
  - 주 버전(Major) : 기존 버전과 호환되지 않게 변경
  - 부 버전(Minor) : 기존 버전과 호환되면서 기능이 추가된 경우
  - 수 버전(Patch) : 기존 버전과 홀환되면서 버그를 수정한 경우
- 버전의 범위
  - `=, >, < 등의 연산기호 + 버전` 사용
  - ~ (틸드) : 마이너 버전이 명시되어 있으면 패치버전을 변경. 마이너 버전이 없으면 마이너 버전을 갱신. 
    - ex)
      - 마이너 버전 있는 경우 : ~1.2.3 표기는 1.2.3 부터 1.3.0 미만 까지 포함
      - 마이너 버전 없느 경우 : ~0 표기는 0.0.0 ~ 1.0.0 미만 까지 포함
  - ^ (캐럿) : 정식버전에서 마이너와 패치 버전을 변경. 0.x 버전은 패치만 갱신
    - ex)
      - ^1.2.3 표기는 1.2.3 부터 2.0.0 미만 까지 포함
      - ^0 표기는 0.0.0 부터 0.1.0 미만 까지 포함
  - 요즘은 ^(캐럿)을 많이 사용함.
    - 라이브러리 정식 릴리즈 전에 버전이 변경되는 경우가 빈번해 짐.
    - 0.1 -> 0.2 로 변경되어도 하위 호환성을 안지키는 경우가 많아서 ^을 통해 이를 지킴.

<br>

# 웹팩

## 1. 배경

- 자바스크립트에서 문법수준에서의 모듈 지원은 ES2015 부터 시작 됨.
- import/export 구문이 없던 모듈 이전 상황을 살펴보는 것이 웹팩 등장 배경을 설명하기 좋음.

```javascript
// math.js
function sum(a, b) {
    return a + b;
}

// app.js
console.log(sum(1, 2));
```

```html
<!DOCTYPE html>
<html lang="en">
<body>
    <script src="src/math.js"></script>
    <script src="src/app.js"></script>
</body>
</html>
```
- 이 경우, 전역으로 함수가 포함되기 때문에 동일한 이름의 함수를 선언하게 되면 덮어씌워지게 됨. 이를 해결하기 위해 IIFE 를 사용하게 됨.
- IIFE(즉시 실행 함수 표현) : 정의하자마자 즉시 실행되는 Javascript Function 을 말함.

```javascript
(function () {
    statements
})();
```

- 위 math 함수를 IIFE로 표현하면 다음과 같이 사용된다.

```javascript
// math.js
var math = math || {};

(function() {
    function sum(a, b) {
        return a + b;
    }

    math.sum = sum;
    
})()

// app.js
console.log(math.sum(1, 2));
```

- 이렇게 하면 math 함수 내에 sum 함수는 선언된 블록 안에서만 할당 되고 사라지게 되고, math.sum 을 통해 접근 가능하게 된다. 

### 다양한 모듈 스펙

- 자바스크립트 모듈을 구현하는 대표적인 명세 : AMD, CommonJS

#### CommonJS

- 자바스크립트를 사용하는 모든 환경에서 모듈을 하는 것이 목표
- export 키워드로 모듈을 생성
- require() 함수로 불러 들이는 방식
- Nodejs에서 이를 사용함.

```javascript
// math.js
export function sum(a, b) { return a + b; }

// app.js
const sum = require('./math.js');
sum(1, 2);
```

#### AMD

- Asynchronous Module Definition
- 비동기로 로딩되는 환경에서 모듈을 사용하는 것이 목표 (주로 브라우져 환경)

#### UMD

- Universal Module Definition
- AMD 기반으로 CommonJS 방식까지 지원하는 통합 형태

#### ES2015 표준

- 각 커뮤니티마다 각자의 스펙을 제안하다가 ES2015 표준 모듈 시스템을 내놓음
- 지금은 바벨과 웹팩을 이용해 모듈 시스템을 사용하는 것이 일반적

```javascript
// math.js
export function sum(a, b) { return a + b; }

// app.js
import * as math from './math.js';
math.sum(1, 2);
```

### 브라우져의 모듈 지원

- 모든 브라우져에서 모듈 시스템을 지원하지는 않음
- IE 포함 몇 브라우져는 여전히 모듈 사용하지 못함
- 크롬의 경우를 살펴보면,

```html
<script type="module" src="app.js"></script>
```

- 위와 같이 type="text/javascript" 대신 type="module"을 사용함
- 그러나 브라우져에 무관하게 모듈을 사용하고 싶기 때문에 **웹팩**이 등장 ...!

## 2. 엔트리/아웃풋

- 웹팩은 흩어져있는 자바스크립트의 의존 관계를 한 곳으로 뭉쳐주는 역할을 함.
- 웹팩에 의해 생성된 뭉쳐진 파일을 번들이라고 하며 웹팩은 그런 의미에서 번들러라고도 부름.

### 웹팩 명령어

- `--mode` : develop, production
- `--entry` : 모듈의 시작점
- `--output-path, -o` : 엔트리를 통해 모든 모듈을 하나로 합친 결과물의 경로
- Ex) webpack 실행

```bash
$ node_modules/.bin/webpack --mode development --entry ./src/app.js -o ./dist
```

- `--config` : 웹팩 컨피그 파일을 통해 웹팩 가능
- ex) ./webpack.config.js.

```javascript
// ES6의 모듈 시스템이 아닌 Node의 모듈 시스템
import path from 'path';

export default {
    mode: 'development',
    entry: {
        main: './src/app.js'
    },
    output: {
        path: path.resolve('./dist'),
        filename: '[name].js' // 이렇게 하면 entry 에 있는 이름들을 동적으로 할당 할 수 있음. (ex. main.js, main2.js ...)
    }
}
```

- package.json

```json
{
  "type": "module",
  "scripts": {
    "build": "webpack"
  },
  "devDependencies": {
    "webpack": "^4.46.0",
    "webpack-cli": "^4.9.2"
  }
}
```
- `type` : module 로 설정해야 함.
- `script` : webpack 으로 지정해두면 알아서 webpack 경로를 찾아서 명령어를 수행해 줌.

- 실습 코드 : [lecture-frontend-dev-env](https://github.com/jeonghwan-kim/lecture-frontend-dev-env.git) by jeonghwan-kim
```bash
$ git clone https://github.com/jeonghwan-kim/lecture-frontend-dev-env.git
```

## 3. 로더

### 역할

- 웹팩은 모든 파일을 모듈로 바라봄
- 자바스크립트로 만든 모듈 뿐만 아닌 css, 이미지, 폰트 글꼴 까지도 전부 모듈로 봄
- 따라서, ES6의 import 구문을 사용하면 자바스크립트 안으로 가져올 수 있음.
- 이를 가능하게 하는 것은 웹팩의 로더 덕분임.
  - 모든 파일을 자바스크립트의 모듈 파일처럼 만들어줌(ex. css -> javascript, image -> data URL 형식의 문자열, typescript -> javascript)

### 커스텀 로더 설정

```javascript
// my-webpack-loader.js
module.exports = function myWebpackLoader(content) {
    console.log('myWebpackLoader가 동작함');
    return content.replace('console.log(', 'alert(');
}
```

```javascript
// webpack.config.js
import path from 'path';

export default {
    module: {
        rules: [
            {
                test: /\.js$/,  // 로더가 처리해야 하는 파일의 패턴(=정규 표현식)
                use: [
                    path.resolve('./my-webpack-loader.js')
                ]
            }
        ]
    }
}
```

- `test` 의 표현식에 해당하는 파일마다 Loader에서 지정한 처리들을 수행하게 됨.

### 자주 사용되는 로더

- css-loader
  - css 파일을 js로 변환해주는 역할
  - css 파일이 js 파일의 모듈로 변경되어 js에서 이를 import하여 사용 할 수 있게 됨
- style-loader
  - js로 변경된 css 파일을 Html 파일에 넣어주는 로더.
  - js로 변경된 css 만으로는 화면에 반영이 안 됨. 이를 html에 넣어주는 것 까지 해야 함.
- file-loader
  - image 파일을 읽을 수 있도록 처리해주는 로더.
- url-loader
  - 사용하는 이미지 갯수가 많아지면 네트워크 리소스 사용 부담이 생겨 성능에 영향을 줌
  - 작은 이미지 여러개를 사용한다면 Data URI Scheme를 사용하는 방법이 더 낫다.
    - Data URI Scheme : 이미지를 Base64로 인코딩 하여 문자열로 소스코드에 넣는 형식
    - 네트워크 통신 없이 인코딩된 데이터를 토대로 바로 이미지 접근 가능
    - ex) `<img src="data:image/png;base64, iVBO.....==" alt=Red dot"/>`
  - url-loader 는 이러한 처리를 자동으로 수행

```javascript
// webpack.config.js
export default {
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /\.(png|jpg|gif|svg)$/,
                loader: 'file-loader',
                options: {
                    publicPath: './dist/', // 파일로더가 처리한 후 생성된 파일의 앞에 붙는 경로
                    name: '[name].[ext]?[hash]' // name: output file name, ext: 확장자, hash: 케싱 무력화
                }
            },
              // file-loader 대신 아래 추가 가능
            {
              test: /\.(png|jpg|gif|svg)$/,
              loader: 'url-loader',
              options: {
                publicPath: './dist/', // 파일로더가 처리한 후 생성된 파일의 앞에 붙는 경로
                name: '[name].[ext]?[hash]', // name: output file name, ext: 확장자, hash: 케싱 무력화
                limit: 2000, // 2kb, 2kb 미만의 파일은 url-loader에 의해 base64로 문자열 변환 처리됨. 그 이상은 file-loader가 파일 복사를 수행함.
              }
            }
        ]
    }
}
```
- 로더 실행 순서는 use 배열의 뒤에서부터 앞으로 수행 됨.
- 정적 파일은 로딩되면서 생성되는 이름이 해쉬값으로 되어있음. 이는 브라우져가 동일 이름의 파일에 대해서는 케싱하기 때문에 내용이 변경되었을 때 실시간 반영하지 못함. 이를 방지하기 위해 매번 파일명이 다르도록 생성함.



<br>



# 참고

- [프론트엔드 개발환경의 이해와 실습 (webpack, babel, eslint..)](https://www.inflearn.com/course/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD/dashboard)