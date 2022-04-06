---
title: "[React] 누구든지 하는 리액트: 초심자를 위한 react 핵심 강좌 - velopert"
last_modified_at: 2022-04-07T02:30:00+09:00
categories:
    - Front End
    - React
    - Inflearn
tags:
    - Front End
    - React
    - Inflearn
toc: true
toc_sticky: true
toc_label: "목차"
---

React : 김민준(velopert)님의 인프런강의 '누구든지 하는 리액트: 초심자를 위한 react 핵심 강좌'를 기반으로 기록해 놓고 싶은 내용들을 작성했습니다. 
{: .notice--info}


# 특별한 점

- 어마어마한 생태계
  - 2006년 jQuery 급으로 큰관심
- 사용하는곳이 많다.
  - Airbnb, BBC, FACEBOOK ...
- 한번 사용하면 좋아하게 된다.

<br>

# 시작하기

- Webpack
  - 코드들을 의존 순서대로 하나 또는 여러개로 만들어냄
- Babel
  - javascript 변환 도구
- 개발 Starter 도구 : [codesandbox](https://codesandbox.io/s/react-basics-forked-wx31t9)

<br>

# JSX

- React 컴포넌트 작성할 때 사용되는 문법
  - html과 비슷하지만 규칙이 있음 ([참고 자료](https://react-anyone.vlpt.us/03.html))

<br>

# 키워드 기록

- 함수형 컴포넌트
  - 함수만 존재하는 컴포넌트
- props
  - 부모 -> 자식, ReadOnly
- state
  - 컴포넌트 스스로가 값을 소유 
  - setState()을 통해서만 값 변경 가능 (this. 로 접근 시 바인드 오류 발생)
- constructor
  - Component가 생성될때마다 수행되는 함수
- LifeCycle API
  - [참고 URL](https://react-anyone.vlpt.us/05.html)
  - 필요할 때마다 찾아서 사용하면 됨.

![image](https://user-images.githubusercontent.com/53864640/162035822-81551023-2e1d-4cf2-8532-36e837f84283.png){: .align-center}
_- [그림 출처](https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)_
{: .text-center}

<br>

# 개발

## 단축키 (in VS Code)

### Extension : Reactjs code snippets
 
- `rcc` + `enter` : React 기본 클래스 컴포넌트 템플릿 작성
- `rsc` + `enter` : React 기본 함수형 컴포넌트 템플릿 작성
- `scu` + `enter` : `shouldComponentUpdate(nextProps, nextState)`

## Tips

- submit 새로고침 방지
```js
handleSubmit = (e) => {
    e.preventDefault();
}
```

- props 에 key를 줘야 하는 이유
  - key가 없으면 Component 가 생성/삭제 될 때 전체적인 주소변화가 발생함.
  - ex) a, b, c, d -> a, b, **z**, c, d : z가 중간에 끼어들게 되면,  
    - c -> z
    - d -> c
    - d 신규 추가
  - 와 같이 쓸모없이 많은 변환이 발생함.

- 불변성 유지를 위한 라이브러리 : [Immutable.js](https://immutable-js.com/), [Immer.js](https://github.com/immerjs/immer)

<br>

# 참고

- [누구든지 하는 리액트: 초심자를 위한 react 핵심 강좌](https://www.inflearn.com/course/react-velopert/dashboard)