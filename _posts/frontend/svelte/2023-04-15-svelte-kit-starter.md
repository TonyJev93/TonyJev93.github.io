---
title: "[Svelte] SvelteKit 시작하기"
last_modified_at: 2023-04-16T22:00:00+09:00
categories:
    - Front End
    - Svelte
tags:
    - Front End
    - Svelte
    - SvelteKit
toc: true
toc_sticky: true
toc_label: "목차"
---

Svelte : SvelteKit 프로젝트를 생성해보고 빌드와 배포까지 어떻게 진행되는지 확인해보자. 
{: .notice--info}

# 작성 목적

SvelteKit 을 사용하여 웹 사이트를 제작한다.<br/>
그 과정에서 SvelteKit의 특징을 확인한다.<br/>
SSR, CSR, SSG 에 대한 빌드 및 배포의 차이를 알아본다.<br/>
각각의 방식에 대해 장/단점을 파악해본다.<br/>
운영을 하는데 있어 고려해야할 문제를 알아본다.
{: .notice--success}

# 시작 하기

- 프로젝트 생성

```
npm create svelte@latest my-app
cd my-app
npm install
npm run dev
```

<br/>

# 프로젝트 구조

```
my-project/
├ src/
│ ├ lib/
│ │ ├ server/
│ │ │ └ [your server-only lib files]
│ │ └ [your lib files]
│ ├ params/
│ │ └ [your param matchers]
│ ├ routes/
│ │ └ [your routes]
│ ├ app.html
│ ├ error.html
│ ├ hooks.client.js
│ └ hooks.server.js
├ static/
│ └ [your static assets]
├ tests/
│ └ [your tests]
├ package.json
├ svelte.config.js
├ tsconfig.json
└ vite.config.js
```

Svelte 프로젝트의 주요 부분은 src 디렉토리에 저장되며, src/routes와 src/app.html을 제외한 모든 것은 선택 사항이다.

- `lib`
  - 유틸리티와 컴포넌트를 위한 라이브러리 코드가 포함
  - `$lib` 별칭을 사용하여 호출
  - svelte-package를 사용하여 패키지화 가능 
- `server`
  - 서버 전용 라이브러리 코드 포함
  - `$lib/server` 별칭 사용하여 호출
  - 클라이언트 코드에서 가져오지 못하도록 막혀있음
- `params`
  - 앱에서 필요한 매개 변수 매처 포함
- `routes`
  - 앱의 경로 포함
  - 단일 경로에서만 사용되는 다른 구성 요소를 함께 배치할 수 있음
- `app.html`
  - 페이지 템플릿
- `error.html`
  - 예외 발생 시 렌더링되는 기본 에러 페이지
- `hooks.client.js`
  - 클라이언트 훅
- `hooks.server.js`
  - 서버 훅
- `static`
  robots.txt나 favicon.png와 같은 정적인 파일 관리
- `tests`
  - Playwright를 사용하여 브라우저 테스트를 추가한 경우에만 존재
  - 테스트 파일 관리
- `package.json`
  - devDependencies로 `@sveltejs/kit, svelte, vite` 포함
  - `"type": "module"`이 포함되어 있으면 `.js` 파일은 import/export 키워드를 사용한 네이티브 자바스크립트 모듈로 해석 됨
- `svelte.config.js`
  - Svelte와 SvelteKit의 설정 관리
- `tsconfig.json`
  - TypeScript 설정 관리
  - SvelteKit은 특정한 설정이 반드시 정해져야 하기 때문에 .svelte-kit/tsconfig.json 파일을 자동으로 생성하며, 이 파일에 사용자가 설정한 파일을 확장
- `vite.config.js`
  - Vite 설정 관리

<br/>

# Routing

## +page

### +page.svelte

- 앱의 페이지 정의
- 초기 세팅을 위해 SSR에서 렌더링되며 이후 변경을 위해 CSR에서도 렌더링 됨
- 서버에서 페이지를 사전 렌더링하여 페이지 속도 및 SEO 개선 가능

> 라우트 간 이동을 위해 \<Link> 대신 \<a> 요소 사용
> - HTML 표준에 맞게 작성 가능하도록 함
> - 브라우저 기본 동작을 최대한 활용하여 앱의 퍼포먼스 개선(캐싱 기능을 통한 빠른 페이지 이동)

### +page.js

- 페이지 렌더링되기 전에 데이터 로드하기 위해 사용
- `load` 함수 exports

```javascript
// src/routes/blog/[slug]/+page.js
import { error } from '@sveltejs/kit';
 
export function load({ params }) {
  if (params.slug === 'hello-world') {
    return {
      title: 'Hello world!',
      content: 'Welcome to our blog. Lorem ipsum dolor sit amet...'
    };
  }
 
  throw error(404, 'Not found');
}
```

- `load`는 `+page.svelte`와 함께 실행됨
- 페이지의 동작을 구성하는 값 export([더 보기](https://kit.svelte.dev/docs/page-options))
  - `export const prerender = true(or false, auto)` 
  - `export const ssr = true(or false)`
  - `export const csr = true(or false)`

### +page.server.js

- `load` 함수가 서버에서만 실행되어야 하는 경우
  - ex) DB 에서 데이터를 가져와야 하거나 API 키와 같은 비공개 환경 변수에 액세스해야 하는 경우

## +error

- `load` 중 오류 발생 시 기본 오류 페이지 렌더링
- 경로마다 `error.svelte` 파일을 추가하여 오류 페이지 사용자 정의 가능
- Svelte는 가장 가까운 error boundary를 찾기 위해 상위 디렉토리로 올라가며 검색
  - `src/error.html` 파일 통해 static한 가장 기본이되는 에러페이지 커스터마이징 가능

> +error.svelte는 handle이나 +server.js 요청 핸들러 내에서 오류가 발생할 때 사용되지 않는다.


## +layout

- 모든 페이지에서 보여져야 하는 요소(헤더, 푸터, ...)가 있는 경우 사용

### +layout.svelte

- `src/routes/+layout.svelte`
- `<slot></slot>`을 반드시 포함해야 함
- 중첩 가능
  - ex) /settings 페이지 뿐만 아니라 /settings/profile, /settings/notifications 같은 중첩된 페이지에도 적용
  - ex) /settings 페이지에만 적용하고 싶은 경우 : `src/routes/settings/+layout.svelte`
- 기본적으로 각각의 레이아웃은 그 위의 레이아웃을 상속함
 
### +layout.js

- `+page.svelte`는 `+page.js` 통해 데이터 로드
- 마찬가지로, `+layout.svelte`는 `+layout.js` 통해 데이터 로드
- `layout.js`에서 export 한 options(ex. prerender, ssr, csr, ...)는 자식 페이지에도 적용 됨
- `load` 함수를 통해 반환된 데이터 역시 자식 페이지에서 사용 가능

### +layout.server.js

- `load` 함수를 서버에서 실행하기 위해 `layout.server.js` 사용

## +server

- 페이지 외에 `+server.js`파일을 사용하여 라우트 정의 가능(`API 라우트 or 엔드포인트` 라고 부름)
- 해당 파일은 HTTP 동사(GET, POST, PATCH, PUT, DELETE, OPTIONS 등)에 해당하는 함수를 export
  - 해당 함수는 `RequestEvent`를 매개변수로 받으며, Response 객체를 반환
  - 해당 함수는 서버에서 실행되며 DB에 접근하여 데이터를 응답으로 반환하거나 외부 API 요청에 대한 결과를 반환

<br/>

# 참고

- [SvelteKit 공식 문서](https://kit.svelte.dev/docs)
