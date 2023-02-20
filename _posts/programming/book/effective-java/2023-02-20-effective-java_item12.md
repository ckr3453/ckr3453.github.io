---
title: "이펙티브자바 - 아이템12) toString을 항상 재정의하라."
categories: 
    - book
date: 2023-02-20
last_modified_at: 2023-02-21
toc: true
toc_sticky: true
excerpt: "toString을 재정의하여 사용하자."
---

## 개요

`Object.toString()`은 인스턴스의 정보를 `클래스명@해시코드_16진수` 문자열 형태로 반환해주는 메서드이다.

```java
public static void main(String[] args) {
    // item11 에서 사용한 PhoneNumber 클래스
    PhoneNumber phoneNumber = new PhoneNumber(111,222,3333);
    System.out.println("phoneNumber = " + phoneNumber.toString());
}
```

<center><img src="https://user-images.githubusercontent.com/36228833/220149832-912d693c-31bb-496c-a4c3-de8b908f4b34.png"></center><br/>

해당 정보만으로는 인스턴스가 가진 주요 정보들을 모두 표현했다고 보기 어렵다.

다음은 `Object.toString()`의 규약중 일부이다.

1. 간결하면서 사람이 읽기 쉬운 형태의 유익한 정보를 반환해야 한다.
2. 모든 하위 클래스에서 이 메서드를 재정의 하라.

<br/>
당연한 얘기지만, 알아볼 수 없는 해시코드 대신에 규약에 따라 `toString()`을 잘 구현한다면 디버깅이 훨씬 편해진다. 

`Object.toString()`가 주요 정보들을 반환하도록 재정의 하는 방법을 알아보자.

## 포맷 명시 여부 결정

먼저 `toString()`을 구현하기 전에 포맷을 명시할지 안할지 정해야한다.

📌 포맷을 명시할 경우
- 해당 객체는 표준적이며 명확하고 사람이 읽을 수 있게 된다.
  - 해당 값을 파싱하여 입출력 및 영속 데이터 저장 등에 활용 가능하다.
- 그러나 해당 포맷에 의존적이게 된다는 단점이 있다.
  - 추후 포맷이 변경되면 해당 포맷을 사용하던 코드들을 전부 수정해야한다.

포맷을 명시하기로 했다면 다음과 같이 **아주 정확하고 상세하게 작성해야한다.**

```java
/**
* 이 전화번호의 문자열 표현을 반환한다.
* 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
* XXX는 지역 코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
* 각각의 대문자는 10진수 숫자 하나를 나타낸다.
*
* 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면,
* 앞에서부터 0으로 채워나간다. 예컨대 가입자 번호가 123이라면
* 전화번호의 마지막 네 문자는 "0123"이 된다.
*/
@Override 
public String toString() {
    return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
}
```

📌 포맷을 명시하지 않을 경우
- 표준으로 사용할 수는 없지만 포맷을 개선할 수 있는 유연성을 얻을 수 있다.

## 반환값에 대한 API 제공

`toString()`이 제공하는 정보가 필요한 사용자를 위해 포맷 명시 여부와 상관없이 `toString()`이 반환한 값에 포함된 정보를 얻어올수 있는 API를 제공해야한다.

```java
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    ...

    @Override 
    public String toString() {
        return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
    }

    // api 제공
    public short getAreaCode() {
        return areaCode;
    }

    public short getPrefix() {
        return prefix;
    }

    public short getLineNum() {
        return lineNum;
    }
}
```

API를 제공하지 않을 경우 사용자는 정보를 얻기위해 다음과 같은 단점들을 감수하고 `toString()` 반환값을 파싱해야만 한다.

- `toString()` 값을 파싱하는 로직을 별도로 수행해야 하므로 성능에 좋지않다.
- 향후 포맷이 변경되면 파싱 로직도 수정해야한다.

## 재정의 제외 대상

다음과 같은 경우에는 `toString()`을 재정의하지 않아도 된다.

- 정적 유틸리티 클래스
  - 유틸 클래스 특성상 `toString()`을 사용할 일이 없기때문에 제공할 이유가 없다.
- 열거 타입(Enum) 클래스
  - 대부분의 열거형 상수들은 이미 읽기 좋은 형태로 `toString()`을 제공하고 있기 때문에 재정의할 필요가 없다.
- 대다수의 컬렉션 구현체
  - 이미 추상 컬렉션 클래스들의 `toString()`을 상속하여 쓰고있다.

## 정리

- `Object.toString()`을 구현 클래스의 주요 정보가 모두 포함되도록 재정의하자.
  - 상위 클래스에서 이미 알맞게 재정의한 경우는 제외

## 📣 Reference

[Effective Java 3/E - Joshua J. Bloch](http://www.yes24.com/Product/Goods/65551284)<br/>
[WegraLee/effective-java-3e-source-code](https://github.com/WegraLee/effective-java-3e-source-code/tree/master/src/effectivejava/chapter3/item12)<br/>