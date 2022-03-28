---
title: "[Vue 3.0] 문법 정리 - Composition API & setup 함수"
last_modified_at: 2022-03-28T22:00:00+09:00
categories:
    - Front End
    - Vue 3.0
tags:
    - Front End
    - Vue 3.0
toc: true
toc_sticky: true
toc_label: "목차"
---

Vue 3.0 : 기본 문법에 대해 정리한다. Vue 2.0과 달라진 부분과 그 동안 모르고 사용했던 개념을 정리한다.
{: .notice--info}

# Composition API & setup function

```vue

<template>
  <div>{{ name }} {{ isSubmitted }}</div>
  <button @click="onSubmit">Submit</button>
</template>

<script>
import {ref, onMounted} from 'vue'

export default {
  // props 설정
  props: {
    name: String
  },
  setup(props) {
    const isSubmitted = ref(false);  // ref = Vue 2.0 의 data 와 동일

    // Vue 2.0 의 method 대신 그냥 javascript method 사용
    const onSubmit = () => {
      isSubmitted.value = true;
    }

    // Vue 2.0의 lifeCycle 이 함수로 선언 됨.
    onMouted(() => console.log('component mounted'));

    // template 에 전달되어야 하는 변수 또는 메서드를 별도로 명시해줘야 함.
    return {
      isSubmitted,
      onSubmit
    }
  }
}
</script>

```

setup 함수는 컴포넌트 인스턴스가 생성되기 전에 실행 됨.<br>
(즉, 컴포넌트 인스턴스에 접근이 필요한 기능은 사용 불가)

`this`를 통해 `data, computed, methods` 에서 선언한 것들에 대한 접근이 불가능하다.

또한, 인스턴스 생성 직후 호출되는 `created` 라이프 사이클 훅도 사용할 수 없다.

<br>

# 커스텀 훅을 사용한 로직 분리 및 재활용

- Vue 2.0 의 mixins 의 경우 컴포넌트 간에 동일 변수명을 사용하면 덮어씌기가 되는 불편함이 있었다.
- Vue 3.0 에서는 `커스텀 훅`을 이용해서 이러한 불편함을 없앨 수 있게 되었다. (React 에서 자주 사용하는 방식이라고 한다.)

EX) Select 컴포넌트를 위한 옵션을 서버에서 가져오는 함수

```javascript
export default function useSomeOptions() {
    const options = ref([]);

    onMounted(() => {
        axios.get('https://api.server.com').then(({data}) => {
            options.value = data;
        })
    })

    return {
        options
    }
}
```

```vue

<template>
  <select>
    <option v-for="(item, index) in options" :key="index">{{ item }}</option>
  </select>
</template>

<script>
import useSomeOptions from './useSomeOptions'

export default {
  name: 'Page component',
  setup() {
    const {options} = useSomeOptions();
    return {
      options,
    };
  },
}
</script>

<style></style>
```

<br>

# script setup

- setup 함수를 사용할 때 불편한 점이 있다. data, methods와 마찬가지로 return 을 통해 템플릿으로 값을 넘기는 과정이 필요하다는 점이다.
- 이는 버그를 유발 할 수 있으며, 개발 생산성을 낮추는 요인중 하나이다.
- 그리고 외부 Component 사용 시 모듈 import 이후 `components` 속성에 추가해주어야 한다는 불편함이 있다.
- Vue3에서 이러한 것들을 해결해 주는 것이 script setup이다.

```vue
<template>
  <!-- 템플릿 안에서 바로 사용 가능 -->
  <div @click="log">{{ msg }}</div>
</template>

<script setup>
// 변수 선언
const msg = 'Hello!'

// 함수 선언
function log() {
  console.log(msg)
}
</script>
```

- Component 모듈도 import만 해주면 template 안에서 사용 가능하다.

<br>

# 참고

- [Vue 3의 setup 기능이 제공하는 간결한 컴포넌트 문법](https://blog.rhostem.com/posts/2021-09-17-vue-3-script-setup)
- [기억보다 기록을](https://kyounghwan01.github.io/blog/Vue/vue3/composition-api/)