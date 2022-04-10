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

프론트엔드 개발환경의 이해와 실습 : 인프런 강의인 "프론트엔드 개발환경의 이해와 실습 by 김정환님"을 수강하먄사 학습힌 내용들을 정리해 보았다.
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
- CDN 서버 장애로 인해 외부 라이브러리를 사용할 수 없을 때가 있음.

### 직접 다운로드 방법

- `https://unpkg.com/react@16/umd/react.development.js`에 직접 접속해서 복사하여 사용
- 그러나 업데이트된 내용들을 자동으로 동기화 할 수 없음

### NPM 이용

- 의존성 자동 업데이트 가능
- `npm install {library}`
- 유의적 버전 (Sementic Version)
  - 주 버전(Major) : 기존 버전과 호환되지 않게 변경
  - 부 버전(Minor) : 기존 버전과 호환되면서 기능이 추가된 경우
  - 수 버전(Patch) : 기존 버전과 호환되면서 버그를 수정한 경우
- 버전의 범위
  - `연산기호(=, >, < 등)`+ `버전` 사용
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

- 자바스크립트 모듈을 구현하는 대표적인 명세 : CommonJS, AMD

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
- 크롬의 경우를 살펴보면

```html
<script type="module" src="app.js"></script>
```

- 위와 같이 type="text/javascript" 대신 type="module"을 사용함
- 그러나 브라우져에 무관하게 모듈을 사용하고 싶기 때문에 **웹팩**이 등장 ...!

## 2. 엔트리/아웃풋

- 웹팩은 흩어져있는 자바스크립트의 의존 관계를 한 곳으로 뭉쳐주는 역할을 함.
- 웹팩에 의해 뭉쳐져 생성된 파일을 번들이라고 하며, 웹팩은 그런 의미에서 번들러라고도 부름.

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
    "build": "webpack --progress"
  },
  "devDependencies": {
    "webpack": "^4.46.0",
    "webpack-cli": "^4.9.2"
  }
}
```
- `type` : module 로 설정해야 함.
- `script` : webpack 으로 지정해두면 알아서 webpack 경로를 찾아서 명령어를 수행해 줌. `--progress` 는 webpack 빌드 상태를 커맨드라인에 보여주는 옵션

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

### 커스텀 로더 만들기

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

## 4. 플러그인

### 역할

- 로더는 파일 단위로 처리하는 반면, 플러그인은 번들된 결과물을 처리
- 번들된 자바스크립트를 난독화 한다거나 특정 텍스트를 추출하는 용도로 사용

### 커스텀 플러그인 만들기

```javascript
// my-webpack-plugin.js
class MyWebpackPlugin {
    apply(compiler) {
        compiler.hooks.done.tap('My Plugin', 
                stats => {console.log('MyPlugin: done');} // Plugin 완료 후 동작하는 콜백 함수
        )
    }
}

module.exports = MyWebpackPlugin;
```

```javascript
// webpack.config.js
import path from 'path';
import MyWebpackPlugin from './my-webpack-plugin';

export default {
    plugins: [
        new MyWebpackPlugin(),
    ]
}
```

- 플러그인은 어떻게 번들 결과에 접근할 수 있는지 살펴보자. 다음은 웹팩 내방 플러그인 BannerPlugin 코드를 참고한 소스이다.

```javascript
// my-webpack-plugin.js
class MyWebpackPlugin {
    apply(compiler) {
        compiler.plugin('emit', (compilation, callback) => {
            const source = compilation.assets['main.js'].source();
            compilation.assets['main.js'].source = () => {
                const banner = [
                    '/**',
                    ' * 이것은 BannerPlugin이 처리한 결과입니다.',
                    ' * Build Date: 2022-04-10',
                    ' */'
                ].join('\n');
                return banner + '\n\n' + source;
            }
            
            callback();
        })
    }
}

module.exports = MyWebpackPlugin;
```

- `compilation.assets['main.js'].source()`을 통해 번들링 된 main.js 파일에 접근할 수 있다.
- 접근한 source 를 조작하여 번들링된 결과를 변형시킬 수 있게 된다.

### 자주 사용하는 플러그인

- BannerPlugin
  - 웹팩이 기본 제공하는 플러그인
  - 결과물에 빌드 정보나 커밋 버전같은 걸 추가할 수 있음
- DefinePlugin
  - 웹팩이 기본 제공하는 플러그인
  - 어플리케이션은 개발환경과 운영환경으로 나눠서 운영함
  - 환경에 따라 API 서버 주소가 다를 수 있음
  - 같은 소스 코드를 두 환경에 배포하기 위해서는 환경 의존적인 정보를 소스가 아닌 곳에서 관리하는 것이 좋음 (배포때마다 코드를 수정하는 것은 곤란하기 때문)
  - DefinePlugin 은 이러한 환경 정보를 제공하기 위한 플러그인이다.
  - node 에서 제공하는 환경변수 사용
    - 웹팩 설정의 mode에 설정한 값 => `process.env.NODE_ENV` 에 들어감
- HtmlWebpackPlugin
  - 써드 파티 패키지
  - HTML 파일을 후처리하는데 사용
  - 빌드 타임의 값을 넣거나 코드를 압축할 수 있음
- CleanWebpackPlugin
  - 써드 파티 패키지
  - 빌드 이전 결과물을 제거하는 플러그인
  - 과거 파일이 남아있는 경우 덮어씌여지면 괜찮지만 그렇지 않을 경우 아웃풋 폴더에 계속 남아있는 것을 방지하기 위함
- MiniCssExtractPlugin
  - 써드 파티 패키지
  - CSS가 많아지면 하나의 자바스크립트 결과물로 만드는 것이 부담이 됨
  - 번들 결과에서 CSS 코드만 뽑아 CSS 파일로 만들어 역할에 따라 파일을 분리하는 것이 좋음
  - 브라우져에서 큰 파일 하나를 내려받기 보다, 여러 개의 작은 파일을 동시에 다운받는 것이 빠름
  - 개발환경에서는 CSS를 하나의 모듈로 처리해도 되지만, 프로덕션 환겅에서는 분리하는 것이 효과적
  - MiniCssExtractPlugin은 CSS를 별도의 파일로 뽑아내는 플러그인
  - MiniCssExtractPlugin 에서 제공해주는 loader 를 사용해야 함.
    - `process.env.NODE_ENV === 'production' ? MiniCssExtractPlugin.loader : 'style-loader'

```javascript
const webpack = require('webpack');
const childPorcess = require('child_process'); // terminal 명령어 수행용
const HtmlWebpackPlugin = require('html-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin'); // default로 export 되어있지 않아서 직접 꺼내와야함.
const MiniCssExtractPlugin = require('mini-css-extract-plugin');


module.exports = {
    plugins: [
        new webpack.BannerPlugin({
          banner: `
            Build Date: ${new Date().toLocaleString()}
            Commit Version: ${childPorcess.execSync('git rev-parse --short HEAD')}
            Author: ${childPorcess.execSync('git config user name')}
          `
        }),
        new webpack.DefinePlugin({
          TWO: '1+1', // 애플리케이션에 TWO 라는 전역 변수가 생성이 됨. ex) console.log(TWO); // 2
          TWO_STRING: JSON.stringify('1+1'), // ex) 문자열 선언, console.log(TWO_STRING); // "1+1"
          'api.domain': JSON.stringify('http://dev.api.domain.com') // 객체 선언, ex) console.log(api.domain); // "http://dev.api.domain.com"
        }),
        new HtmlWebpackPlugin({
          template: './src/index.html',  // build 과정에 html 을 포함시켜 main.js 와 같은 번들된 javascript 를 index.html 에 자동으로 넣어줌.
          templateParameters: {
              env: process.env.NODE_ENV === 'development' ? '(개발용)' : '' // index.html 에서 'EJS 문법'인 <%= env %> 를 통해 ENV에 접근할 수 있음.
          },
          minify: process.env.NODE_ENV === 'production' ? {
              collapseWhitespace: true, // 공백 제거
              removeComments: true, // 주석 제거
          } : false // 운영환경에서만 활성화
        }),
        new CleanWebpackPlugin(),    // dist 폴더를 비워주는 역할
        ...(process.env.NODE_ENV === 'production'
            ? [new MiniCssExtractPlugin({filename: '[name].css'})]
            : []
        ),  // 운영환경인 경우에만 수행, 개발환경에서는 하나의 javascript를 만드는 것이 더욱 빌드속도가 빠름.
    ]
}
```

<br>

# 바벨(Babel)

## 1. 배경

- 고대 히브리 신화의 이야기를 살펴보자.

사람들이 하늘 높이 올라가기 위해 바벨탑을 쌓기 시작하였다. 그러나 하느님이 그들이 사용하는 언어를 서로 다르게 바꾸어 서로 소통할 수 없게 만들어 탑을 쌓는 것을 실패하게 만들었다.
{: .notice--info}

- 사용하는 말이 달라 바벨탑 쌓기에 실패했듯이, 브라우져마다 사용하는 언어가 조금씩 달라 프론트엔드 코드는 일관적이지 못할 때가 많음.
- FE 개발에서 크로스브라우징 이슈는 코드의 일관성을 해치고 초심자를 불안하게 만듦. (히브리어로 바벨이 '혼돈'이라는 뜻)
- 크로스브라우징의 혼란을 해결해 줄 수 있는 것이 **바벨**이다.
- ES2015로 작성한 코드를 모든 브라우져에서 동작하도록 호환성을 지켜줌.(Typescript, JSX 등 다른 언어로 분류된 것도 포함)
- 
## 2. 바벨 기본동작

- 설치 방법

```bash
# babel 설치
$ npm install @babel/core @babel/cli 

# babel 실행
$ npx babel app.js
```

- 빌드 과정
  1. 파싱(Parsing) - 토큰 분해
  2. 변환(Transforming) - ES6 -> ES5 변환
  3. 출력(Printing) - 변환 결과 출력

## 3. 플러그인

- 바벨은 파싱과 출력만 담당하고 변환 작업은 **플러그인**이 처리한다.

### 커스텀 플러그인

```javascript
// src/app.js:
const alert = msg => window.alert(msg);

// my-bebel-plugin.js
module.exports = function myBabelPlugin() {
    return {
        visitor: {
            Identifier(path) {
                const name = path.node.name;
                
                // 바벨이 만든 AST 노드를 출력
                console.log('Identifier() name:', name)
                
                // 변환작업: 코드 문자열을 역순으로 변환
                path.node.name = name
                        .split("")
                        .reserve()
                        .join("");
            }
        }
    }
}
```

```bash
# babel 실행
$ npx babel app.js --plugins './my-babel-plugin.js'
```

```bash
// 출력 결과
Identifier() name: alert
Identifier() name: msg
Identifier() name: window
Identifier() name: alert
Identifier() name: msg
const trela = gsm = > wodniw.trela(gsm); // 역순으로 정리 된 결과를 볼 수 있음.
```

- const => var 변환

```javascript
// my-bebel-plugin.js
module.exports = function myBabelPlugin() {
    return {
        visitor: {
            VariableDeclaration(path) {
                console.log('VariableDeclaration() kind:', path.node.kind); // const
                
                // const => bar 변환
                if (path.node.kind === 'const') {
                    path.node.kind = 'var'
                }
            }
        }
    }
}
```

```bash
// 출력 결과
VariableDeclaration() kind: const
var alert = msg = > wodniw.alert(msg); // const => var
```

### 플러그인 사용하기

- block-scoping
  - const, let 처럼 블록 스코핑을 따르는 예약어를 함수 스코핑을 사용하는 var 로 변경
- arrow-functions
  - IE 는 화살표 함수를 지원하지 않는데 arrow-functions 플러그인을 이용해서 일반 함수로 변경 가능
- strict-mode
  - ES5에서 지원하는 엄격 모드를 사용하는 것이 안전하기 때문에 "use strict" 구문을 추가해야 함
  - strict-mode 플러그인이 이를 제공 해줌

### 컨피그 파일 사용

- `babel.config.js` 파일 사용

```javascript
// babel.config.js
module.exports = {
    plugins: [
        "@babel/plugin-transform-block-scoping",
        "@babel/plugin-transform-arrow-functions",
        "@babel/plugin-transform-strict-mode",
    ]
}
```

## 4. 프리셋 사용

#### 커스텀 프리셋

```javascript
// my-babel-preset.js
module.exports = function myBabelPreset() {
    return {
        plugins: [
            "@babel/plugin-transform-block-scoping",
            "@babel/plugin-transform-arrow-functions",
            "@babel/plugin-transform-strict-mode",
        ],
    }
}
```

```javascript
// babel.config.js
module.exports = {
    presets: [
        './my-babel-preset.js'
    ]
}
```

### 주요 프리셋

- preset-env
  - 바벨 7 버전 이후 연도별 각 프리셋(babel-reset-es2015 ~ latest)이 env 하나로 합쳐짐
- preset-flow
- preset-react
- preset-typescript

```javascript
// babel.config.js
module.exports = {
    presets: [
        '@babel/preset-env', {
        targets: { // 특정 타겟에 맞추어 변환
            chrome: '79',
            ie: '11',
        }
      }
    ]
}
```

- 브라우져별 문법 사용 가능여부는 [Can I use](https://caniuse.com/) 에서 확인 가능
- Promise 와 같이 IE 11 에서 지원하지 않는 함수는 babel 을 통해서 변환하더라도 찾을 수 없기 때문에 프로그램이 죽게 된다.
- 이렇게 변환 할 수 없는 것들에 대해서는 **"폴리필"** 이라고 부르는 코드조각을 추가해서 해결한다.

```javascript
// babel.config.js
module.exports = {
    presets: [
        '@babel/preset-env', {
        targets: { // 특정 타겟에 맞추어 변환
            chrome: '79',
            ie: '11',
        },
        // 폴리필 설정
        useBuiltIns: 'usage',   // 'entry', false
        corejs: {
            version: 2,
        }
      }]
}
```

## 5. 웹팩으로 통합

- 실무 환경에서는 바벨을 직접 사용하기 보다 웹팩으로 통합하여 사용
- `babel-loader`와 같이 바벨이 로더 형태로 제공 됨

```javascript
// webpack.config.js
export default {
    module: {
        rules: [
            {
                test: /\.js$/,  // 로더가 처리해야 하는 파일의 패턴(=정규 표현식)
                loader: 'babel-loader',
                exclude: /node_modules/ // node_modules은 로더 대상 제외
            }
        ]
    }
}
```

# 참고

- [프론트엔드 개발환경의 이해와 실습 (webpack, babel, eslint..)](https://www.inflearn.com/course/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD/dashboard)