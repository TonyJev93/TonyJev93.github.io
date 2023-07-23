---
title: "[Study] 기술 부채 - Spring"
last_modified_at: 2023-07-15T23:00:00+09:00
categories:
    - interview
tags:
    - Study
    - 기술 부채
    - Spring
toc: true
toc_sticky: true
toc_label: "목차"
---

Study : 내가 부족한 기술(Spring)에 대해 정리한다.
{: .notice--info}

# Singleton

스프링에서 빈을 등록할 때 사용되는 패턴

한 번 생성 해둔 객체를 에플리케이션 종료시 까지 사용하는 패턴

- [생성 방법](https://www.techiedelight.com/ko/implement-singleton-pattern-in-java/)
  1. 기본 생성자 감추기 
    - private SingleTon(){};
    - static final INSTANCE = new SingleTon();
    - public static SingleTon getInstance() {return INSTANCE;}
  2. Lazy Loading
    - private SingleTon(){if INSTANCE != null -> throw exception;};
    - public static SingleTon getInstance() {if INSTANCE == null -> new SingleTon; return INSTANCE;}
  3. Syncronized
    - public static SingleTon getInstance() {if synchronized(SingleTon.class) {INSTANCE == null -> new SingleTon;} return INSTANCE;}
  4. Serializable
     - private Object readResolver() { return INSTANCE; }

or Enum 사용

## 단점

- 생성자가 private 으로 인한 상속 불가(다형성 X)
- 아무나 접근하여 자유롭게 수정 및 공유할 수 있는 전역의 상태를 띔(객체지향 측면에서 안좋음)


# [IoC, DI](https://velog.io/@ohzzi/Spring-DIIoC-IoC-DI-%EA%B7%B8%EA%B2%8C-%EB%AD%94%EB%8D%B0)