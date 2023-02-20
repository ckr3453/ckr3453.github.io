---
title: "이펙티브자바 - 아이템11) equals를 재정의하려거든 hashCode도 재정의하라."
categories: 
    - book
date: 2023-02-17
last_modified_at: 2023-02-19
toc: true
toc_sticky: true
excerpt: "equals를 재정의할때는 항상 hashCode도 재정의 하자."
---

## 개요

`equals`를 재정의한 클래스들은 `hashCode`도 재정의해야 한다.

> hashCode란 해시 알고리즘으로 생성한 객체의 고유한 정수값이다. 해시 테이블이라는 자료구조에 사용된다.

`hashCode`를 재정의하지 않는다면 내부적으로 `equals`와 `hashCode`를 사용하여 중복체크를 하는 컬렉션 원소(`HashMap`, `HashSet`)로 사용할 때 문제를 일으킨다.

```java
Map<PhoneNumber, String> map = new HashMap<>();
map.put(new PhoneNumber(777, 111, 1234), "jenny");

// 두개의 PhoneNumber는 논리적으로 같으나 해시코드가 다르기때문에 null 반환
String name = map.get(new PhoneNumber(777, 111, 1234)); 
System.out.println("name = " + name); // null

Set<PhoneNumber> set = new HashSet<>();
set.add(new PhoneNumber(777, 111, 1234));
set.add(new PhoneNumber(777, 111, 1234));

// 위와 마찬가지로 2개의 인스턴스는 논리적으로 같으나 해시코드가 다르기 때문에 2 반환
System.out.println("set.size() = " + set.size()); // 2
```

`Object` 명세에도 다음과 같이 관련한 내용이 명시되어있다.

1. `equals` 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 `hashCode` 메서드는 몇번을 호출해도 일관되게 항상 같은 값을 반환해야 한다. (단, 애플리케이션이 다시 실행된다면 달라져도 상관없음.)
2. `equals(Object)`가 두 객체를 같다고 판단했다면, 두 객체의 `hashCode`는 똑같은 값을 반환해야 한다.


## 좋은 hashCode를 작성하기 위한 요령

좋은 해시 함수라면 서로 다른 인스턴스에 다른 `hashCode`를 반환한다.

- 1) `int` 변수 `result`를 선언한 후 값 `c`로 초기화 한다. 이때 `c`는 해당 객체의 첫번째 핵심필드를 단계 2.1 방식으로 계산한 해시코드다
- 2) 해당 객체의 나머지 핵심 필드 `f` 각각에 대해 다음 작업을 수행한다.
  - 2.1) 해당 필드의 해시코드 `c`를 계산한다.
    - 2.1.1) 기본 타입 필드라면, `Type.hashCode(f)`를 수행한다. 여기서 `Type`은 해당 기본 타입의 박싱 클래스다.
    - 2.1.2) 참조 타입 필드면서 이 클래스의 `equals` 메서드가 이 필드의 `equals`를 재귀적으로 호출해 비교한다면, 이 필드의 `hashCode`를 재귀적으로 호출한다. 계산이 더 복잡해질 것 같으면, 이 필드의 표준형(canonical representation)을 만들어 그 표준형의 `hashCode`를 호출한다. 필드의 값이 `null`이면 0을 사용한다.
    - 2.1.3) 필드가 배열이라면 핵심 원소 각각을 별도 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용해 핵심 원소의 해시코드를 계산한 다음, 2.2 방식으로 갱신한다. 배열에 핵심 원소가 하나도 없다면 단순히 상수(0을 추천)를 사용한다. 모든 원소가 핵심 원소라면 `Arrays.hashCode`를 사용한다.
  - 2.2) 단계 2.1 에서 계산한 해시코드 `c`로 `result`를 갱신한다. 코드로는 다음과 같다.

    ```java
    result = 31 * result + c;
    ```
- 3) `result`를 반환한다.


다음의 요령을 적용하여 작성한(재정의한) `hashCode`는 다음과 같다.


```java
public final class PhoneNumber {
    // 핵심 필드
    private final short areaCode, prefix, lineNum;

    ...

    @Override 
    public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof PhoneNumber))
            return false;
        PhoneNumber pn = (PhoneNumber)o;
  
        return pn.lineNum == lineNum && pn.prefix == prefix && pn.areaCode == areaCode;
    }

    @Override 
    public int hashCode() {
        int result = Short.hashCode(areaCode); // 첫번째 핵심필드가 기본타입 이므로, Type.hashCode(f)를 수행 (2.1.1 참고)
        result = 31 * result + Short.hashCode(prefix); // 위에서 계산한 해시코드를 이어서 갱신 (2.2 참고)
        result = 31 * result + Short.hashCode(lineNum); // 위에서 계산한 해시코드를 이어서 갱신 (2.2 참고)
        return result; // result 반환 (2.3 참고)
    }
}
```

hashCode를 작성하였으면 단위 테스트를 작성하여 동치인 인스턴스에 대해 똑같은 해시코드를 반환하는지 확인해보자.

```java
PhoneNumber num1 = new PhoneNumber(111,222,3333);
PhoneNumber num2 = new PhoneNumber(111,222,3333);

System.out.println("num1 = " + num1);
System.out.println("num2 = " + num2);
System.out.println("num1.equals(num2) = " + num1.equals(num2));
```

<center><img src="https://user-images.githubusercontent.com/36228833/219956006-60aafb9e-d54a-482d-97ef-6a781d97f810.png"></center>

> result를 갱신할때 31을 곱하는 이유는? 곱셈없이 hashCode를 구현한다면 모든 아나그램(구성요소는 같으나 순서만 다른 문자열)의 해시코드가 같아질 수 있음. 또한 31이 홀수이면서 소수이기 때문이다.

## Objects.hash()

해당 요령 외에도 Objects 클래스는 임의의 개수만큼 객체를 받아 해시코드를 계산해주는 정적 메서드 hash를 제공한다.

이 메서드를 사용하면 앞서 설명한 요령대로 hashCode를 단 한줄로 작성할 수 있다.

하지만 내부적으로 입력 인수를 담기위한 배열을 만들고 (입력 인수가 기본타입 일경우) 박싱/언박싱 과정도 거쳐야하기 떄문에 속도는 상대적으로 느리다.

사용 방법은 다음과 같다.

```java
@Override 
public int hashCode() {
    return Objects.hash(lineNum, prefix, areaCode);
}
```

해당 메서드는 성능에 민감하지 않은 경우에만 사용하자.

## 캐싱 & 지연 초기화

클래스가 불변이고 `hashCode`를 계산하는 비용이 크다면, 인스턴스가 초기화될때 미리 계산하여 캐싱하는 방식을 고려하자. (주로 해시의 키로 사용될 경우)

그와 반대로 해시의 키로 사용되지 않는 경우라면 실제로 `hashCode`가 호출될 때만 계산하도록 지연 초기화 방식을 사용하자.

```java
private int hashCode;

@Override
public int hashCode() {
    int result = hashCode;
    if (result == 0){
      result = Short.hashCode(areaCode);
      result = 31 * result + Short.hashCode(prefix);
      result = 31 * result + Short.hashCode(lineNum);
      hashCode = result;
    }
    
    return result;
}
```

단, 위와 같이 구현할 경우 `thread-safe` 하게 만들도록 신경써야한다.

## 주의할점

- hashCode를 구현할 때 **반드시 핵심필드를 포함해야 한다.**

핵심 필드를 생략할 경우, 결과적으로 얼마 되지않는 `hashCode`에 수많은 인스턴스가 몰려서 해시테이블의 성능을 심각하게 떨어뜨릴 수 있다.

(핵심 필드가 들어가지 않으면 다양한 `hashCode` 값이 나올수 없다.)

- `hashCode` 생성 규칙을 사용자에게 자세히 알리지 말자.

그래야 사용자가 해당 값에 의지하지 않게 되고 계산방식을 바꿀수도 있다.

## 정리

- `equals`를 재정의할때 반드시 `hashCode`도 재정의하자.
- 왠만해서는 그냥 `@EqualsAndHashCode` 혹은 ide가 제공하는 기능을 활용하여 자동으로 구성하는 방법을 사용하자.

## 📣 Reference

[Effective Java 3/E - Joshua J. Bloch](http://www.yes24.com/Product/Goods/65551284)<br/>
[WegraLee/effective-java-3e-source-code](https://github.com/WegraLee/effective-java-3e-source-code/tree/master/src/effectivejava/chapter3/item11)<br/>
[해시 함수 - 위키피디아](https://ko.wikipedia.org/wiki/%ED%95%B4%EC%8B%9C_%ED%95%A8%EC%88%98)<br/>