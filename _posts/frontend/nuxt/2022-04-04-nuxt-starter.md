---
title: "[Nuxt.js] 시작하기"
last_modified_at: 2022-04-04T23:50:00+09:00
categories:
    - Front End
    - Nuxt.js
tags:
    - Front End
    - Nuxt.js
toc: true
toc_sticky: true
toc_label: "목차"
---

Nuxt.js : 인프런 장기효님의 강의와 강의노트를 기반으로 Nuxt.js에 대해 정리해보았습니다.<br>
특정 개념에 대해 찾아보는 방식으로 활용하기 위해 작성되었기에 체계없이 정리된 점 양해부탁드립니다.
{: .notice--info}

# Nuxt

- The Intuitive Vue Framework
- Vue 프레임워크를 감싼 프레임워크
- [Nuxt 소개 - Cracking Vue.js](https://joshua1988.github.io/vue-camp/nuxt/intro.html)

## Nuxt란?

- SSR 프레임워크 (서버에서 모두 작성해서 클라이언트로 보낸 뒤 화면에 그리 방식)
- 웹 애플리케이션 제작에 필요한 Vuex, Router, Axios와 같은 라이브러리를 미리 구성한 SPA, SSR, 정적 웹 사이트를 쉽게 제작할 수 있게 함.

## 왜 SSR 쓰는가?

- 검색 엔진 최적화(SEO)
- 빠른 페이지 랜더링
  - CSR의 경우 브라우저를 사용하는 디바이스 사양에 따라 랜덜이 속도가 달라짐
- OG(Open Graph) Tag 적용

## SSR 단점은?

- 학습 비용 (Node.js)
- 서버 빌드에 대한 이해
- 브라우저 관련 API 다룰 때 주의 필요

## Nuxt 장점 및 특징

### 장점

- 검색 엔진 최적화
- 초기 프로젝트 설정 비용 감소 및 생산성 향상
  - ESLint, Prettier
  - Router, Store 라이브러리 설정 파일 X
  - 파일 기반 라우팅 방식(라우터 설정 필요 X)
- 페이지 로딩 속도
- UX
- 코드 스플리팅

### 특징

- SSR
- 규격화된 폴더 구조
- pages 폴더 기반의 자동 라우팅 설정
- 코드 스플리팅

<br>

# 시작하기

- npm init nuxt-app learn-nuxt 명령어 수행 및 프로젝트 기본설정
    ```bash
    $ npm init nuxt-app learn-nuxt
    
    create-nuxt-app v4.0.0
    ✨  Generating Nuxt.js project in learn-nuxt
    ? Project name: learn-nuxt
    ? Programming language: JavaScript
    ? Package manager: Npm
    ? UI framework: None
    ? Nuxt.js modules: (Press <space> to select, <a> to toggle all, <i> to invert selection)
    ? Linting tools: ESLint, Prettier
    ? Testing framework: None
    ? Rendering mode: Universal (SSR / SSG)
    ? Deployment target: Server (Node.js hosting)
    ? Development tools: jsconfig.json (Recommended for VS Code if you're not using typescript)
    ? Continuous integration: None
    ? Version control system: Git
    ```

- 설치 완료 후 실행
    ```bash
    $ cd learn-nuxt
    $ npm run dev
    ```

- Node 버전 변경이 안되는 현상 발생 시 참고...
    ```bash
    # node version 변경이 안될 때(active 경로가 계속 유지될 때)
    $ ln -sf installed경로 active경로
    ```

<br>

# 프로젝트 구조

- .nuxt
  - 빌드 파일
- assets
  - 이미지, 리소스 등
- components
  - Vue Component
- layouts
  - router 기준 특정 url 에 접근했을 때 뜨는 페이지 컴포넌트의 최상위 루트 컴포넌트
- middleware
  - SSR 진행 시 `Server -> Browser` 파일 넘기기전에 항상 실행시키는 함수
- pages
  - 파일 기반 Routing
  - ![image](https://user-images.githubusercontent.com/53864640/161656871-ff98d7ba-c8d0-4ce2-8a04-b7febfb4609c.png)
    - folder 가 URL Path가 됨.
    - File 로 _{pathVariable}를 만들면 URL에서 ~/{pathVariable}가 자동 맵핑됨.
- plugins
  - Vue Plugin
- static
  - 정적 자원(favicon)
- store
  - Vuex의 store
  
# 개발

- layouts
  - default.vue
    - `<Nuxt /> == <router-view />`

## SSR 깜빡이는 현상 제거

### asyncData
- **넉스트 REST API 호출 방식** 
- Vue Component 속성
- 페이지 컴포넌트
  - pages 하위 폴더에서만 사용가능한 컴포넌트, pgaes 하위가 아니면 수행 안 됨.
  - 화면이 그려지기 전에 수행 됨. -> 그래서 this에 접근 불가능

```javascript
async asyncData() {
    const response = await axios.get('http://localhost:3000/products')
    console.log(response)
    const products = response.data
    return {products}
}
```

- params 사용법
```javascript
async asyncData({ params }) { // params = router 정보($route) 
  const response = await fetchProductById(params.id)
  const product = response.data
  return {product}
}
```

- v-model & props 사용

```vue
<!-- Parent-->
<template>
  <search-input v-model="searchKeyword"/>
</template>

<!-- Child-->
<template>
  <input type="text" :value="value" @input="$emit('input', $event.target.value)"/>
</template>
<script>
export default {
  model: {
    // props 변수명 다르게 하고싶을 때 사용.(기본적으로 v-model 사용 시 props 는 value 를 써야함.)
  },
  props: {
    value: {  // value 를 props로 받아야 v-model 이 가능함.
      type: String,
      default: () => '',
    },
  }
}
</script>

```

## CSS

### 글로벌 CSS 적용

- nuxt.config.js
```javascript
export default {
  ...
  // Global CSS: https://go.nuxtjs.dev/config-css
  css: [],
  ...
}
```
    
<br>


## Vuex 사용 방법

![img](https://user-images.githubusercontent.com/53864640/161774397-4275064c-121c-4bd0-8d38-bd5fb66c3af2.png){: .align-center}
_- [그림 출처](https://joshua1988.github.io/vue-camp/vuex/concept.html)_
{: .text-center}

- Vue.js 의 상태 관리 라이브러리이자 패턴을 의미([소개글](https://joshua1988.github.io/vue-camp/vuex/concept.html))
- Nuxt에서 Vuex를 기본 설정해둠.
- store 폴더 밑에 javascript 생성 시 자동으로 vuex 스토어가 자동 빌드 됨


## nuxtServerInit

- store 의 action 함수
- nuxtServerInit 라고 정의 되어있는 특수 함수
- universal 모드에서 사용할 수 있는 함수
- 서버사이드 렌더링 시점에 실행 됨
- 스토어에 미리 데이터를 설정해 놓거나 서버에서만 접근할 수 있는 데이터를 다룰 때 유용 (ex. 사용자 인증/인가 토큰)

## fetch

- 페이지 컴포넌트 뿐만 아니라 일반 뷰 컴포넌트에서도 사용할 수 있는 데이터 호출 속성
- asyncData 와 동일한 시점에 수행됨
- 용도
  - 컴포넌트가 생성되고 나서 실행됨
  - 브라우저에서 URL 주소 변경해서 페이지를 이동할 때
- 주의
  - nuxt 2.12 이전 버전에서는 fetch() 안에 파라미터를 넘길 수 있었음. (이후 deprecated 됨)

## 환경변수

- nuxt.config.js
```js
export default {
  // env config
  env: {
    baseURL: process.env.NODE_ENV === 'production'
            ? 'https://my-json-server.typicode.com/TonyJev93/nuxt-sample-api'
            : 'http://localhost:3000'
  }
}
```

<br>

# 배포

- [배포 방법](https://joshua1988.github.io/vue-camp/nuxt/deployment.html)
- nuxt.config.js
```javascript
export default {
    target: 'server' // default, target or static
}
```

## Heroku 배포

- [접속 URL](https://heroku.com)
- [가이드](https://nuxtjs.org/deployments/heroku/)
- 가입
  ![image](https://user-images.githubusercontent.com/53864640/161801439-e62716f6-d925-43fa-86f0-df91e870f4d1.png)
- 배포하기
  - Deploy > `Deploy Branch` 버튼
- log 보기 (Activity)
  ![img-log](https://user-images.githubusercontent.com/53864640/161812027-3c939d6a-53b6-44d3-be03-c894fbb4c1fa.png)

## Json Server 배포

### My JSON Server

- (접속 URL)[https://my-json-server.typicode.com/]
- 사용법
```
1. 다음과 같이 GitHub 레파지토리를 만든다. : <your-username>/<your-repo>
2. 레파지토리 안에 db.json 파일을 생성한다.
3. https://my-json-server.typicode.com/<your-username>/<your-repo> 에 접속한다.
```
- ex) https://my-json-server.typicode.com/TonyJev93/nuxt-sample-api

## SSG 배포

- Static Site Generator
- [가이드](https://joshua1988.github.io/vue-camp/nuxt/deployment.html#ssg-static-site-generator)
- 배포 방식 중 하나
- 정적 Static Resource를 서버에 배포하는 방식 (<-> Server(Node.js hosting) 배포방식)
- nuxt.config.js
    ```js
    export default {
        target: 'static'
    }
    ```
  - `npm run generate` 명령어로 수행
  - `.dist` 폴더에 빌드파일 생성됨
- Netlify(CD, Continuous Delivery)를 이용하여 손쉽게 배포 가능
  - [접속 URL](https://www.netlify.com/)
  - deploy 방법
  ![netlify](https://user-images.githubusercontent.com/53864640/161817157-b973974f-7ac0-4b8e-848c-f7ce13f54fbf.png)
  - redirect 오류 해결([참고](https://formason.tistory.com/13))
    - root 폴더에 netlify.toml 파일 생성
    ```
  [[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
  ```

## Meta Tag

- [Meta 설정관련 참고](https://joshua1988.github.io/vue-camp/nuxt/meta-tags.html)
- [페이지별 메타태그 설정 방법](https://joshua1988.github.io/vue-camp/nuxt/meta-tags.html#%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%8C%E1%85%B5-head-%E1%84%90%E1%85%A2%E1%84%80%E1%85%B3-%E1%84%89%E1%85%A5%E1%86%AF%E1%84%8C%E1%85%A5%E1%86%BC)

```vue
<script>
export default {
  head: {
    title: 'Shopping Item Detail',
    meta: [
      {
        hid: 'description', // 동일 meta hid 끼리 덮어 씌워짐
        name: 'description',
        content: '이 상품은 ~~입니다.'
      },
    ],
  },
}
</script>
```

메타태그 내에서 데이터를 연결하기 위해 아래와 같이 사용할 수 있다.

```vue
<script>
export default {
  // head 를 함수로 이용하면 데이터 연결 가능
  head() {
    return {
      title: `Shopping Item Detail - ${this.product.name}`,
      meta: [
        {
          hid: 'description',
          name: 'description',
          content: `이 상품은 ${this.product.name}입니다.`
        },
      ],
    }
  },
}
</script>
```


## OG Tag

- [OG 태그 관련 가이드](https://joshua1988.github.io/vue-camp/nuxt/meta-tags.html#open-graph-%E1%84%90%E1%85%A2%E1%84%80%E1%85%B3-%E1%84%89%E1%85%A5%E1%86%AF%E1%84%8C%E1%85%A5%E1%86%BC)
- vue 파일내에 head 설정
```vue
<script>
export default {
  // head 를 함수로 이용하면 데이터 연결 가능
  head() {
    return {
      title: `Shopping Item Detail - ${this.product.name}`,
      meta: [
        {
          hid: 'og:title',
          property: 'og:title',
          content: `상품 상세 페이지`
        },
        {
          hid: 'og:description',
          property: 'og:description',
          content: `상품 상세 정보를 확인해보세요.`
        },
        {
          hid: 'og:image',
          property: 'og:image',
          content: `${this.product.imageUrl}`
        },
      ],
    }
  },
}
</script>
```
- 디버깅 도구들
  - [페이스북 OG 태그 디버깅 도구](https://developers.facebook.com/tools/debug/)
  - [카카오 OG 태그 디버깅 도구](https://developers.kakao.com/tool/clear/og)

<br>

# Javascript Tip

```javascript
someArray.map((item) => {
  return {
    ...item, // 기존 데이터 그대로 사용
    someProperty: value, // 그 중 someProperty에 해당하는 값 덮어쓰기
  }
})
```
위 소스는 아래와 같은 소스임.(축약 버전)

```javascript
someArray.map((item) => ({
  ...item, // 기존 데이터 그대로 사용
  someProperty: value, // 그 중 someProperty에 해당하는 값 덮어쓰기
}))
```

<br>

# VSCode Short-cut

## Vue
### Vue vscode snippets Plugin
- `vpr + enter`

<br>

# 참고

- [인프런강의 - Nuxt.js 시작하기](https://www.inflearn.com/course/%EB%84%89%EC%8A%A4%ED%8A%B8-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0/dashboard)