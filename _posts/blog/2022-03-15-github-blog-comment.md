---
title: "[Github Blog] minimal mistakes - 댓글 기능 추가"
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

깃허브를 이용한 블로그 만들기 관련 Tips : 댓글 기능구현하는 방법
{: .notice--info}

# 댓글 기능 추가

## DISQUS 가입

[DISQUS](https://disqus.com/) 에 접속하여 회원가입을 한다.


![image](https://user-images.githubusercontent.com/53864640/158429533-deb657e9-77c0-40d7-9707-1e4b7e60c715.png){: .align-center}

`I want to install Disqus on my site` 를 선택한다.

![image](https://user-images.githubusercontent.com/53864640/158431764-c9289a3d-4bba-4c54-b885-2e644fd215db.png){: .align-center}

`Website Name` 에 본인의 블로그 주소를 입력한다. 이 때, 하단에 보이는 `blogname-github-io.disqus.com`에서 `blogname-github-io` 가 `shortname` 이 된다. 

![image](https://user-images.githubusercontent.com/53864640/158433281-43401ce6-7872-4bda-bf5d-c21c64a301f4.png){: .align-center}

`Create Site` 후 가격정책을 고르는데, 무료 사용을 위해 하단에 `Basic` 을 선택한다. 이후 웹사이트 종류를 Jekyll 을 선택한다.

![image](https://user-images.githubusercontent.com/53864640/158434336-d3650bb9-1d86-465e-afa6-5a0c2ef71b3a.png){: .align-center}

## _config.yml 수정

```yaml
comments:
  provider               : "disqus"
  disqus:
    shortname            : "tonyjev93-github-io"

...

# Defaults
defaults:
  # _posts
  - scope:
      ...
    values:
      ...
      comments: true # <== comment 를 true 로 변경
      ...
```

## 반영

local 에서는 comment 적용 여부를 확인 할 수 없다. 그러므로 git push 이후 실제 블로그 내에서 확인이 필요하다. 반영이 약 5~10분 걸리므로 배포 후 대기해보자.

# 참고

- [[Github Blog] minimal mistakes - 댓글 기능 추가하기](https://eona1301.github.io/github_blog/GithubBlog-Comment/)