---
title: "[Next.js] 시작하기"
last_modified_at: 2022-07-16T23:50:00+09:00
categories:
    - Front End
    - Next.js
tags:
    - Front End
    - Nuxt.js
toc: true
toc_sticky: true
toc_label: "목차"
---

Nuxt.js : NextJS 기본기를 다지기 위해 정리하는 포스팅
{: .notice--info}

# Create a Next.js App

## Requirements

- Node.js (10.13 ~)

## 설치

```bash
$ npx create-next-app {project-name} --use-npm 
```

## 실행

```bash
$ cd {project-name}
$ npm run dev
```

<br>

# Navigate Between Pages

## Pages in Next.js

- Next.js의 페이지는 root에 위치한 `pages` 디랙토리에서 export된 React Component이다.
- ex) 
  - directory : `pages/index.js` -> route : `/`
  - directory : `pages/posts/first-post.js` -> route : `/posts/first-post`

## Link Component

### 사용법

```javascript
import Link from 'next/link'
```

### Before

```javascript
<h1 className="title">
  Learn <a href="https://nextjs.org">Next.js!</a>
</h1>
```

### After

```javascript
<h1 className="title">
  Read{' '}
  <Link href="/posts/first-post">
    <a>this page!</a>
  </Link>
</h1>
```

### 특징

- Client Side Rendering : javascript에 의해 화면전환이 되므로 Browser로 이동하는 것 보다 빠름
- prefetching : `production` 모드로 빌드 시 `Link`에 해당하는 페이지를 자동으로 background에 `prefetch`하여 Link 클릭 시 이미 Loading된 화면으로 빠르게 이동 가능
- Code splitting : 각 페이지마다 꼭 필요한 요소들만 Loading 하여 빠름

<br>

# Assets, Metadata, and CSS

## Assets

- 이미지와 같은 static assets을 `public` directory에 저장
  - 해당 directory에 있는 파일들은 `pages`에서 참조가능하다.
  - `robots.txt`도 `public` 하위에 사용 가능

### Image Component

`<img/>` element 대신 `<Image/>` Component 사용

- 특징
  - 화면 size에 대해 responsive
  - Viewport에 접근 시 Loading
  - 최신 이미지 format을 자동 채택
  - 외부 이미지에 대해서도 자동 최적화 제공
  - 빌드 타임이 아닌 On-demand로 image를 최적화함
  - default로 Lazy loading을 수행(viewport 밖에서 이미지에 의해 속도적으로 손해보는 일 없음)

## Metadata

페이지의 Metadata(in `<head/`>)를 수정하기 위해 사용

### `<Head/>`

- `<head/>`의 extension Component 이다.
- `next/head`를 통해 import 가능

```javascript
import Head from 'next/head';
```

## Third-Party Javascript

analytics, ads, custom support widgets 와 같은 Third-party Javascript를 적용하는 방법

### Adding Third-Party Javascript

- 가능한 빨리 load 되어야 하는 script는 보통 `<head/>`에 위치한다.

#### Before

```javascript
<Head>
  <title>First Post</title>
  <script src="https://connect.facebook.net/en_US/sdk.js" />
</Head>
``` 

- 위 경우 외부 javascript가 화면에 사용될 다른 javascript와 같은 페이지에 위치한다는 점에서 좋지 못하다.
- 또한, 외부 javascript 내에 render-blocking 하거나 delay page content 하는 경우 성능상 큰 영향을 준다.

#### After(Using the Script Component)

- `<script>`의 extension인 `next/script`를 사용하자.
- 이는 추가적인 스크립트가 패치되거나 실행될 때 최적화를 수행한다.

```javascript
import Script from 'next/script';

...
<Head>
    <title>First Post</title>
</Head>
<Script
    src="https://connect.facebook.net/en_US/sdk.js"
    strategy="lazyOnload"
    onLoad={() =>
        console.log(`script loaded correctly, window.FB has been populated`)
    }
/>
```

## CSS Styling

```javascript
<style jsx>{`
  …
`}</style>
```

- `styled-jsx` : `CSS-in-JS` 라이브러리
  - React Component 내에 CSS를 작성할 수 있도록 함
  - 이는 `scoped` 범위
  
Next.js는 built-in 된 styled-jsx를 제공함. (그치만, 유명한 `styled-components` or `emotion`을 사용할 수도 있음.)

## Layout Component

`Layout` component는 전체 페이지에 걸쳐 공유된다.

- top-level에 `components` 폴더를 만든다.
- 해당 폴더 내에 `layout.js`를 만든다.

```javascript
// layout.js
export default function Layout({ children }) {
  return <div>{children}</div>;
}
```

```javascript
// first-post.js
import Link from "next/link";
import Head from "next/head";
import Layout from "../../components/layout";

export default function FirstPost() {
    return (
        <Layout>
            <Head>
                <title>First Post</title>
            </Head>

            <h1>First Post</h1>
            <h2>
                <Link href="/">
                    <a>Back to home</a>
                </Link>
            </h2>
        </Layout>
    );
}
```

### Adding CSS

`Layout` component 에 `CSS Modules`를 이용하여 CSS를 React component에 추가한다.

- `components/layout.module.css` 파일 생성 (CSS Modules는 반드시 .module.css 명명규칙을 따라야함.)

```css
.container {
  max-width: 36rem;
  padding: 0 1rem;
  margin: 3rem auto 6rem;
}
```

```javascript
import styles from './layout.module.css';

export default function Layout({ children }) {
  return <div className={styles.container}>{children}</div>;
}
```

### Automatically Generates Unique Class Names

![image](https://user-images.githubusercontent.com/53864640/179342233-4f5f7ee4-a8ce-4168-9d50-e073e8b41b94.png)

- div의 class name이 자동으로 임의의 값이 맵핑되어 있음. (`layout_container_...`)
- 이는 CSS Modules가 자동으로 고유의 class names를 만들기 때문이다. 이를 통해 class name 의 충돌을 걱정할 필요가 없어진다.
- CSS Modules 또한 code splitting 을 수행한다. -> 각 화면당 최소한의 CSS loading 가능 = smaller bundle sizes
- CSS Modules 는 build time에 번들되어 .css 파일로 추출된다.

## Global Styles

- CSS Modules는 component-level 에서 유용하게 사용된다.
- 전체 페이지에 CSS 적용을 위해 Next.js에서 지원하는 것을 사용하자.

### global CSS 적용

- `pages/_app.js` 파일 생성

```javascript
export default function App({ Component, pageProps }) {
  return <Component {...pageProps} />;
}
```

- `App` component는 모든 다른 페이지가 거쳐가는 최상위 컴포넌트이다.
- `_app.js` 추가 후 development server를 재시작 한다.
- global CSS를 `_app.js`에 import 한다. (그 외 파일에는 global CSS를 import 할 수 없다.)
- global CSS 파일은 어디에나 아무 이름으로 존재할 수 있다.
  - top-level에 `styles` directory를 생성하고 안에 `global.css` 파일을 생성한다.

```css
/*global.css*/
html,
body {
  padding: 0;
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, Segoe UI, Roboto, Oxygen, Ubuntu,
    Cantarell, Fira Sans, Droid Sans, Helvetica Neue, sans-serif;
  line-height: 1.6;
  font-size: 18px;
}

* {
  box-sizing: border-box;
}

a {
  color: #0070f3;
  text-decoration: none;
}

a:hover {
  text-decoration: underline;
}

img {
  max-width: 100%;
  display: block;
}
```

```javascript
// _app.js
import '../styles/global.css';

export default function App({ Component, pageProps }) {
  return <Component {...pageProps} />;
}
```

## Polishing Layout

- 다음 lesson 수행 전 전체적인 파일 정리 > 다음 [페이지](https://nextjs.org/learn/basics/assets-metadata-css/polishing-layout) 를 참고하여 진행

<br>

# Pre-rendering and Data Fetching

## Pre-rendering

- Next.js 는 defualt 로 모든 페이지를 pre-render
- 즉, 모든 페이지에 대해 HTML을 생성한다.
- 이는 성능상이네 SEO 측면에서 유리하다.
- browser 에 페이지가 load 될 때 Javascript 가 실행되며 완전히 interactive한 페이지를 만든다. (이러한 프로세스를 hydration 이라고 부른다.)
- 순수 React.js는 Pre-rendering 을 지원하지 않는다.

![image](https://nextjs.org/static/images/learn/data-fetching/pre-rendering.png)

![image](https://nextjs.org/static/images/learn/data-fetching/no-pre-rendering.png)

## Two Forms of Pre-rendering

### Static Generation / Server-side Rendering

- HTML 페이지가 만들어지는 시점에 차이가 있다.
- Static Generation : build time에 HTML 생성. 생성된 HTML은 요청마다 재사용 된다.
- Server-side Rendering : 각 요청마다 HTML이 생성된다.
  
![image](https://nextjs.org/static/images/learn/data-fetching/static-generation.png)

![image](https://nextjs.org/static/images/learn/data-fetching/server-side-rendering.png)

- (주의) development 모드에서는 Static Generation 이더라도 각 요청마다 페이지를 생성한다.

### Per-page Basis

각 페이지 마다 Static Generation 과 Server-side Rendering 을 선택하여 hybrid 하게 Next.js app 을 만들 수 있다.

### When to Use Static Generation vs Server-side Rendering

- Static Generation
  - 가능하면 Static Generation 사용을 권장
  - 빌드 한번으로 페이지가 생성되며, CDN 사용이 가능하다.
  - 매 요청마다 server rendering 하는 것보다 훨씬 빠르다. 
  - Ex)
    - Marketing pages
    - Blog posts
    - E-commerce product listings
    - Help and documentation
  - 요청마다 결과가 다른 페이지의 경우 사용이 부적절하다.
- Server-side Rendering
  - 더 느림
  - 항상 최신 데이터가 포함된 페이지를 반환한다.
  - 자주 변경되는 데이터를 채우기 위해 pre-rendering 대신 client-side 를 사용할 수 있다.

## Static Generation with Data using `getStaticProps`

![image](https://nextjs.org/static/images/learn/data-fetching/static-generation-with-data.png)

- Next.js 에서는 page component 를 export 할 때 `getStaticProps`라고 불리는 `async` function 도 export 할 수 있다.
  - `getStaticProps`는 `production` 모드일 때 build time에 수행된다.
  - 해당 함수 내에서 외부 데이터를 fetch 할 수있고 이를 props를 통해 page에 전달할 수 있다.
  - (주의) development mode 에서는 요청이 있을 때마다 `getStaticProps`를 수행한다.

## Example - blog data

- 디렉토리 최상단에 `posts`라는 폴더를 생성한다.
- 해당 폴더 안에 `pre-rendering.md`와 `ssg-ssr.md` 파일을 생성한다.

```markdown
---
title: 'Two Forms of Pre-rendering'
date: '2020-01-01'
---

Next.js has two forms of pre-rendering: **Static Generation** and **Server-side Rendering**. The difference is in **when** it generates the HTML for a page.

- **Static Generation** is the pre-rendering method that generates the HTML at **build time**. The pre-rendered HTML is then _reused_ on each request.
- **Server-side Rendering** is the pre-rendering method that generates the HTML on **each request**.

  Importantly, Next.js lets you **choose** which pre-rendering form to use for each page. You can create a "hybrid" Next.js app by using Static Generation for most pages and using Server-side Rendering for others.
```

```markdown
---
title: 'When to Use Static Generation v.s. Server-side Rendering'
date: '2020-01-02'
---

We recommend using **Static Generation** (with and without data) whenever possible because your page can be built once and served by CDN, which makes it much faster than having a server render the page on every request.

You can use Static Generation for many types of pages, including:

- Marketing pages
- Blog posts
- E-commerce product listings
- Help and documentation

  You should ask yourself: "Can I pre-render this page **ahead** of a user's request?" If the answer is yes, then you should choose Static Generation.

  On the other hand, Static Generation is **not** a good idea if you cannot pre-render a page ahead of a user's request. Maybe your page shows frequently updated data, and the page content changes on every request.

  In that case, you can use **Server-Side Rendering**. It will be slower, but the pre-rendered page will always be up-to-date. Or you can skip pre-rendering and use client-side JavaScript to populate data.
```

- markdown 파일 최 상단에 metadata section 이 존재하는데 이는 `YAML Front Matter` 라고 부르며, `gray-matter` 라이브러리에 의해 parsing 된다.

![image](https://nextjs.org/static/images/learn/data-fetching/index-page.png)

- `gray-matter` 라이브러리 설치 (markdown 내 metadata 를 parse 해주는 라이브러리)
  
```bash
$ npm install gray-matter
```

- file system 으로부터 data를 fetching 해주는 간단한 라이브러리 생성
  - 최상위 디렉토리에 `lib` 폴더 생성(`lib`는 Next.js에서 별도로 지정된 폴더명은 아니므로 임의로 네이밍 가능하다. 일반적인 컨벤션은 `lib` or `utils` 이다.)
  - 해당 폴더 내에 `posts.js` 파일 생성

```javascript
// /lib/posts.js
import fs from 'fs';
import path from 'path';
import matter from 'gray-matter';

const postsDirectory = path.join(process.cwd(), 'posts');

export function getSortedPostsData() {
  // Get file names under /posts
  const fileNames = fs.readdirSync(postsDirectory);
  const allPostsData = fileNames.map((fileName) => {
    // Remove ".md" from file name to get id
    const id = fileName.replace(/\.md$/, '');

    // Read markdown file as string
    const fullPath = path.join(postsDirectory, fileName);
    const fileContents = fs.readFileSync(fullPath, 'utf8');

    // Use gray-matter to parse the post metadata section
    const matterResult = matter(fileContents);

    // Combine the data with the id
    return {
      id,
      ...matterResult.data,
    };
  });
  // Sort posts by date
  return allPostsData.sort(({ date: a }, { date: b }) => {
    if (a < b) {
      return 1;
    } else if (a > b) {
      return -1;
    } else {
      return 0;
    }
  });
}
```

- `pages/index.js` 파일의 `getStaticProps`에서 `getSortedPostsData` 함수를 import 하여 호출해보자.

```javascript
// /pages/index.js
...

import {getSortedPostsData} from '../lib/posts';

export async function getStaticProps() {
  const allPostsData = getSortedPostsData();
  return {
    props: {
      allPostsData,
    },
  };
}

export default function Home({allPostsData}) {
  return (
          <Layout home>
            ...

            {/* Add this <section> tag below the existing <section> tag */}
            <section className={`${utilStyles.headingMd} ${utilStyles.padding1px}`}>
              <h2 className={utilStyles.headingLg}>Blog</h2>
              <ul className={utilStyles.list}>
                {allPostsData.map(({id, date, title}) => (
                        <li className={utilStyles.listItem} key={id}>
                          {title}
                          <br/>
                          {id}
                          <br/>
                          {date}
                        </li>
                ))}
              </ul>
            </section>
          </Layout>
  );
}
```

### 주의

- `getStaticProps` 는 page 내에서만 export 가능하다. (non-page 파일에서는 사용 불가)
- Request Time 에 Fetch Data가 필요한 경우 -> Static Generation 사용 불가. Server-side Rendering 또는 skip pre-rendering 을 사용한다.

## Fetching Data at Request Time

### Server-side Rendering

- request time 에 fetch data가 필요한 경우 Server-side Rendering 사용
- `Server-side Rendering` 사용을 위해 `getServerSideProps` export 가 필요하다.

```javascript
export async function getServerSideProps(context) {
  return {
    props: {
      // props for your component
    },
  };
}
```

- `context`는 request specific parameters 를 포함하고 있다.
- Time to first byte (TTFB) `getStaticProps`에 비해 느리다. (모든 요청마다 결과를 반드시 비교해야 하고 별도의 설정없이는 CDN 에 케싱할 수 없기 때문이다.)

### Client-side Rendering

- data를 pre-render 할 필요 없는 경우, Client-side Rendering 을 사용할 수 있다.
  - 외부 데이터가 필요하지 않는 페이지의 일부는 정적으로 생성해라.(pre-render)
  - page가 로딩될 때, Client에서 외부 데이터를 fetch 해서 남은 부분을 채운다.
- ex) User dashboard에 사용.
  - dashboard는 사용자 특화된 페이지이며 SEO와 연관이 없고 데이터가 자주 업데이트 되므로 pre-render 가 필요 없다.

### SWR([참고](https://swr.vercel.app/ko))

- Next.js 팀에서 만든 data fetching 을 위한 React hook 이다.
- CSR 사용 시 적극 사용을 권장한다.
- 특징
  - Caching
  - Revalidation
  - Focus tracking
  - Refetching on interval

```javascript
import useSWR from 'swr';

function Profile() {
  const { data, error } = useSWR('/api/user', fetch);

  if (error) return <div>failed to load</div>;
  if (!data) return <div>loading...</div>;
  return <div>hello {data.name}!</div>;
}
```

# Dynamic Routes

## Page Path Depends on External Data



<br>

# 참고

- [Vercel - NEXT.JS 공식 문서](https://nextjs.org/learn/foundations/about-nextjs)
