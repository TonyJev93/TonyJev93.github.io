---
title: "[Study] 기술 부채 - HashMap"
last_modified_at: 2023-07-15T23:00:00+09:00
categories:
    - interview
tags:
    - Study
    - 기술 부채
    - HashMap
    - 자료구조
toc: true
toc_sticky: true
toc_label: "목차"
---

Study : 내가 부족한 기술(HashMap)에 대해 정리한다.
{: .notice--info}

# 구조 및 원리

- 해시 키 : Key 값을 해시함수를 통해 해시코드로 변형, 해시 키는 버킷에 보관됨. 해시 키 만큼 버킷이 생성 됨.
- 버킷 : 데이터를 보관하고 있는 공간. 해시코드 마다 버킷 존재
- 버킷배열 : 초기에 일정 크기로 생성 됨. 필요에 따라 내부적으로 크기를 변경시킴.(해시코드가 많아지면 발생)
- 해시충돌 : 서로 다른 Key 이지만 해시코드가 같은 경우 발생. 키에 해당하는 값은 동일 버킷에 저장되며 LinkedList or Tree(보통 데이터 8개 이상인 경우)를 사용하여 충돌을 처리함.(O(n) vs O(log n))
