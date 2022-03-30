---
title: "[Design Pattern] 기타 - 마커 인터페이스 패턴 (Marker Interface Pattern)"
last_modified_at: 2022-03-17T02:33:00+09:00
categories:
    - Design Pattern
tags:
    - Design Pattern
    - 마커 인터페이스 패턴
toc: true
toc_sticky: true
toc_label: "목차"
---

디자인 패턴 : 마커 인터페이스 패턴에 대해 알아보자.
{: .notice--info}



# 마커 인터페이스 패턴

직렬화때 사용하는 `Serializable`과 클론할때 사용하는 `Cloneable`의 소스코드를 타고 들어가보면 비어있는 인터페이스만 존재하는 것을 볼 수 있다. 처음에는 아무 생각이 없었지만 디자인 패턴 스터디를 하던 중 이것이 `마커 인터페이스 패턴`이라는 말을 듣고 궁금해져서 관심을 갖게 되었다. `EventListener`도 해당 패턴에 속한다고 한다.

해당 패턴에 대한 이름은 GoF 패턴에도 명시가 되어있지 않아 생소하기도 했고, 비어있는 인터페이스가 과연 무슨 의미가 있을까 싶기도 하다.

## 마커 인터페이스

JAVA의 `마커 인터페이스`는 `일반 인터페이스`와 동일하다. 다만, 아무 메소드도 선언되어있지 않은 인터페이스일 뿐이다.

```java
public interface SampleObject{
    
}
```

정말 단순하게 인터페이스만 존재하고 메소드가 없어 사용처를 알 수 없다.

숨은 기능이 있을 것이라고 생각이 들지만 실상은 대부분 **타입 체크**를 하기 위해 사용된다.

## Serializable

`Serializable`을 예로 들어보자, `Serializable` 인터페이스는 아래와 같이 되어있다.

```java
public interface Serializable{
    
}
```

직렬화를 하기 위해서는 위의 Serializable 인터페이스를 무조건 implements 해야 한다. 그렇지 않은 클래스의 경우 직렬화를 할 수 없다.(**NotSerializableException** 발생)

직렬화가 필요한 클래스의 경우 Serializable 를 구현하기만 하면 된다. 구현할 메소드가 없기 때문에 구현이라고 할 것도 없이 **선언만 해주면 끝**이다.

실제 직렬화를 수행하는 함수의 구현을 보면 

```java

// ObjectOutputStream.class -> writeObject() 중에서

if (obj instanceof String){
        ...
}else if (obj instanceof Serializable) {
  writeOrdinaryObject(...);
} else{
  throw new NotSerializableException(...);
}
```

와 같이 구현되어있는 것을 볼 수 있다. 정말 Serializable가 선언 되어있는지 정도만 확인할 뿐 그 이상 사용되지 않는다.

그래서 **마커 인터페이스**라고 불린다.

## vs 마커 어노테이션

그렇다면 이러한 동작은 어노테이션으로도 구현이 가능하다. 비어있는 어노테이션을 생성한 뒤 클래스가 해당 어노테이션을 가지고 있는지 확인만 하면 된다.

그럼 인터페이스와의 차이점은 무엇일까?

바로, **컴파일 시점**에 발견할 수 있냐 없냐의 차이이다. **마커 인터페이스**는 컴파일 시점에 체크할 수 있다. 그리고 특정 인터페이스를 구현한 클래스에만 적용해야 한다고 하면 그 특정 인터페이스를 `마커 인터페이스`가 상속하도록 하면 된다.

반면, **마커 어노테이션**은 언제 쓸까?

마커 어노테이션은 처음에는 마커 인터페이스와 같은 용도로 사용하다가 추 후에 관리하고 싶은 정보들이 생기게 되면 어노테이션에 `default` 값을 갖도록 하기만 하면 된다. 즉, **유연하고 확장성을 갖게될 염려가 있는 경우 사용**하면 좋을 것 같다.
이는 **마커 인터페이스**에 추가 메소드가 생기는 순관 하위 호환성이 깨진다는 단점과 비교할 수 있다.


[⬅️ 디자인패턴 목차보기](/design%20pattern/design-pattern-overview/)