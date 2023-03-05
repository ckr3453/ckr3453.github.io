---
title: "이펙티브자바 - 아이템15) 클래스와 멤버의 접근 권한을 최소화하라."
categories: 
    - book
date: 2023-03-03
last_modified_at: 2023-03-03
toc: true
toc_sticky: true
excerpt: "캡슐화 하기"
---

## 개요

객체지향 프로그래밍에서 정보 은닉 혹은 캡슐화라고 불리는 개념은 많은 장점을 가지고 있다.

- 시스템 개발 속도를 높인다.
  - 컴포넌트간 구현 내용을 알 필요가 없다.
  - 즉, 여러 컴포넌트를 병렬로 개발할 수 있다.
- 시스템 관리 비용을 낮춘다.
  - 컴포넌트가 작게 나뉘어져 있어서 디버깅 및 교체가 용이하다.
- 성능 최적화에 도움을 준다.
  - 다른 컴포넌트에 영향을 주지않고 특정 컴포넌트를 최적화하기 쉬운 구조가 된다. 
- 소프트웨어 재사용성을 높인다.
  - 외부 컴포넌트에 종속되지 않기 때문에 재사용성이 높다. (독자적 동작 가능)
- 큰 시스템을 제작하는 난이도를 낮춰준다.
  - 개발 단계에서 컴포넌트 별 검증이 용이하다.

자바는 접근 제한자와 같이 캡슐화를 위한 다양한 장치를 제공한다.

이 접근 제한자를 제대로 활용하여 **모든 클래스와 멤버의 접근성을 가능한 좁히는 것**이 캡슐화의 핵심이다.

접근제한자의 범위는 다음과 같다. 

`public` > `protected` > `package-private` > `private`

- `public`
  - 모든곳에서 접근가능하다.
- `protected`
  - `package-private`의 접근 범위를 포함하고 이 멤버를 선언한 클래스의 하위클래스에서도 접근할 수 있다.
- `package-private` (default)
  - 접근제한자를 명시하지 않을때 적용된다. (단, 인터페이스의 멤버는 `public` 적용)
  - 멤버가 소속된 패키지 안의 모든 클래스에 접근 가능하다.
- `private`
  - 멤버를 선언한 가장 바깥의 클래스에서만 접근할 수 있다.

## 접근성을 최소화하여 컴포넌트 설계하기

- 공개 api가 아니면 항상 `pacakage-private`으로 설정하자
  - 공개 api가 된다면 코드의 하위 호환을 위해 영원히 관리해줘야하며 쉽게 수정하지 못한다.

    ```java
    class Test {
      // 이하 생략
    }
    ```

- `private static` 중첩 클래스 사용하기
  - 한 클래스에서만 사용하는 `pacakage-private` 클래스나 인터페이스가 있다면?
  - `public` 대신 해당 클래스 내부에 `private static`으로 구현하여 해당 클래스만 접근 가능하도록 설계하자.

    ```java
    class Member {

      private static class PasswordPolicy {
          // Member 클래스만 접근 가능한 내부 클래스
      }
    }
    ```

- 공개 api를 제외한 모든 멤버는 `private`으로 설계하자.
  - 동일 패키지 내 다른 클래스가 접근해야되는 멤버에 한해서만 `pacakage-private`으로 풀어주자.
  - 단, 이로 인하여 컴포넌트를 더 분해해야 할지에 대해서는 고민해봐야 한다.

    ```java
    class Member {

      private int id;
      private String name;
      private String age;
      String corpCode;    // 동일 패키지 내 다른 클래스가 접근 가능

      // 이하 생략
    }
    ```

- 공개 api에서 `protected` 멤버의 수는 적을수록 좋다.
  - 멤버의 접근권한이 `protected`가 되는순간 공개 api의 대상이 되므로 영원히 지원되어야한다.
  - 접근 범위로 인하여 내부 동작 방식을 api 문서에 명시해야 할 수도 있다.

- 단순히 테스트를 위해 클래스, 인터페이스, 멤버의 접근 범위를 넓히지 말자.
  - `private` 멤버를 `package-private` 까지는 허용할 수 있다. (그이상은 안됨)
  - 테스트 코드를 동일 패키지 내에 두어서 테스트를 진행하자.

- `public` 클래스의 인스턴스 필드는 되도록 `public`이 아니어야한다.
  - 필드에 담을 수 있는 값을 제한할 수 없게된다.
  - 즉, 불변을 보장할 수 없으며 `thread-safe` 하지 않다.
  - 단, 클래스의 추상 개념을 완성하는 데 꼭 필요한 구성요소의 상수라면 `public static final`로 공개해도 된다.
    - 관례상 대문자 + _(언더바) 조합으로 필드명을 정한다.
    - 반드시 기본 타입 값 혹은 불변 객체를 참조해야 한다.
    - 가변 객체를 참조하게 되면 (ex. 배열) 변경이 가능해지니 주의해야한다.
    - 해당 필드를 반환하는 접근자 메서드는 두면 안된다.

      ```java
      class Stack {
        public static final Object[] elements;   // 접근, 변경가능
        public int size = 0;                     // 접근, 변경가능

        // 이하 생략
      }
      ```

      ```java
      public class Main {
          public static void main(String[] args) {
            Stack.elements[0] = 3;
            Stack.elements[1] = 24;
        }
      }
      ```

    - 해결방법1. `public` 불변 `list` 추가하기

      ```java
      class Stack {
        // private으로 변경
        private static final Object[] ELEMENTS;   

        // 대신 public 불변 list를 추가
        public static final List<Object> list = 
          Collections.unmodifiableList(Arrays.asList(ELEMENTS));

        // 이하 생략
      }
      ```

    - 해결방법2. 복사본 반환하기

      ```java
      class Stack {
        // private으로 변경
        private static final Object[] ELEMENTS;   
        
        // 대신 배열의 복사본을 반환 (방어적 복사)
        public static final Object[] values() {
            return ELEMENTS.clone();
        }

        // 이하 생략
      }
      ```

## 모듈 시스템

자바 9부터 모듈 시스템이라는 개념이 도입되면서 모듈 단위의 두가지 암묵적 접근 수준(모듈 내에서의 `public`, `protected`)이 추가됐다.

클래스들의 묶음 -> 패키지이고, 패키지의 묶음 -> 모듈이다.

모듈에 속하는 패키지 중 공개할 패키지들을 (관례상 `module-info.java` 파일에) 선언할 수 있다.

이를 통해 `protected` 혹은 `public` 으로 공개했어도 패키지를 공개하지 않으면 모듈 외부에서 접근할 수 없다.

물론 모듈 내부에서는 이러한 암묵적 접근 수준에 대해서 영향을 받지 않는다.

```java
module my.module {
    exports com.my.package.name; // 해당 패키지 노출
    exports com.my.package.name to com.specific.package;  // 특정 모듈/패키지 에만 노출
}
```

모듈 시스템을 사용할 때 해당 모듈의 `.jar` 파일 경로를 `classpath`에 두면 

해당 모듈 내 모든 패키지가 마치 모듈이 없는 것처럼 적용되니 주의하자. (모듈 공개여부 적용x)

## 정리

- 프로그램 요소의 접근성은 가능한 최소한으로 유지하자.
  - 최소한의 공개 api를 제외하고는 클래스, 인터페이스, 멤버가 외부로 공개되는일이 없어야 한다.

## 📣 Reference

[Effective Java 3/E - Joshua J. Bloch](http://www.yes24.com/Product/Goods/65551284)<br/>
[WegraLee/effective-java-3e-source-code](https://github.com/WegraLee/effective-java-3e-source-code/tree/master/src/effectivejava/chapter3/item15)<br/>
[[이펙티브 자바 3판] 아이템 15. 클래스와 멤버의 접근 권한을 최소화하라](https://madplay.github.io/post/minimize-the-accessibility-of-classes-and-members)<br/>
[2. Java 9의 새로운 기능 - 모듈(1)](https://mslim8803.tistory.com/39)<br/>