---
title: "[Github Blog] minimal mistakes - 파비콘(Favicon) 세팅하기"
last_modified_at: 2022-03-15T12:35:00+09:00
categories:
    - Github Blog
tags:
    - Github Blog
    - minimal mistakes
    - Tip
toc: true
toc_sticky: true
toc_label: "목차"
---

깃허브를 이용한 블로그 만들기 관련 Tips : 파비콘(Favicon) 세팅 방법 
{: .notice--info}

# 파비콘(favicon) 세팅

## 아이콘 찾기

- 아래 사이트에 접속하여 마음에 드는 Favicon 을 다운로드 받는다.
    - [flaticon](https://www.flaticon.com/)
    - [Icooon-mono](https://icooon-mono.com/)
    - [Thenounproject](https://thenounproject.com/)
    - [Iconfinder](https://www.iconfinder.com/)
    - [Freepik](https://www.freepik.com/)
    - [FreeVectors](https://www.freevectors.net/)

## 아이콘 만들기

- [realfavicongenerator](https://realfavicongenerator.net/) 에 접속한다.
<br>
<br>
![image](https://user-images.githubusercontent.com/53864640/158215212-6599a18c-ae80-47ab-aa13-060b141e69d1.png)

- `Select Your Favicon image`버튼을 클릭 후 사용할 favicon 파일을 선택하여 업로드 한다.

<br>

![image](https://user-images.githubusercontent.com/53864640/158215651-151e9060-e379-4a91-838c-9e435cae1f0b.png)

- 업로드 후 화면 제일 하단에 내려가 `Generate your Favicons and HTML code` 버튼을 클릭한다.

<br>

![image](https://user-images.githubusercontent.com/53864640/158210875-7d13cf8f-a02c-4444-9a59-7b4ee5af0f0c.png)

- `Favicon package` 버튼을 클릭하여 `zip` 파일을 다운받는다.

## 적용하기

- 다운받은 `zip` 파일을 압축 해제한 이후 내부에 있는 파일들을 전부 `assets/images/logos`하위로 복사한다.
- **custom.html** 파일을 아래와 같이 수정 (해당 코드는 favicon 생성 직후 사이트에 나타나는 소스이다.)

```html
<link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png">
<link rel="icon" type="image/png" sizes="32x32" href="/favicon-32x32.png">
<link rel="icon" type="image/png" sizes="16x16" href="/favicon-16x16.png">
<link rel="manifest" href="/site.webmanifest">
<link rel="mask-icon" href="/safari-pinned-tab.svg" color="#5bbad5">
<meta name="msapplication-TileColor" content="#da532c">
<meta name="theme-color" content="#ffffff">
```

- 이후, `href=`에 해당하는 파일 경로만 각자 저장한 파일 위치에 맞게 수정하면 된다.

# 참고

- [[Github Blog] minimal mistakes - 파비콘(Favicon) 세팅하기](https://eona1301.github.io/github_blog/GithubBlog-Favicon/)