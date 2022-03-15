---
title: "[Github Blog] minimal mistakes - 구글 검색창 노출"
last_modified_at: 2022-03-16T01:46:00+09:00
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

깃허브를 이용한 블로그 만들기 관련 Tips : 구글 Analytics를 통한 방문자 통계 등록
{: .notice--info}

# 구글 검색창 노출

## Google search 접속 및 가입

[Google search Console](https://search.google.com/search-console/about) 에 접속한다.

![image](https://user-images.githubusercontent.com/53864640/158446457-521ea175-42c9-447c-82af-6ab2c2b754c7.png){: .align-center}

`시작하기` 버튼을 클릭한다.

![image](https://user-images.githubusercontent.com/53864640/158446649-b181533f-0b2e-401a-9077-9cf0dfbe3d48.png){: .align-center}

개인 Domain이 있지 않다면, 우측 `URL 접두어`를 통해 블로그 URL을 입력한다.

## 소유권 확인

![image](https://user-images.githubusercontent.com/53864640/158446968-b60e9cae-bcd5-4630-921c-9365d1fa5a59.png){: .align-center}

위와 같이 본인 사이트가 맞는지 확인하는 절차가 진행된다. `html` 파일을 다운로드 받은 후 `_config.xml` 파일과 동일한 경로에 위치시키면 된다.

<img width="507" alt="image" src="https://user-images.githubusercontent.com/53864640/158447500-e73808b7-b01b-4fae-b9fa-b43364966ae0.png">{: .align-center}

정상적으로 업로드가 되었는지 확인하기 위해서 `local`환경에서 위와 같이 주소창에 `localhost:4000/다운받은_파일명.html`을 입력해본다. 위와 같이 나온다면 정상적으로 업로드한 것이다.

## 크롤링 허용

### sitemap.xml 파일 생성

```
---
layout: null
---

<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.sitemaps.org/schemas/sitemap/0.9 http://www.sitemaps.org/schemas/sitemap/0.9/sitemap.xsd"
        xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
    {% for post in site.posts %}
    <url>
        <loc>{{ site.url }}{{ post.url }}</loc>
        {% if post.lastmod == null %}
        <lastmod>{{ post.date | date_to_xmlschema }}</lastmod>
        {% else %}
        <lastmod>{{ post.lastmod | date_to_xmlschema }}</lastmod>
        {% endif %}

        {% if post.sitemap.changefreq == null %}
        <changefreq>weekly</changefreq>
        {% else %}
        <changefreq>{{ post.sitemap.changefreq }}</changefreq>
        {% endif %}

        {% if post.sitemap.priority == null %}
        <priority>0.5</priority>
        {% else %}
        <priority>{{ post.sitemap.priority }}</priority>
        {% endif %}

    </url>
    {% endfor %}
</urlset>
```

`sitemap.xml 파일`을 `google HTML 파일`과 동일한 위치(Root 위치)에 생성하면 된다. 이를 통해, Google 크롤러가 주기적으로 블로그의 URL을 체크할 수 있게 된다.

<img width="1400" alt="image" src="https://user-images.githubusercontent.com/53864640/158449304-739be8c7-288a-44f0-ba10-275f12e5d877.png">{: .align-center}

[http://127.0.0.1:4000/sitemap.xml](http://127.0.0.1:4000/sitemap.xml)에 접속하였을 때 위와 같이 나타난다면 정상적으로 처리된 것이다.

## robots.txt 만들기

```text
User-agent: *
Allow: /

Sitemap: https://tonyjev93.github.io/sitemap.xml
```

이제 접근하는 크롤러는 robots.txt를 보고 접근하고자 하는 sitemap의 위치를 확인하고, 제한을 확인하여 본래의 웹사이트로 가져가게 됩니다. Allow에 본인이 원하는 정보만 입력하거나 제한을 두고싶은 내용을 입력하면 크롤러가 확인하여 진행해준다.




# 참고

- [[Github Blog] minimal mistakes - 검색창 노출시키기](https://eona1301.github.io/github_blog/GithubBlog-Search/)