---
title: "[Study] 기술 부채 - ConcurrentHashMap"
last_modified_at: 2023-07-15T23:00:00+09:00
categories:
    - interview
tags:
    - Study
    - 기술 부채
    - ConcurrentHashMap
toc: true
toc_sticky: true
toc_label: "목차"
---

Study : 내가 부족한 기술(ConcurrentHashMap)에 대해 정리한다.
{: .notice--info}


- [참고 - [Java] ConcurrentHashMap 이란 무엇일까?](https://devlog-wjdrbs96.tistory.com/269)
- [참고 - [Java] ConcurrentHashMap는 어떻게 Thread-safe 한가?](https://velog.io/@alsgus92/ConcurrentHashMap%EC%9D%98-Thread-safe-%EC%9B%90%EB%A6%AC)
- [참고- [Java] ConcurrentHashMap에 대해 알아보자](https://yeoonjae.tistory.com/entry/Java-ConcurrentHashMap%EC%97%90-%EB%8C%80%ED%95%B4-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90%EC%9E%91%EC%84%B1%EC%A4%91)

---
- 
- **HashTable, HashMap, ConcurrentHashMap** 차이점 비교

# Hashtable 클래스

```java
public class Hashtable<K,V>
    extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable {

    public synchronized int size() { }

    @SuppressWarnings("unchecked")
    public synchronized V get(Object key) { }

    public synchronized V put(K key, V value) { }
}
```

- 메서드 마다 `synchronized` 키워드가 존재
  - Thread-Safe
  - 병목 현상 발생 -> 느림
  - Collection Framework 나오기 이전 부터 존재하던 클래스로 최근에 잘 사용 안함

# HashMap 클래스

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {

    public V get(Object key) {}
    public V put(K key, V value) {}
}
```

- `synchronize` 키워드가 없음
  - Map 인터페이스 구현 중 성능이 제일 좋음
  - Thread-Safe 하지는 않음
  - 멀티 쓰레드 환경에서 `Hashtable`의 대체가 될 수 없음

그럼 `Hashtable` 보다 빠르고 Thread-Safe한 클래스는 없을까?

# ConcurrentHashMap 클래스

- JDK 1.5 에서 검색, 업데이트 동시성 성능 향상을 위해 등장

```java
public class ConcurrentHashMap<K, V> extends AbstractMap<K, V>
        implements ConcurrentMap<K, V>, Serializable {

    public V get(Object key) {
    }

    public boolean containsKey(Object key) {
    }

    public V put(K key, V value) {
        // put()메소드 호출 시 putVal()메소드를 호출한다. 
        return putVal(key, value, false);
    }

    /** Implementation for put and putIfAbsent */
    final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;

        // table은 가변테이블로 내부적으로 관리한다. 
        for (Node<K, V>[] tab = table; ; ) {
            Node<K, V> f;
            int n, i, fh;

            if (tab == null || (n = tab.length) == 0)
                tab = initTable();

                // 새로운 노드를 삽입하기 위해, 해당 bucket 값을 가져와(tabAt) 비어 있는지 확인한다.(== null)
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {

                // 위의 조건을 일치한다면 
                // node를 담고있는 volatile 변수에 접근하여 기대값(null)을 비교하여(casTabAt) 
                // 같으면(null이면) 새로운 Node를 생성하고 같지않다면 다시 반복문으로 돌아간다. 
                if (casTabAt(tab, i, null,
                        new Node<K, V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin 
                // 새로운 Node를 생성할 땐 lock을 걸지 않는다. 
            } else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);

            else {
                V oldVal = null;

                // 이미 노드가 존재하는 경우 
                // synchronized 블록을 사용하여 lock을 건다. 
                synchronized (f) {

                    if (tabAt(tab, i) == f) {    // f는 비어있지 않은 bucket
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K, V> e = f; ; ++binCount) {
                                K ek;

                                // 새로운 노드로 교체
                                if (e.hash == hash &&
                                        ((ek = e.key) == key ||
                                                (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K, V> pred = e;

                                // Separate chaining에 노드 추가
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K, V>(hash, key,
                                            value, null);
                                    break;
                                }
                            }
                        }

                        // 트리에 노드 추가
                        else if (f instanceof TreeBin) {
                            Node<K, V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K, V>) f).putTreeVal(hash, key,
                                    value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }
}
```

- `synchronized` 키워드가 메서드 전체에 붙어 있지는 않음
- get()에는 아에 없고, put() 중간에 존재
  - 읽을 때 여러 쓰레드가 읽을 수 있음
  - 쓰기 작업은 특정 쎄그먼트 or 버킷에 Lock 사용

**put 파헤치기**
 
1. 노드 없을 때 
   - tabAt() 통해 버킷 가져옴 -> 버킷 비어있는지 확인
   - 버킷 비어있으면 casTabAt() 통해 노드 가져옴 -> 노드 생성
   - tabAt(), casTabAt() 통해서 버킷 가져옴
     - 메모리에 직접 접근하는 [UnSafe](https://rangken.github.io/blog/2015/sun.misc.unSafe/) 사용
   - cas 알고리즘으로 무한루프 수행
2. 노드 존재하는 경우
   - synchronized 사용하여 하나의 쓰레드에서만 접근 가능하도록 함
     - 서로 다른 쓰레드가 같은 버킷에 접근시에만 블락 잡힘
   - 버킷 크기가 `TREEIFY_THRESHOLD = 8` 보다 큰 경우 Linked List 대신 Tree 로 운용
     - treeifyBin()을 통해 Tree 구조로 변경됨.
       - treeifyBin() 내부 구현 보면 HashMap 크기가 `MIN_TREEIFY_CAPACITY = 64` 미만인 경우 계속 HashMap이 유지 됨.(즉, 버킷 8개 이상이더라도 HashMap 크기가 64 이상이어야 Tree 구조)
   - 버킷 크기가 `UNTREEIFY_THRESHOLD = 6` 보다 작으면 Tree 대신 다시 Linked List로 운용


기타> ConcurrentHashMap의 entrySet(), keySet(), values()이 존재 > entrySet() 에서 remove()가 발생하면 CocurrentHashMap에 정의된 remove()가 수행되므로 remove()로 인한 동기화 문제는 발생하지 않음.   


```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {

    private static final int DEFAULT_CAPACITY = 16;

    // 동시에 업데이트를 수행하는 쓰레드 수
    private static final int DEFAULT_CONCURRENCY_LEVEL = 16;
}
```

- `DEFAULT_CAPACITY` : 버킷 수
- `DEFAULT_CONCURRENCY_LEVEL` : 동시 작업 가능한 쓰레드 수
- `DEFAULT_CAPACITY == DEFAULT_CONCURRENCY_LEVEL` 이유 : 버킷 단위 락을 잡기 때문에 버킷이 아니면 락을 기다릴 필요 없음



**요약**

- ConcurrentHashMap의 구조
  - 세그먼트
    - 여러개 존재
    - 쓰레드마다 각각 동시에 접근 가능
    - 세그먼트 단위로 락이 잡힘
  - 버킷
    - 세그먼트 내에서 여러개
    - 실제 데이터를 담고 있는 공간
    - ConcurrentHashMap크기에 따라 동적으로 조정 
    - 각 버킷은 HashMap의 역할을 수행
  - 세그먼트는 배열, 버킷은 세그먼트 배열 내에서 링크드 리스트로 존재
    - 세그먼트 배열 고정(생성자 또는 CPU 코어 수에 따라 자동 결정)
    - 버킷 : 동일한 해시코드를 가진 키들은 동일한 버킷에 저장됨, 해시 충돌 시 동일한 버킷 내 링크드 리스트로 연결되어 데이터 저장
        - 내부적으로 버킷은 배열로 구성됨
        - 버킷 내 데이터는 링크드 리스트 또는 트리(해시 충돌 적을 때 사용 됨 JAVA 8 이상)로 관리 됨

1. 반복문을 실행한다.
2. 새로운 노드를 삽입하기 위해 bucket값을 가져와 비어있는지(null값) 확인한다.
3. 비어있다면 Node를 담고있는 volatile 변수에 접근하여 기댓갑(null)과 일치하는지 확인한다.
   1. 일치한다면 Node를 삽입한다.(lock은 걸지 않는다.)
   2. 일치하지 않는다면 다시 1번으로 간다. (재시도)
4. 비어있지 않다면 synchronized를 사용하여 lock을 건다. 이외의 내부적인 구현은 HashMap과 동일하다.

읽기 보다는 쓰기의 성능이 중요할 때 사용한다.

읽기는 동기화가 안되기 때문에 가장 최근에 업데이트된 작업의 결과를 반영한다.

## HashMap와 다른점

- key, value에 null 허용하지 않음