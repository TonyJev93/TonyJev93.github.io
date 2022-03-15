---
title: "[Github Blog] minimal mistakes - 방문자 통계 등록"
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

# 구글 통계 등록하기

[Google Analytics](https://analytics.google.com/analytics/web/provision/#/provision) 에 접속한다.

![image](https://user-images.githubusercontent.com/53864640/158440277-4ca55588-1bf1-44ce-9eed-6dfae0ffa7f5.png){: .align-center}

`측정 시작` 버튼을 클릭한다.

![image](https://user-images.githubusercontent.com/53864640/158440465-9536d80b-7611-45ba-bad1-511f73ffe0db.png){: .align-center}

`계정 이름`은 통계에 영향을 주지 않기 때문에 자유롭게 작성하길 바란다.

![image](https://user-images.githubusercontent.com/53864640/158441345-9fdc7472-0fda-4e05-8402-ef6930d09b89.png){: .align-center}

`속성 이름`은 블로그 주소를 작성해주고, `보고 시간대, 통화`는 대한민국으로 통일한다.

속성 설정까지 완료 후 통계할 웹에 대한 간단한 설문조사를 진행하고, `웹 스트림 추가`를 진행 한다.

![image](https://user-images.githubusercontent.com/53864640/158442204-f9b864cc-35c1-4ebe-b376-6c6eebde6d1a.png){: .align-center}

스트림 추가가 완료되면 위 화면이 나타나는데 `측정 ID`가 `tracking_id` 이다. 해당 ID를 `_config.yml` 파일에 세팅해주자.

## _config.yml 수정

```yaml
# Analytics
analytics:
  provider               : "google-gtag" # false (default), "google", "google-universal", "google-gtag", "custom"
  google:
    tracking_id          : "G-B9YPZJL07H"
    anonymize_ip         : # true, false (default)
```

## 반영

모든 설정이 완료되었으므로, `git push`를 통해 블로그에 반영하고 확인하면 된다.

# 참고

- [[Github Blog] minimal mistakes - 방문자 통계(Analytics)하기](https://eona1301.github.io/github_blog/GithubBlog-Analytics/)