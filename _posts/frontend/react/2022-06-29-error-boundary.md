---
title: "[React] Error Boundary"
last_modified_at: 2022-06-30T11:30:00+09:00
categories:
    - Front End
    - React
tags:
    - Front End
    - React
toc: true
toc_sticky: true
toc_label: "목차"
---

React : React 16에 등장한 에러 핸들링 방식인 Error Boundary에 대해 알아보자. 
{: .notice--info}

# 배경

만약 **예상치 못한 이유로 에러가 발생**했는데 별도의 처리를 하지 않았다면 아래와 같은 **하얀색 화면**을 마주하게 될 것이다.

<img width="588" alt="image" src="https://user-images.githubusercontent.com/53864640/176354600-33c6a5a3-8b05-41d4-b7f8-abad7677462f.png">

사용자에게 이러한 화면을 제공하는 것은 상당히 **불친절**하다. 또한, 개발자 입장에서도 에러가 어디서 발생했는지 추적하고 싶지만 별도로 기록하지 않는다면 이는 어려울 것이다.

이러한 문제점을 해결하기 위해 React에서 에러를 어떻게 핸들링 하는지 알아볼 필요가 있다.

# 소개

## try/catch

가장 **원초적인 에러 핸들링 방식**으로는 `try/catch`가 있다.

``` javascript
import * as React from 'react'
import ReactDOM from 'react-dom'

function ErrorFallback({error}) {
  return (
    <div role="alert">
      <p>Something went wrong:</p>
      <pre style={{color: 'red'}}>{error.message}</pre>
    </div>
  )
}

function Greeting({subject}) {
  try {
    return <div>Hello {subject.toUpperCase()}</div>
  } catch (error) {
    return <ErrorFallback error={error} />
  }
}

function Farewell({subject}) {
  try {
    return <div>Goodbye {subject.toUpperCase()}</div>
  } catch (error) {
    return <ErrorFallback error={error} />
  }
}

function App() {
  return (
    <div>
      <Greeting />
      <Farewell />
    </div>
  )
}

ReactDOM.render(<App />, document.getElementById('root'))
```

위 소스는 `Greeting`, `Farewell` Component 의 Props로 `subject`를 안 넘긴 경우(undefined) `subject.toUpperCase()`메서드 에서 **NPE 에러를
발생**시켜 error가 catch되어`ErrorFallback` 컴포넌트를 Rendering 해주는 소스 코드이다.

이는 정상적으로 의도한대로 동작한다.

하지만 이는 App 내 모든 컴포넌트에 `try/catch`를 감싸주어야 하는 **문제점이 존재**한다.

따라서, `try/catch`의 특징을 살려 각 컴포넌트 내부가 아닌 전체를 감싸 바깥에 `try/catch`을 wrapping 해줄 수도 있다.

``` javascript
import * as React from 'react'
import ReactDOM from 'react-dom'

function ErrorFallback({error}) {
  return (
    <div role="alert">
      <p>Something went wrong:</p>
      <pre style={{color: 'red'}}>{error.message}</pre>
    </div>
  )
}

/* try/catch 제거 */
function Greeting({subject}) {
  return <div>Hello {subject.toUpperCase()}</div>
}

/* try/catch 제거 */
function Farewell({subject}) {
  return <div>Goodbye {subject.toUpperCase()}</div>
}

/* 전체 컴포넌트를 try/catch로 wrapping */
function App() {
  try {
    return (
      <div>
        <Greeting />
        <Farewell />
      </div>
    )
  } catch (error) {
    return <ErrorFallback error={error} />
  }
}

ReactDOM.render(<App />, document.getElementById('root'))
```

그치만 이는 동작하지 않는다.

왜냐하면, `try/catch`는 **명령형(imperative) 코드에서만 작동**하는데 React의 컴포넌트는 명령에 의해 호출(Calling)되는 것이 아닌 **선언(Declarative)적으로 존재**하기
때문이다.

따라서, 이러한 문제점을 극복하기 위해 나타난 것이 **Error Boundary**이다.

<br/>

## Error Boundary

### 특징

`Error Boundary`는 **React 16**에 들어서며 새로운 개념으로 소개되었으며 다음과 같은 기능을 제공해준다.

1. Error Boundary가 감싸고 있는 모든 `자식 컴포넌트` 내에서 발생한 Javascript 에러를 **Catch** 할 수 있다.
2. 에러를 **Logging** 할 수 있다.
3. `깨진 화면(흰 화면)` 대신 `fallback UI`를 **Display**할 수 있다.

### 사용법

`Error Boundary`도 일종의 **컴포넌트**이다.

**다른 컴포넌트와 차이점**이 있다면 React 생애주기(Lifecycle) 중 `static getDerivedStateFromError()` or `componentDidCatch()`를 포함한 **클래스(Class) 컴포넌트**라는 점이다.

해당 생애주기는 **함수형 컴포넌트에서 구현 불가능**하므로 반드시 **클래스 컴포넌트**로 생성되어야 한다.

* `static getDerivedStateFromError()`
    * **용어 풀이** : Error로 부터 파생된 State 값을 얻는다.
        * *state 란? -* Component 내에서 사용되는 상태값
    * **용도** : 하위의 자손 컴포넌트에서 오류가 발생했을 때 render 단계에서 호출
* `componentDidCatch()`
    * **용어 풀이** : 컴포넌트가 Catch를 수행 했다.
    * **용도** : 하위의 자손 컴포넌트에서 오류가 발생했을 때 commit 단계에서 호출, Error log를 남기기 위해 사용

``` javascript
// 클래스 컴포넌트
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

static getDerivedStateFromError(error) { 
    // fallback UI 에 사용하기 위한 state 업데이트 
    return { hasError: true }; // <- return된 값이 state에 반영 
}

componentDidCatch(error, errorInfo) { 
    // 발생한 error를 logging 및 reporting 
    logErrorToMyService(error, errorInfo); 
}

render() { 
    if (this.state.hasError) { 
        // You can render any custom fallback UI 
        return <h1>Something went wrong.</h1>; 
    }

    return this.props.children; 
} }
```

* `componentDidCatch(error, errorInfo)` \> errorInfo 샘플

```
{ componentStack : 
  " at HomeContainer (http://localhost:3090/static/js/bundle.js:27819:65)
    at Outlet (http://localhost:3090/static/js/bundle.js:145634:26)
    at div at MainLayout (http://localhost:3090/static/js/bundle.js:2049:84)
    at Routes (http://localhost:3090/static/js/bundle.js:145733:24)
    at ErrorBoundary (http://localhost:3090/static/js/bundle.js:3540:163)
    at App (http://localhost:3090/static/js/bundle.js:657:78)
    at Provider (http://localhost:3090/static/js/bundle.js:141667:20)
    at Router (http://localhost:3090/static/js/bundle.js:145658:30)
    at BrowserRouter (http://localhost:3090/static/js/bundle.js:144386:23)"
}
```

이렇게 생성된 `ErrorBoundary`를 통해 아래와 같이 에러가 커버되어야 하는 컴포넌트를 감싸주면 된다.

``` javascript
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>
```

<br/>

# 주의

* `ErrorBoundary` 자체 내부에서 발생하는 에러는 스스로 `catch` 할 수 없다. -> 만약 `ErrorBoundary`를 감싼 또 다른 `ErrorBoundary`가 있다면 그쪽으로 에러를 전파할
  것이다.

<br/>

# 심화

## react-error-boundary

`ErrorBoundary`가

1. 과거의 컴포넌트 선언 방식인 Class Component 사용
2. `getDerivedStateFromError` or `componentDidCatch` 생애주기 구현

위와 같은 규칙을 반드시 지켜야한다는 불편함(?) 때문에 `react-error-boundary` 라이브러리가 등장하게 되었다.

해당 라이브러리에 대한
설명은 [react-error-boundary 블로그](https://kentcdodds.com/blog/use-react-error-boundary-to-handle-errors-in-react)를 통해 알아보도록
하자.

# 참고

* [Error Boundaries 공식사이트](https://reactjs.org/docs/error-boundaries.html)
* [Use react-error-boundary to handle errors in React](https://kentcdodds.com/blog/use-react-error-boundary-to-handle-errors-in-react)