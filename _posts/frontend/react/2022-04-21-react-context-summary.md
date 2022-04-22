---
title: "[React] 모던 리액트 : 문법 정리"
last_modified_at: 2022-04-22T08:30:00+09:00
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

React : 리액트에서 사용되는 문법을 정리한다. 해당 글에서 사용되는 모든 예제는 [벨로퍼트와 함께하는 모던 리액트](https://react.vlpt.us/)에서 발췌하였으며 강조를 위해 문법에 맞지 않지만 불필요한 코드들은 생략하였음을 참고 바랍니다.
{: .notice--info}

# useState

state
: 컴포넌트에서 동적으로 관리되는 값

useState
: 컴포넌트에서 상태를 관리 할 수 있는 함수

```jsx
import React, { useState } from 'react';

function Counter() {
  const [number, setNumber] = useState(0);

  const onIncrease = () => {
    setNumber(number + 1);
  }

  const onDecrease = () => {
    setNumber(number - 1);
  }

  return (
    <div>
      <h1>{number}</h1>
      <button onClick={onIncrease}>+1</button>
      <button onClick={onDecrease}>-1</button>
    </div>
  );
}
```

- `useState(initialValue)` : 파라미터로 초기 세팅값을 전달
- `set{state name}(xxx)` : state의 setter 함수는 선언한 변수명 앞에 `set`을 붙이는 컨벤션을 따른다.

```js
  const onIncrease = () => {
    setNumber(prevNumber => prevNumber + 1);
  }
```

- 위와 같이 Setter 함수에서 기존 값을 파라미터로 사용할 수 있음. (`함수형 업데이트`)

<br>

# 여러 Input 상태 관리

```jsx
import React, { useState } from 'react';

function InputSample() {
  const [inputs, setInputs] = useState({
    name: '',
    nickname: ''
  });

  const { name, nickname } = inputs; // 비구조화 할당을 통해 값 추출

  const onChange = (e) => {
    const { value, name } = e.target; // 우선 e.target 에서 name 과 value 를 추출
    setInputs({
      ...inputs, // 기존의 input 객체를 복사한 뒤
      [name]: value // name 키를 가진 값을 value 로 설정
    });
  };

  return (
    <div>
      <input name="name" placeholder="이름" onChange={onChange} value={name} />
      <input name="nickname" placeholder="닉네임" onChange={onChange} value={nickname}/>
    </div>
  );
}

export default InputSample;
```

- 위와 같이 `<input>`태그 안에 `name` 속성을 이용하여 event(`onChange`) 내에서 `e.target.name`을 통해 setState 에 어떤 state를 세팅할지 분기 처리 가능

<br>

# useRef

특정 DOM을 선택하기 위한 속성

- 기존 Javascript
  - DOM Selector 함수 사용 : `getElementById`, `querySelector`
- in React
  - useRef Hook 함수 사용

함수형 컴포넌트에서는 `useRef`, 클래스형 컴포넌트에서는 콜백 함수 사용하거나 `React.createRef` 함수 사용

```jsx
import React, { useState, useRef } from 'react';

function InputSample() {
  const [inputs, setInputs] = useState({
    name: '',
    nickname: ''
  });
  const nameInput = useRef();

  const onReset = () => {
    setInputs({
      name: '',
      nickname: ''
    });
    nameInput.current.focus();
  };

  return (
    <div>
      <input
        name="name"
        placeholder="이름"
        onChange={onChange}
        value={name}
        ref={nameInput}
      />
    </div>
  );
}

export default InputSample;
```

- `<input>` 태그 안에 `ref` 라는 속성을 사용하여 기존에 `useRef()`를 통해 선언한 `nameInput`를 맵핑 하였다.
- 맵핑을 통해 `nameInput`는 이제 `<input>` 을 대신할 수 있게 된다.
- `useRef(initialVal)` : DOM 접근말고도 컴포넌트 내에서 변수 선언 할 때도 사용 됨. useRef의 파라미터로 값을 넣어주면 `.current` 값의 기본값이 되며, someVariable.current 로 값을 조회할 수 있음.

<br>

# useEffect

- 호출되는 시점
  - 컴포넌트가 마운트 됐을 때(처음 나타날 때)
  - 언마운트 됐을 때(사라질 때)
  - 업데이트 될 때(특정 props가 바뀔 때)

## deps 에 특정 값 없는 경우

```jsx
import React, { useEffect } from 'react';

function User({ user, onRemove, onToggle }) {
  useEffect(() => {
    console.log('컴포넌트가 화면에 나타남');
    return () => {
      console.log('컴포넌트가 화면에서 사라짐');
    };
  }, []);
  return (
    <div>
      <b
        style={{
          cursor: 'pointer',
          color: user.active ? 'green' : 'black'
        }}
        onClick={() => onToggle(user.id)}
      >
        {user.username}
      </b>
      &nbsp;
      <span>({user.email})</span>
      <button onClick={() => onRemove(user.id)}>삭제</button>
    </div>
  );
}

function UserList({ users, onRemove, onToggle }) {
  return (
    <div>
      {users.map(user => (
        <User
          user={user}
          key={user.id}
          onRemove={onRemove}
          onToggle={onToggle}
        />
      ))}
    </div>
  );
}

export default UserList;
```

- useEffect(function, deps)
  - 첫번째 파라미터 : `cleanup` 함수. useEffect 의 뒷정리 담당
  - 두번째 파라미터 : 의존 값이 들어있는 배열 (`deps`)
    - deps 배열을 비운 경우 : 컴포넌트가 나타날 때만 `cleanup` 함수 호출
    - deps 자체가 비어 있는 경우 : 컴포넌트가 사라질 때 `cleanup` 함수 호출

## deps 에 특정 값 있는 경우

```jsx
import React, { useEffect } from 'react';

function User({ user, onRemove, onToggle }) {
  useEffect(() => {
    console.log('user 값이 설정됨');
    console.log(user);
    return () => {
      console.log('user 가 바뀌기 전..');
      console.log(user);
    };
  }, [user]);
  return (
    <div>
      <b
        style={{
          cursor: 'pointer',
          color: user.active ? 'green' : 'black'
        }}
        onClick={() => onToggle(user.id)}
      >
        {user.username}
      </b>
      &nbsp;
      <span>({user.email})</span>
      <button onClick={() => onRemove(user.id)}>삭제</button>
    </div>
  );
}

function UserList({ users, onRemove, onToggle }) {
  return (
    <div>
      {users.map(user => (
        <User
          user={user}
          key={user.id}
          onRemove={onRemove}
          onToggle={onToggle}
        />
      ))}
    </div>
  );
}

export default UserList;
```

- deps 에 특정 값을 넣은 경우
  - 컴포넌트가 처음 마운트 될 때 호출
  - 지정한 값이 바뀔 때 호출
- deps 안에 특정 값이 있다면
  - 언마운트시에도 호출
  - 값이 바꾸기 전에도 호출
- useEffect 안에서 사용하는 상태나 props 가 있는 경우 useEffect 의 deps 에 넣어주는 것이 규칙이다.
  - 넣지 않은 경우 : useEffect 에 등록한 함수가 실행 될 때 최신 상태와 props를 가르키지 않게 됨.

## deps 파라미터 생략하기

- 컴포넌트가 리렌더링 될 때마다 호출

```jsx
import React, { useEffect } from 'react';

function User({ user, onRemove, onToggle }) {
  useEffect(() => {
    console.log(user);
  });
}
```

- 참고로 리액트 컴포넌트는 부모 컴포넌트가 리렌더링 되면 (자식은 바뀐 내용이 없어도)자식 컴포넌트도 같이 리렌더링 됨.
  - 이는 최적화가 필요

<br>

# useMemo

- 특정 연산을 처리하는 함수가 계속 해서 리렌더링 되는 컴포넌트에 의존되어 있다면, 특정 함수는 해당 컴포넌트가 리렌더링 될 때 마다 리턴 값이 변경되지 않았더라도 계속 호출된다.
- `useMemo` Hook 함수는 이전에 계산한 값을 재사용할 수 있도록 하는 함수이다. (성능 최적화)

```jsx
import React, { useState, useMemo } from 'react';

function countActiveUsers(users) {
  console.log('활성 사용자 수를 세는중...');
  return users.filter(user => user.active).length;
}

function App() {
    const [users, setUsers] = useState([
        {
          id: 1,
          username: 'velopert',
          email: 'public.velopert@gmail.com',
          active: true
        },
    ]);

    const count = useMemo(() => countActiveUsers(users), [users]);
}

```

- useMemo
  - 첫 번째 파라미터 : 어떻게 연산할지 정의하는 함수
  - 두 번째 파라미터 : deps 배열, 배열 안에 넣은 내용이 바뀌면 등록한 함수를 호출하여 값을 연산, 만약 내용이 바뀌지 않았더라면 이전 연산 값 재사용

<br>

# useCallback

- `useMemo`와 비슷한 Hook
- `useMemo`는 특정 결과값을 재상용하는 반면, `useCallback`은 특정 함수를 새로 만들지 않고 재사용하고 싶을 때 사용한다.

컴포넌트 내에 선언된 함수들은 컴포넌트가 리렌더링 될 때 마다 새로 만들어진다.

함수를 선언하는 것 자체는 크게 성능에 영향을 주지 않지만, 

나중에 컴포넌트에서 `props`가 바뀌지 않았으면 Virtual DOM 에 새로 렌더링 하는 것 조차 하지 않고 컴포넌트의 결과물을 재사용하는 최적화 작업을 하게 되는데,

이 작업을 하려면, 함수를 재사용하는 것은 필수이다.

```jsx
import React, { useRef, useState, useMemo, useCallback } from 'react';
import UserList from './UserList';
import CreateUser from './CreateUser';

function App() {
  const [inputs, setInputs] = useState({
    username: '',
    email: ''
  });
  const { username, email } = inputs;
  const onChange = useCallback(
    e => {
      const { name, value } = e.target;
      setInputs({
        ...inputs,
        [name]: value
      });
    },
    [inputs]
  );
  const [users, setUsers] = useState([
    {
      id: 1,
      username: 'velopert',
      email: 'public.velopert@gmail.com',
      active: true
    },
  ]);

  const nextId = useRef(4);
  const onCreate = useCallback(() => {
    const user = {
      id: nextId.current,
      username,
      email
    };
    setUsers(users.concat(user));

    setInputs({
      username: '',
      email: ''
    });
    nextId.current += 1;
  }, [users, username, email]);

  const onRemove = useCallback(
    id => {
      // user.id 가 파라미터로 일치하지 않는 원소만 추출해서 새로운 배열을 만듬
      // = user.id 가 id 인 것을 제거함
      setUsers(users.filter(user => user.id !== id));
    },
    [users]
  );
  const onToggle = useCallback(
    id => {
      setUsers(
        users.map(user =>
          user.id === id ? { ...user, active: !user.active } : user
        )
      );
    },
    [users]
  );
  return (
    <>
      <CreateUser
        username={username}
        email={email}
        onChange={onChange}
        onCreate={onCreate}
      />
      <UserList users={users} onRemove={onRemove} onToggle={onToggle} />
    </>
  );
}

export default App;
```

- 사용시 주의
  - useCallback 내에서 사용되는 props가 있다면 무조건 deps 배열안에 포함시켜야 함
    - 넣지 않은 경우 해당 값을 참조할 때 가장 최신 값임을 보장 할 수 없음.
  - props 로 받아온 함수가 있다면 이 또한 deps 에 넣어 줘야 함.

<br>

# React.memo

props 가 바뀌지 않았더라면 컴포넌트 리렌더링을 하지 않도록 방지하는 최적화 함수

이 함수를 사용하면, 리렌더링이 필요한 상황에서만 리렌더링을 하도록 설정 가능

```jsx
import React from 'react';

const CreateUser = ({ username, email, onChange, onCreate }) => {
    ...
}

export default React.memo(CreateUser);
```

- `React.memo(SomeComponent)` 와 같이 컴포넌트를 감싸주기만 하면 됨.

<br>

# useReducer

- 컴포넌트 내부에서 처리되는 로직을 분리하기 위해 사용
- 외부 파일에 작성하여 불러와 사용 가능

```jsx
function reducer(state, action) {
  // 새로운 상태를 만드는 로직
  // const nextState = ...
  return nextState;
}
```

- reducer가 반환할 상태는 곧 컴포넌트가 지닐 새로운 상태
- `action`은 업데이트를 위한 정보를 가지고 있음. 주로 `type` 값을 지닌 객체 형태로 사용(정해진 규칙은 없음)

```javascript
// 카운터에 1을 더하는 액션
{
  type: 'INCREMENT'
}
// 카운터에 1을 빼는 액션
{
  type: 'DECREMENT'
}
// input 값을 바꾸는 액션
{
  type: 'CHANGE_INPUT',
  key: 'email',
  value: 'tester@react.com'
}
// 새 할 일을 등록하는 액션
{
  type: 'ADD_TODO',
  todo: {
    id: 1,
    text: 'useReducer 배우기',
    done: false,
  }
}
```

- 위 소스는 액션 객체 예시이다.

```javascript
const [state, dispatch] = useReducer(reducer, initialState);
```

- reducer 사용법
  - state : 컴포넌트에서 사용 할 수 있는 상태
  - dispatch : 액션을 발생시키는 함수(ex. `dispatch({ type: 'INCREMENT' }))
  - useReducer
    - 첫 번째 파라미터 : reducer 함수
    - 두 번째 파라미터 : 초기 상태값


## 적용 전

```jsx
import React, { useState } from 'react';

function Counter() {
  const [number, setNumber] = useState(0);

  const onIncrease = () => {
    setNumber(prevNumber => prevNumber + 1);
  };

  const onDecrease = () => {
    setNumber(prevNumber => prevNumber - 1);
  };

  return (
    <div>
      <h1>{number}</h1>
      <button onClick={onIncrease}>+1</button>
      <button onClick={onDecrease}>-1</button>
    </div>
  );
}

export default Counter;
```

## 적용 후

```jsx
import React, { useReducer } from 'react';

function reducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1;
    case 'DECREMENT':
      return state - 1;
    default:
      return state;
  }
}

function Counter() {
  const [number, dispatch] = useReducer(reducer, 0);

  const onIncrease = () => {
    dispatch({ type: 'INCREMENT' });
  };

  const onDecrease = () => {
    dispatch({ type: 'DECREMENT' });
  };

  return (
    <div>
      <h1>{number}</h1>
      <button onClick={onIncrease}>+1</button>
      <button onClick={onDecrease}>-1</button>
    </div>
  );
}

export default Counter;
```

## useReducer vs useState

- 상황에 따라 사용
- 컴포넌트에서 관리하는 값이 딱 하나인 경우 `useState`
- 컴포넌트에서 관리하는 값이 여러개이며 구조가 복잡한 경우 `useReducer`
- 그러나 이것은 정답이 없고, 경험적으로 사용하기 편한것을 결정하도록 하자.

<br>

# 커스텀 Hooks 만들기

- 반복되는 로직을 쉽게 재사용 할 수 있도록 하는 기능
- 커스텀 Hooks 사용
  - `use` 라는 prefix가 붙은 파일을 생성하여 사용
  - `useState, useEffect, useReducer, useCallback` 등 Hooks 를 사용하여 원하는 기능을 구현하여 컴포넌트 내에서 사용하고 싶은 값들을 반환한다.

```jsx
// useInput.js
import { useState, useCallback } from 'react';

function useInputs(initialForm) {
  const [form, setForm] = useState(initialForm);
  // change
  const onChange = useCallback(e => {
    const { name, value } = e.target;
    setForm(form => ({ ...form, [name]: value }));
  }, []);
  const reset = useCallback(() => setForm(initialForm), [initialForm]);
  return [form, onChange, reset];
}

export default useInputs;
```

```jsx
import React from 'react';
import useInputs from './hooks/useInputs';

function App() {
  const [{ username, email }, onChange, reset] = useInputs({
    username: '',
    email: ''
  });
  
  ...
}

export default App;
```

<br>

# 참고

- [벨로퍼트와 함께하는 모던 리액트](https://react.vlpt.us/)