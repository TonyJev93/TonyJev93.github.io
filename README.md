# TonyJev's Blog

- Jekyll Theme : [minimal-mistakes](https://mmistakes.github.io/minimal-mistakes/)

## 실행방법

```bash
$ bundle
$ sudo bundle exec jekyll serve
```

## 꾸미기

- minimal-mistakes A to Z : https://eona1301.github.io/a_to_z/GithubBlog/

## Error

### 현상

> /Users/nhn/.rbenv/versions/3.1.1/lib/ruby/gems/3.1.0/gems/jekyll-4.2.2/lib/jekyll/commands/serve/servlet.rb:3:in `require': cannot load such file -- webrick (LoadError)

### 원인

- ruby 3.x.x 부터 webrick 을 기본으로 포함하지 않음.

### Solution

```bash
bundle add webrick
```
