---
title: "[Svelte] Svelte 파헤치기"
last_modified_at: 2023-04-10T22:00:00+09:00
categories:
    - Front End
    - Svelte
tags:
    - Front End
    - Svelte
toc: true
toc_sticky: true
toc_label: "목차"
---

Svelte : Svelte를 알아보기 위해 필요한 것들을 같이 알아보는 파헤치기를 한 번 해볼까 한다. 관련 내용들을 두서 없이 정처 없이 써내려갈 것이다. 
{: .notice--info}

# 의식의 흐름

이 포스트를 작성하면서 내가 **알고자 하는 것과 얻어 내야 하는 것**이 무엇인지 의식의 흐름대로 작성해본다.<br/>
먼저, Svelte가 FE 프레임워크인 것은 알고 있다. 인기가 많아지고 있는 추세다 정도도 알고 있다.<br/>
그럼 과연 얼마나 인기가 많아지고 있는지 그 **추세**와 다른 FE 프레임워크랑 뭐가 다르기에 인기가 많아지고 있는지 그 **특징**을 알아봐야 할 것 같다.<br/>
**빌드 툴은 무엇**을 쓰고 있으며 **데코레이션 기능을 사용 할 수 있는지**도 확인 해보자.<br/>
그 전에 **빌드 툴을 왜 알아야 하는지 각각의 차이점**도 확인해보고 **데코레이션의 원리**도 파악해보자.<br/>
마지막으로 Svelte를 꼭 써야만 하는 이유 단 한 가지를 알아내보자. **"Svelte는 OOO 때문에 써야 한다."** 라는 문장을 이 글의 마지막에 완성시켜보자.<br/>
아마 완성시키지 못할 수도 있다.(꼭 써야할 이유를 못찾는다면.)
{: .notice--success}

생각나는대로 글을 정리할 것 이기 때문에 다소 예의없고 질서없이 글이 작성될 수 있습니다. 양해 부탁드립니다.

# Svelte 란 무엇인가?

- JavaScript 프레임워크 중 하나
- 웹 애플리케이션 개발 도구

## 특징

- **컴파일러 사용**하여 코드 변환
  - 런타임 라이브러리 크기가 매우 작음. 
  - 로드 시간 빠름
  - 실행 시간 빠름
- 작성해야할 코드 양이 적음
- `상태 관리, 라우팅` 관련 기능 제공 X. But, 라이브러리 통합하여 사용
- 최근 인기 많음

## 언제 쓰면 좋을까?

### Svelte

- 빠른 성능
- 적은 번들링 크기
- 간결한 코드
- 반응형 컴포넌트
- 높은 생상성

Svelte 컴파일러는 코드를 최적화하여 빌드 크기를 줄이고 성능을 향상시킴.

**따라서, 작은 규모의 프로젝트나 빠른 프로토타이핑에 적합**
{: .notice--warning}

그렇다면 다른 FE 프레임워크는? -> React.js, Vue.js

### React.js

- 뛰어난 생태계
- 방대한 커뮤니티
- 다양한 라이브러리, 도구 및 플러그인

컴포넌트 기반 아키텍처를 사용하여 개발자가 재사용 가능한 코드를 만들고 유지보수하기 쉽게 만듦

가상 DOM 기반 렌더링 방식 사용하여 높은 성능 제공

**따라서, 대규모의 복잡한 애플리케이션 개발 시 적합**
{: .notice--warning}

### Vue.js

- 뛰어난 생산성
- 직관적인 API
- 반응형 시스템
- 유연한 컴포넌트 아키텍처

가상 DOM 기반 렌더링 방식을 사용하여 성능을 향상 시킴

React.js와 마찬가지로 컴포넌트 기반 아키텍처를 사용하여 개발자가 재사용 가능한 코드를 만들고 유지보수하기 쉽게 만듦.

**따라서, 중간 규모의 애플리케이션 개발 시 적합**
{: .notice--warning}

<br/><br/>

# 동작 원리

## 1. Svelte 컴포넌트 정의 

- Svelte 애플리케이션 코드는 Svelte 컴포넌트로 구성
- 이 컴포넌트는 `.svelte` 확장자를 가진 파일에 작성 됨

```html
<script>
  let count = 0;

  function handleClick() {
    count += 1;
  }
</script>

<button on:click={handleClick}>
  Clicked {count} {count === 1 ? 'time' : 'times'}
</button>
```

## 2. 컴파일러 사용하여 최적화된 JavaScript 코드 생성

- Svelte 빌드를 수행하게 되면 Svelte 컴파일러가 `.svelte` 파일을 읽어 최적화된 JavaScript 코드로 변환
- Svelte 컴파일러는 `.svelte` 파일에서 HTML, CSS, JavaScript 코드를 추출하여 최적화된 JavaScript 코드로 변환
- 위 예제를 컴파일한 결과

```javascript
// 템플릿 생성
function create_fragment(ctx) {
  let button;
  return {
    c() {
      button = document.createElement("button");
      button.textContent = "Clicked " + ctx.count + " " + (ctx.count === 1 ? "time" : "times");
    },
    m(target, anchor) {
      target.insertBefore(button, anchor);
    },
    p(ctx, [dirty]) {
      if (dirty & /*count*/ 1) button.textContent = "Clicked " + ctx.count + " " + (ctx.count === 1 ? "time" : "times");
    },
    i: noop,
    o: noop,
    d(detaching) {
      if (detaching) target.removeChild(button);
    }
  };
}

//  상태, 상태 변경 함수 생성
function instance($$self, $$props, $$invalidate) {
  let count = 0;

  function handleClick() {
    $$invalidate(0, count += 1);
  }

  return { count, handleClick };
}

/* Svelte 컴포넌트
 * - 컴포넌트가 제공해야 하는 기능 담당
 *  - 컴포넌트의 라이프사이클 메서드(onMount, beforeUpdate, afterUpdate, onDestroy 등)를 구현할 수 있도록 함
 *  - createEventDispatcher 메서드를 제공하여 이벤트 디스패치(dispatch)를 가능하게 함
 *  - $$prop_def와 같은 특수 변수를 제공하여 컴포넌트 속성(props)을 정의할 수 있음
 *  - init 함수를 제공하여 컴포넌트를 초기화하고 상태(state) 변경 함수(update)를 만들 수 있음
 */
class App extends SvelteComponent {
  constructor(options) {
    super();
    init(this, options, instance, create_fragment, safe_not_equal, {});
  }
}
```

- 필요한 요소만 업데이트 하기 때문에 높은 성능 제공

## 3. 최적화된 JavaScript 코드 실행

- 빌드 결과 생성된 최적화된 JavaScript가 브라우저에서 실행 됨

## 4. 변경 사항 추적

- Svelte 는 데이터가 변경된 부분만 다시 렌더링 수행
  - 데이터가 변경되면 변경된 부분만 최적화된 JavaScript 코드에 적용되어 렌더링 됨
- 이를 위한 `변경 사항 추적 시스템` 사용

## 5. 상태 관리

# 컴파일러

- `고급 언어 -> 기계어` 변환 프로그램
- 사용시 프로그램 속도, 실행 효율성 높아짐
- 오류 검사 및 수정이 쉬움

## Svelte 컴파일러
- 개발자가 작성한 Svelte 구성 요소 코드를 변환하여 최적화된 JavaScript 코드를 생성. 이렇게 생성된 JavaScript가 브라우저에서 실행 됨
- `템플릿 코드`를 `가상 DOM` 대신 `실제 DOM API` 호출로 변환하여 실행 시간에 불필요한 연산을 줄임
  - 템플릿 코드 : 동적으로 변경되는 데이터와 함께 사용되며 동적인 데이터를 템플릿에 적용하여 최종적으로 렌더링된 HTML을 생성
  - Svelte 템플릿 코드 : HTML과 매우 유사. But, 동적인 데이터를 쉽게 바인딩 가능하게 하여 반응형 데이터 처리 기능 제공 
- 이를 통해 빠른 실행 속도와 작은 번들 크기를 가능하게 함

## Svelte 컴파일러 동작 과정

1. 파싱
   - Svelte 구성 요소 코드를 파싱하여 추상 구문 트리(Abstract Syntax Tree, AST)로 변환
2. 변환
   - AST 분석하여 JavaScript 코드를 생성하는 변환 작업 수행
   - 구성 요소의 템플릿 코드를 가상 DOM 대신 실제 DOM API 호출하여 변환
3. 생성
   - 최적화된 JavaScript 코드를 생성
   - 이 코드가 브라우저에서 실행 됨
   
## 다른 FE 프레임워크와의 차이점

### React.js

- `JSX` 라는 HTML과 유사한 특별한 문법을 사용하여 컴포넌트 정의
- Babel 또는 TypeScript와 같은 도구 사용하여 JavaScript 코드로 변환

### Vue.js

- `템플릿`을 사용하여 컴포넌트 정의
- Vue 컴파일러가 이를 JavaScript 코드로 변환

### 가상 DOM 렌더링 vs 실제 DOM API 호출을 통한 렌더링

- React.js, Vue.js
    - 가상 DOM 사용하여 렌더링 수행
    - Svelte 는 실제 DOM API 호출하여 렌더링 수행
- **가상 DOM 렌더링**
    - 메모리에 가상의 DOM 트리 유지하며 변경이 발생한 부분만 실제 DOM에 적용하는 방식
        - 실제 DOM과 동일한 구조를 메모리 상에 복제하여 가상 DOM 트리를 구성하는 과정에서 불필요한 복제 작업을 수행
        - 이에 따른 오버헤드 발생으로 성능 이슈 발생
        - 반면, 불필요한 DOM 조작을 최소화 하여및 성능 향상도 가능
- **실제 DOM API 호출 렌더링**
    - 가상 DOM 보다 빠르게 렌더링 수행 가능
        - 메모리 상에 가상 DOM 트리를 복제하는 작업이 필요하지 않기 때문에 오버해드가 적음
        - 컴파일러가 최적화된 코드 생성 과정에서 변화 감지와 관련된 최적화를 수행하여 불필요 DOM 조작을 최소화하는 방식으로 렌더링 수행
    - DOM 조작이 많은 경우 성능 문제 발생

<br/><br/>

# Svelte 빌드

## 빌드 Tool

- 기본적으로 Rollup 사용
- Svelte 애플리케이션을 더 쉽게 생성하고 관리하기 위해 **SvelteKit 이라는 공식 프레임워크**도 제공

## Rollup VS Vite

### Rollup

- JavaScript 모듈을 번들링하기 위한 효율적이고 강력한 도구
- 주로 가벼운 라이브러리 및 프레임워크 번들링을 위해 사용
- 트리 셰이킹 기능을 통해 사용하지 않는 코드를 제거하여 번들 파일 크기를 최소화

### Vite

- Rollup의 구성 복잡성을 숨기고 많은 Vite 플러그인을 사용할 수 있게 해줌
- 개발 서버 시작 시 Rollup 보다 빠름
- 실시간 개발을 위해 빠른 개발 서버와 HMR(Hot Module Replacement)를 내장
  - Rollup 도 HMR 지원하나 Vite가 보다 더 빠르고 정확한 업데이트 제공

### 둘의 공통점

- esbuild 라는 빌드 도구 사용

esbuild
: Go로 작성된 빠르고 경량화된 빌드도구
: 웹을 위한 극도로 빠른 현대적인 번들러

## (기타) Webpack

- 모듈 번들러
- HTML, CSS, JavaScript, 이미지 등의 모든 리소스들을 모듈로 보고 이를 조합하여 정적인 번들 파일을 생성하는 도구
- 번들링 된 파일을 웹 서버에서 정적 파일로 제공하거나 다이나믹한 로딩 방식을 적용할 수 있음
- 주로 JavaScript 파일을 대상으로 모듈 번들링 수행
- `로더`라는 개념을 사용하여 JavaScript가 아닌 파일들(CSS, 이미지, 폰트 등)도 모듈로 인식하고 번들링 할 수 있음
- 특징
  - 모듈 번들링 : 여러 개의 모듈을 하나의 파일로 번들링하여 처리 속도 높임
  - 로더 사용 : JavaScript가 아닌 파일도 모듈로 인식하여 처리 가능
  - 플러그인 사용 : 로더 이외의 작업 수행 가능
  - 다양한 설정 가능 : 개발, 배포 등 다양한 환경에 대한 설정 가능

## 데코레이터 사용

- Vite, Webpack 기본적으로 데코레이터 지원 X
    - Babel 설정을 통해 사용 가능

데코레이터
: ECMAScript 6에서 도입된 기능
: 클래스, 메서드, 프로퍼티 등에 주석과 비슷한 형태로 기능을 부여할 수 있게 해줌
: 함수 형태로 작성 되며 데코레이터로 동작할 대상(클래스, 메서드, 프로퍼티 등)을 인수로 받고 데코레이팅 결과를 반환
: 이때 데코레이터 함수는 대상을 변경하지 않고, 대상을 감싸는 새로운 대상을 생성하여 반환

### 데코레이터 예시

- 클래스 생성자 변경

```javascript
// MyClass를 인수로 받음
function MyDecorator(target) {
  // 클래스의 생성자를 변경
  const newConstructor = function (...args) {
    console.log("creating instance with decorator");
    return new target(...args);
  };

  // 기존과 신규 클래스 간의 프로토 타입 객체 연결
  // 데코레이터 함수가 반환하는 새로운 클래스의 프로토 타입 객체는 인자로 받은 클래스의 프로토타입 객체와 동일한 메서드와 프로퍼티를 가지고 있어야 함. 때문에 두 객체 연결
  newConstructor.prototype = target.prototype;
  return newConstructor;
}

@MyDecorator
class MyClass {
  constructor() {
    console.log("creating instance without decorator");
  }
}

const myObj = new MyClass(); // "creating instance with decorator"
```

- 프로퍼티 유효성 검증

```javascript
function validate(target, name, descriptor) {
  const method = descriptor.value;
  descriptor.value = function (...args) {
    const [arg] = args;
    if (!arg || typeof arg !== 'string') {
      throw new Error(`Invalid argument type: ${arg}`);
    }
    return method.apply(this, args);
  };
  return descriptor;
}

class Example {
  @validate
  greet(name) {
    console.log(`Hello, ${name}!`);
  }
}

const example = new Example();
example.greet('World'); // Hello, World!
example.greet(123); // throws Error: Invalid argument type: 123
```

<br/><br/>

# SvelteKit

- React의 Next, Vue의 Nuxt와 유사함
- Svelte를 사용하여 강력하고 성능이 뛰어난 웹 애플리케이션을 빠르게 개발하는 프레임워크
- Vite를 통한 빌드 수행
    - HMR(Hot Module Replacement) 지원(수정된 모듈만 바로 반영하여 재빌드 없이 빠르게 변경 사항 반영)

## 특징

- Build Optimizations
- Offline support
- Prefetch
- Configurable Rendering
- SSR, CSR, SSG

## Routing

- `filesystem-based-router` 사용

<br/><br/>

# 결론

## 쓰는 이유!

Svelte 는 **컴팩트한 프레임워크**이다.

컴팩트한 프레임워크
: 작고 가벼운 프레임워크. React, Vue와 비교할 때 코드 양이 적고 빌드된 파일의 크기가 작아서 작은 규모의 애플리케이션 개발에 적합

Svelte는 높은 성능과 간결한 코드 작성을 지원하여, 더 적은 코드로 더 높은 성능을 구현할 수 있도록 한다.