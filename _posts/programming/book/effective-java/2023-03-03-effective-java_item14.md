---
title: "이펙티브자바 - 아이템14) Comparable을 구현할지 고려하라."
categories: 
    - book
date: 2023-03-03
last_modified_at: 2023-03-03
toc: true
toc_sticky: true
excerpt: "Comparable 인터페이스를 구현할 때 고려사항"
---

## 개요

`Comparable` 인터페이스는 비교용 메서드인 `compareTo()`를 지원한다.

`compareTo()`는 단순 동치 비교에 순서까지 비교할수 있으며 제네릭 하다는 점에서 `Object.equals()`와 다르다.

순서를 비교할수 있는 특징 때문에 `Comparable`을 구현한 클래스는 **내부 인스턴스들이 어떠한 순서가 있다는 것**을 의미하기도 한다.

알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성한다면 반드시 `Comparable` 인터페이스를 구현할 것을 권한다.

(예로 컬렉션 객체인 `TreeSet`, `TreeMap`와 유틸 클래스인 `Collections`, `Arrays`등이 있다.)

`Comparable` 인터페이스를 구현할 때 고려해야할 사항들을 알아보자.


## compareTo() 일반 규약

`compareTo()`의 일반규약은 [아이템10에서 언급한 `equals`의 일반규약](https://ckr3453.github.io/book/effective-java_item10/#%EC%A7%80%EC%BC%9C%EC%95%BC%ED%95%A0-5%EA%B0%80%EC%A7%80-%EC%9D%BC%EB%B0%98-%EA%B7%9C%EC%95%BD%EB%93%A4)과 동일하게 대칭성, 추이성, 반사성을 충족해야한다.

- 이 객체와 주어진 객체의 순서를 비교한다.
  - 이 객체가 주어진 객체보다 작으면 음수, 같으면 0, 크면 양수를 반환한다.
  - 비교할 수 없는 타입의 객체가 주어지면 `ClassCastException`을 던진다.
- `x.compareTo(y) < 0`이면 `y.compareTo(x) > 0`이어야 한다. (대칭성)
  - x가 y보다 작으면 y가 x보다 커야한다.
- `x.compareTo(y) > 0 && y.compareTo(z) > 0` 이면 `x.compareTo(z) > 0` 이다. (추이성)
  - x과 y보다 크고 y는 z보다 크면 결국 x는 z보다 커야한다.
- `x.compareTo(y) == 0` 일때 `x.compareTo(z) == y.compareTo(z)` 이다. (반사성)
  - x와 y가 같을 때 동일한 크기인 z에 대해서도 x와 z는 같고 y와 z는 같다.
- `(x.compareTo(y) == 0) == (x.equals(y))` 이다. (권장)
  - x와 y의 `compareTo()`와 `equals()`의 결과가 일관되어야 한다.
  - 이를 지키지 않았을 때 다음과 같이 원치 않는 결과가 나올 수 있다.

  ```java
  // HashSet.add(Object o)는 저장할 객체의 equals()와 hashCode()를 호출하여 비교한다.
  // equals로 비교하면 1.0과 1.00은 서로 다르기 때문에 원소가 2개 나온다.
  HashSet<BigDecimal> hashSet = new HashSet<>();
  hashSet.add(new BigDecimal("1.0"));
  hashSet.add(new BigDecimal("1.00"));
  hashSet.forEach(bigDecimal -> System.out.println("hashSet's bigDecimal = " + bigDecimal));
  
  // TreeSet.add(Object o)는 저장할 객체의 compareTo()를 호출하여 비교한다.
  // compareTo로 비교하면 1.0과 1.00은 서로 같기 때문에 원소가 1개 나온다.
  TreeSet<BigDecimal> treeSet = new TreeSet<>();
  treeSet.add(new BigDecimal("1.0"));
  treeSet.add(new BigDecimal("1.00"));
  treeSet.forEach(bigDecimal -> System.out.println("treeSet's bigDecimal = " + bigDecimal));
  ```

<center><img src="https://user-images.githubusercontent.com/36228833/222679906-6b55aea8-1e44-49eb-a066-83905e81aaee.png"></center><br/>

## compareTo() 구현하기

구현하기에 앞서 몇가지 주의사항을 짚고 넘어가자.

- `Comparable`은 제네릭 인터페이스이므로 `compareTo()`의 매개변수 타입은 컴파일시 정해진다.
  - 타입이 잘못됐다면 컴파일 자체가 되지 않기떄문에 따로 **입력받은 매개변수 타입을 확인하거나 형변환 할 필요가 없다**는 뜻이다.
- `compareTo()`는 `equals()`처럼 각 필드가 동치인지를 비교하는게 아니라 순서를 비교한다.
  - 객체 참조 필드를 비교하려면 해당 객체의 `compareTo()`를 호출하게된다.
  - 만약 `Comparable` 인터페이스를 구현하지 않은 필드이거나 표준이 아닌 순서로 비교할 시 비교자(`Comparator`)를 사용하자.
  - 비교자는 직접 만들거나 자바가 제공하는 것 중에서 쓰면 된다.
- 자바 7부터는 `compareTo()`에서 값 기본 타입 필드를 비교할 때 박싱된 기본 타입 클래스들의 `compare()`를 사용하자.
  - 관계 연산자 `<` 와 `>` 를 사용하는 이전 방식은 거추장스럽고 오류를 유발하여 추천하지 않는다.
- 가장 핵심적인 필드부터 비교하자.
  - 핵심 필드를 우선으로 비교하여 의미없는 비교를 최소화 하자.

주의사항을 참고하여 작성한 예시는 다음과 같다.

### 객체 참조 필드 비교
```java
// 대소문자를 구분하지 않는 클래스
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    @Override
    public int compareTo(CaseInsensitiveString cis){
        // 자바에서 제공하는 문자열 비교자(Comparator)
        // 대소문자에 상관없이 알파벳 순서에 따라 비교해준다. 
        return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
    }
}
```

```java
CaseInsensitiveString cisA = new CaseInsensitiveString("AaAa");
CaseInsensitiveString cisB = new CaseInsensitiveString("bBBb");
// a가 b보다 순서가 먼저(작음)이므로 음수(-1)가 출력되야 한다.
System.out.println("cisA.compareTo(cisB) = " + cisA.compareTo(cisB));
```

<center><img src="https://user-images.githubusercontent.com/36228833/222680115-d329be63-f2ad-44bb-8f02-583652a017c9.png"></center><br/>

### 기본 타입 필드 비교

```java
// 핸드폰 번호 클래스
public final class PhoneNumber implements Comparable<PhoneNumber>{
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "지역코드");
        this.prefix   = rangeCheck(prefix,   999, "프리픽스");
        this.lineNum  = rangeCheck(lineNum, 9999, "가입자 번호");
    }

    private static short rangeCheck(int val, int max, String arg) {
        if (val < 0 || val > max)
            throw new IllegalArgumentException(arg + ": " + val);
        return (short) val;
    }

    @Override
    public int compareTo(PhoneNumber pn) {
        // short는 기본 정수형 타입(primitive)이다.
        // 박싱된 타입인 Short의 compare()를 사용한다.
        // 핵심필드 순으로 비교하여 성능을 최적화 한다.
        int result = Short.compare(areaCode, pn.areaCode);
        if(result == 0) {
            result = Short.compare(prefix, pn.prefix);
            if(result == 0) {
                result = Short.compare(lineNum, pn.lineNum);
            }
        }

        return result;
    }
}
```

```java
PhoneNumber pn = new PhoneNumber(11, 123, 1243);
PhoneNumber pn2 = new PhoneNumber(11, 21, 4432);
// pn과 pn2의 첫번째 인수인 areaCode가 같으므로 두번째 인수인 prefix를 비교한다.
// pn의 prefix 값이 pn2의 prefix 값보다 102 크다 (= pn은 pn2보다 102만큼 순서가 뒤에있다.)
// 즉 양수(102)가 출력 되어야 한다.
System.out.println("pn.compareTo(pn2) = " + pn.compareTo(pn2));
```

<center><img src="https://user-images.githubusercontent.com/36228833/222680145-566917ee-42e7-4956-b5fd-dcbe65610e93.png"></center><br/>

### 비교자 생성 메서드 (기본타입)

자바 8부터 메서드 체이닝 방식으로 비교자(`Comparator`)를 생성할 수 있게끔 지원한다.

이 방식은 간결하게 비교자를 작성할수 있다는 장점이 있지만 **약간의 성능 저하가 뒤따른다.**

`comparingInt()`는 람다식을 인수로 받으며 해당 인수에 맞는 비교자를 반환한다. 입력 인수의 타입을 명시해야 한다.

`thenComparingInt()`도 람다식을 인수로 받으며 `comparingInt()`에서 반환된 비교자로 추가 비교를 수행한다. 앞서 인수 타입을 작성했으므로 여기선 작성하지 않아도 된다.

위의 작성한 `PhoneNumber` 클래스의 `compareTo()`를 다시 작성해보겠다.

```java
public final class PhoneNumber implements Comparable<PhoneNumber>{
    // ..이하 생략

    private static final Comparator<PhoneNumber> COMPARATOR = 
        comparingInt((PhoneNumber pn) -> pn.areaCode) // areaCode를 비교
            .thenComparingInt(pn -> pn.prefix)        // prefix를 비교
            .thenComparingInt(pn -> pn.lineNum);      // lineNum을 비교

    @Override
    public int compareTo(PhoneNumber pn) {
        return COMPARATOR.compare(this, pn);
    }
}
```

<center><img src="https://user-images.githubusercontent.com/36228833/222680193-114db4fb-d25d-4112-8556-03825fc64f40.png"></center><br/>

(순서가 얼만큼 차이나는지 까지는 출력되지 않는것 같다.)

+) 추가로 다른 기본 타입을 비교하고 싶을때는 

<center><img src="https://user-images.githubusercontent.com/36228833/222680245-23fce358-c996-4869-8547-7e3e5b06ed4f.png"></center><br/>

`short`처럼 더 작은 정수 타입은 `comparingInt`를, `float`과 `double`은 `comparingDouble`을, `long`은 `comparingLong`을 사용하면 된다.

### 비교자 생성 메서드 (객체 참조 타입)

위에서 설명한 기본타입과 유사하다.

단 참조타입의 경우 인수를 2개까지 받을 수 있는데, 두번째 인수의 경우 비교 대상을 반대방향(내림차순) 혹은 정방향(오름차순)으로 비교할 수 있게끔 지원한다.

그리고 추가로 `thenComparing()`은 두번째 인수에 대상과 비교할 비교자를 직접 넣을 수 있게끔 다중정의 되어있다.

```java
public final class PhoneNumber implements Comparable<PhoneNumber>{
    
    private static final Comparator<PhoneNumber> COMPARATOR = 
        comparing(PhoneNumber::getAreaCode, naturalOrder()) // 정방향(오름차순)
            .thenComparing(PhoneNumber::getPrefix, reverseOrder()) // 반대방향(내림차순)
            .thenComparingInt(pn -> pn.lineNum);

    // ..이하 생략
}
```

```java
PhoneNumber pn = new PhoneNumber(11, 123, 1243);
PhoneNumber pn2 = new PhoneNumber(11, 21, 4432);
// pn의 prefix가 pn2의 prefix보다 더 크지만 비교자의 순서 영향으로 음수를 출력한다.
System.out.println("pn.compareTo(pn2) = " + pn.compareTo(pn2));
```

<center><img src="https://user-images.githubusercontent.com/36228833/222680309-ef094487-21bb-4f49-8135-7be7f40f3c59.png"></center><br/>


## 값의 차를 기준으로 비교하지 말기

다음과 같이 `compareTo()` 나 `compare()`를 작성할 때 `값의 차`를 기준으로 작성하는 경우가 종종 있다. (본인도 코딩테스트 문제를 풀때 종종 이런식으로 구현했었다..)

```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return o1.hashCode() - o2.hashCode(); // 값의 차로 순서를 비교하는 것은 오류를 야기할 수 있다.
    }
}
```

이 방식으로 사용할 경우 비교 대상 값의 범위에 따라 **정수 오버플로우 혹은 부동소수점 계산 오류를 발생시킬 수 있다.**

따라서 해당 방식이 아닌 위에 기술한 비교자(`Comparator`)를 활용한 방법으로 비교하자.

## 정리

- 순서를 고려해야하는 클래스를 작성할때는 꼭 `Comparable` 인터페이스를 구현하자.
- `compareTo()` 에서 값을 비교할 때 `<` 와 `>` 사용을 지양하자.
- 값의 차로 비교를 하지말고 비교자(`Comparator`)를 활용하자

## 📣 Reference

[Effective Java 3/E - Joshua J. Bloch](http://www.yes24.com/Product/Goods/65551284)<br/>
[WegraLee/effective-java-3e-source-code](https://github.com/WegraLee/effective-java-3e-source-code/tree/master/src/effectivejava/chapter3/item14)<br/>
[Java의 복사 생성자 및 팩토리 메소드](https://www.techiedelight.com/ko/copy-constructor-factory-method-java/)<br/>
[[Java] String - CASE_INSENSITIVE_ORDER, length(), isEmpty()](https://priming.tistory.com/21)<br/>
