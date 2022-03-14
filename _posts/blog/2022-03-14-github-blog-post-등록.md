---
title: "[Github Blog] Post 등록"
last_modified_at: 2022-03-15T12:35:00+09:00
categories:
    - Github Blog
tags:
    - Github Blog
    - Tip
toc: true
toc_sticky: true
toc_label: "목차"
---

깃허브를 이용한 블로그 만들기 관련 Tips
{: .notice--info}

# Post 등록

## 양식

- 파일명 : `YYYY_MM_DD-title.md`
- 내부양식

    | name              | mean                                |
    |-------------------|-------------------------------------|
    | title             | 실제 화면에 보이는 제목                         |
    | categories            | 포스터의 소속 카테고리
    | tags                 | 포스터 태그                                |
    | toc                  | table of content, 우측 상단 목차 노출 여부      |
    | toc_label            | toc 이름                              |
    | toc_icon             | toc 아이콘                             |
    | toc_sticky           | toc 스크롤 이동시 고정노출 여부(default : true) |
    | last_modified_at     | 게시글 마지막 수정일, 포스터에는 년월일만 표기됨         |

## 정렬

### 문자 정렬

- `{: .text-direction}`

왼쪽 정렬(Default)
{: .text-left}

```md
왼쪽 정렬 (Default)
{: .text-left}
```

중앙 정렬 
{: .text-center}

```md
중앙 정렬
{: .text-center}
```

오른쪽 정렬 
{: .text-right}

```md
오른쪽 정렬
{: .text-right}
```

### 이미지 정렬

<br>
<br>

- `{: .align-direction}`

<br>
<br>

![image](https://user-images.githubusercontent.com/45550607/102208312-9b284180-3f12-11eb-8467-7b5ea1779ac7.png)
{: .align-left}
왼쪽 정렬(Default)

<br>
<br>
<br>

```md
![image](https://user-images.githubusercontent.com/45550607/102208312-9b284180-3f12-11eb-8467-7b5ea1779ac7.png){: .align-left}
왼쪽 정렬(Default)
```

<br>

![image](https://user-images.githubusercontent.com/45550607/102208312-9b284180-3f12-11eb-8467-7b5ea1779ac7.png){: .align-center}
중앙 정렬 
{: .text-center}

```md
![image](https://user-images.githubusercontent.com/45550607/102208312-9b284180-3f12-11eb-8467-7b5ea1779ac7.png){: .align-center}
중앙 정렬
{: .text-center}
```

<br>

![image](https://user-images.githubusercontent.com/45550607/102208312-9b284180-3f12-11eb-8467-7b5ea1779ac7.png){: .align-right}
오른쪽 정렬
{: .text-right}

<br>
<br>
<br>

```
![image](https://user-images.githubusercontent.com/45550607/102208312-9b284180-3f12-11eb-8467-7b5ea1779ac7.png){: .align-right}
오른쪽 정렬
{: .text-right}
```

## 문자 박스

| Notice Type |       Class        |
| :---------: | :----------------: |
|   Default   |     `.notice`      |
|   Primary   | `.notice--primary` |
|    Info     |  `.notice--info`   |
|   Warning   | `.notice--warning` |
|   Success   | `.notice--success` |
|   Danger    | `.notice--danger`  |

**Default 문자 박스입니다.** 문자 박스는 색이 달라 원하는 정보를 블로그 방문자에게 보여주기 좋은 것 같습니다! Notice Type의 뜻에 접근하셔도 되고, 원하는 색상에 따라 사용해주셔도 될 것 같습니다.
{: .notice}

**Primary 문자 박스입니다.** 문자 박스는 색이 달라 원하는 정보를 블로그 방문자에게 보여주기 좋은 것 같습니다! Notice Type의 뜻에 접근하셔도 되고, 원하는 색상에 따라 사용해주셔도 될 것 같습니다.
{: .notice--primary}

**Info 문자 박스입니다.** 문자 박스는 색이 달라 원하는 정보를 블로그 방문자에게 보여주기 좋은 것 같습니다! Notice Type의 뜻에 접근하셔도 되고, 원하는 색상에 따라 사용해주셔도 될 것 같습니다.
{: .notice--info}

**Warning 문자 박스입니다.** 문자 박스는 색이 달라 원하는 정보를 블로그 방문자에게 보여주기 좋은 것 같습니다! Notice Type의 뜻에 접근하셔도 되고, 원하는 색상에 따라 사용해주셔도 될 것 같습니다.
{: .notice--warning}

**Success 문자 박스입니다.** 문자 박스는 색이 달라 원하는 정보를 블로그 방문자에게 보여주기 좋은 것 같습니다! Notice Type의 뜻에 접근하셔도 되고, 원하는 색상에 따라 사용해주셔도 될 것 같습니다.
{: .notice--success}

**Danger 문자 박스입니다.** 문자 박스는 색이 달라 원하는 정보를 블로그 방문자에게 보여주기 좋은 것 같습니다! Notice Type의 뜻에 접근하셔도 되고, 원하는 색상에 따라 사용해주셔도 될 것 같습니다.
{: .notice--danger}


# 참고

- [[Github Blog] minimal mistakes - 포스팅 글 써보기](https://eona1301.github.io/github_blog/GithubBlog-Posting/)