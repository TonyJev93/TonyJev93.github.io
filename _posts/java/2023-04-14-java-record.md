---
title: "[Java] Record 클래스"
last_modified_at: 2023-04-15T23:50:00+09:00
categories:
    - Java
tags:
    - Java
    - Record
toc: true
toc_sticky: true
toc_label: "목차"
---

JAVA : Record 에 대해 파해쳐 보자.
{: .notice--info}

# Record

- Java 16 새롭게 도입
- 데이터를 저장하는 클래스를 더욱 간결하고 명확하게 정의할 수 있도록 하는 기능

## 특징

### Immutable 클래스

- Record 클래스는 불변 클래스로 생성자에서 한 번 값이 할당되면 변경 불가능
- 데이터를 보호하고 안전한 코드 작성을 위함

### 코드의 간결성 

- 자동으로 생성자, equals, hashCode, toString 등을 생성해줌
- 가독성 향상 및 반복 작업 제거

### 값 타입(Value Type)

- 값 타입으로 사용 가능
- 불변성 및 캡슐화 보장

값 타입
: 객체의 상태를 나타내는 것이 아닌 객체의 값 자체를 나타내는 것

### 패턴 매칭

- 패턴 매칭 기능과 함께 사용 시 강력한 기능 발휘

패턴 매칭
: Java 16 새롭게 도입
: 객체의 상태를 검사하고 이를 기반으로 실행 흐름을 변경하는 기능

```java
public class Test {
    public void test() {
        // instanceof 연산자와 캐스팅을 사용한 코드
        if (shape instanceof Rectangle) {
            Rectangle rectangle = (Rectangle) shape;
            if (rectangle.width() > rectangle.height()) {
                System.out.println("This is a wide rectangle");
            } else {
                System.out.println("This is a tall rectangle");
            }
        } else if (shape instanceof Circle) {
            System.out.println("This is a circle");
        } else {
            System.out.println("This is not a valid shape");
        }

        // 패턴 매칭을 사용하여 간결하게 작성한 코드
        if (shape instanceof Rectangle r && r.width() > r.height()) {
            System.out.println("This is a wide rectangle");
        } else if (shape instanceof Rectangle r && r.width() < r.height()) {
            System.out.println("This is a tall rectangle");
        } else if (shape instanceof Circle) {
            System.out.println("This is a circle");
        } else {
            System.out.println("This is not a valid shape");
        }
    }
}
``` 


### Serializable 인터페이스 구현

- Serializable 인터페이스 구현 가능
  - 객체를 직렬화하여 저장 및 전송 가능

    