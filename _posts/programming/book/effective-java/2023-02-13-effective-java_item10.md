---
title: "이펙티브자바 - 아이템10) equals는 일반 규약을 지켜 재정의 하라."
categories: 
    - book
date: 2023-02-17
last_modified_at: 2023-02-17
toc: true
toc_sticky: true
excerpt: "equals를 재정의할때 지켜야 할 규약들"
---

## 개요

`equals` 를 재정의하여 사용할 때 고려해야할 몇가지 사항이 있다. 

아래와 같은 상황일 경우에는 재정의하지 않는 것이 안전하다.

### 🖇️ 각 인스턴스가 본질적으로 고유하다.

`java.lang.Integer`나 `java.lang.String` 처럼 값을 표현하는 클래스가 아닌 `java.lang.Thread` 처럼 동작하는 개체를 표현하는 클래스의 경우, 

`Object`의 기본 `equals`를 사용하는 것이 좋다. **(값보다 동작하는 개체임을 나타내는게 더 중요하기 때문이다.)**

### 🖇️ 인스턴스의 논리적 동치성을 검사할 일이 없다.

예를 들어 두 개의 `Random` 객체가 같은 난수열을 만드는지 확인하는 것은 의미가 없다.

+) 논리적 동치성 검사가 필요한 경우

ex) 정규 표현식을 검사하기 위해 `java.utils.regex.Pattern`는 `equals`를 재정의 하여 두 개의 인스턴스가 같은 정규 표현식을 나타내는지 검사한다.

### 🖇️ 상위 클래스에서 재정의한 `equals`가 하위 클래스에도 딱 들어맞는다.

`Map`, `Set`, `List` 클래스는 각각의 상위 `Abstract` 클래스로부터 `equals`를 상속받아서 수정없이 그대로 사용한다.

### 🖇️ 클래스가 `private` 이거나 `package-private`(패키지 전용클래스)여서 `equals` 메서드를 호출할 일이 없다.

이럴 경우에는 `equals`가 호출되지 않도록 다음과 같이 재정의하여 막아야한다.

```java
@Override 
public boolean equals(Object o) {
  throw new AssertionError();
}
```

### 🖇️ 인스턴스 통제 클래스인 경우

값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스라면 `equals`를 재정의 하지 않아도 된다. (`Enum`도 여기 해당된다.)


그렇다면 언제 `equals` 를 재정의해야 할까? 

값이 아니라 논리적 동치성을 비교해야 하는데 **상위 클래스의 `equals`가 논리적 동치성을 비교하도록 재정의되지 않았을 때** 이다.

`equals`를 재정의하기 위해 지켜야할 일반 규약들을 살펴보자.


## 지켜야할 5가지 일반 규약들

### 📌 반사성(reflexivity)

> null이 아닌 모든 참조 값 x에 대해, x.equals(x) = true다.

즉, 객체는 자기 자신과 같아야 한다는 의미이다. (이건 일부러 위반하는게 더 어렵다.)

```java
public static void main(String[] args) {
    List<String> list = new ArrayList<>();
    list.add("test");
    System.out.println("list = " + list.contains("test")); // true
}
```

### 📌 대칭성(symmetry)

> null이 아닌 모든 참조값 x, y에 대해, x.equals(y) = true 면 y.equals(x) = true 다.

두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다는 의미다.

아래의 예제를 통해 대칭성이 위배되는 상황을 확인해보자.

```java
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    // 대칭성 위배
    @Override 
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
        if (o instanceof String)  // 한 방향으로만 작동한다!
            return s.equalsIgnoreCase((String) o);
        return false;
    }

    public static void main(String[] args) {
        CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
        String s = "polish";

        s.equals(cis); // false (cis의 존재를 모름.)
        cis.equals(s); // true  (그러나 cis는 String의 존재를 알고있음.)
    }
}
```

해당 방식은 String이 cis의 존재를 모르기 때문에 나타나는 대칭성 위배 현상이다.

`equals` 를 다음과 같이 수정한다.

```java
// 양방향으로 작동하게끔 수정한 equals 메서드
@Override 
public boolean equals(Object o) {
  // 동일한 타입일 경우에만 true 반환
  return o instanceof CaseInsensitiveString && ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```

### 📌 추이성(transitivity)

> null이 아닌 모든 참조값 x, y, z에 대해, x.equals(y) = true 면 y.equals(z) = true면 x.equals(z) = true 다.

x = y 이고 y = z 일때 x = z 여야 한다는 의미이다.

추이성이 위배되는 상황을 만들기 위해 상위 클래스에 없는 새로운 필드를 하위 클래스에 추가하는 상황을 상상해보자.

```java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override 
    public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
}
```

x, y로 이루어진 Point라는 클래스를 상속받는 ColorPoint 클래스 이다.

```java
public enum Color { 
  RED, ORANGE, YELLOW, GREEN, BLUE, INDIGO, VIOLET 
}
```

```java
public class ColorPoint extends Point {
   private final Color color;

   public ColorPoint(int x, int y, Color color) {
      super(x, y);
      this.color = color;
   }
}
```

만약 ColorPoint의 `equals`를 재정의하지 않는다면 Point의 `equals`가 상속되어 Color 정보는 비교대상에 들어가지 않게된다.

그렇다면 다음과 같이 색상정보까지 비교하는 ColorPoint의 `equals`를 구현하면 어떨까?

- 대칭성이 위배되는 경우

  ```java
  public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }

    @Override
    public booelan equals(Object o) {
        if(!(o instanceof ColorPoint))
          return false;
        return super.equals(o) && ((ColorPoint) o).color == color;
    }

    public static void main(String[] args) {
        Point p = new Point(1, 2);
        ColorPoint cp = new ColorPoint(1, 2, Color.RED);

        // 대칭성 위배됨.
        p.equals(cp); // true
        cp.equals(p); // false (class 타입이 달라서 false 도출)
    }
  }
  ```

`instanceof`로 인해 `false`를 도출함으로써 대칭성이 위배된다.

대칭성을 지키기 위해 Point 타입인 경우 `Point.equals`를 적용하여 색상 비교를 무시하도록 수정한다.

- 대칭성은 지켜지나 추이성이 위배되는 경우

  ```java
  public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }

    @Override
    public booelan equals(Object o) {
        if(!(o instanceof Point))
          return false;

        // o가 Point면 색상을 무시하고 비교한다.
        if(!(o instanceof ColorPoint))
          return o.equals(this);

        // o가 ColorPoint면 색상까지 비교한다.
        return super.equals(o) && ((ColorPoint) o).color == color;
    }

    public static void main(String[] args) {
        ColorPoint cp1 = new ColorPoint(1, 2, Color.RED);
        Point p = new Point(1, 2);
        ColorPoint cp2 = new ColorPoint(1, 2, Color.BLUE);

        // 추이성이 위배됨.
        cp1.equals(p);    // true
        p.equals(cp2);    // true
        cp1.equals(cp2);  // false (색상이 다르므로 false -> 추이성 위배)
    }
  }
  ```

그러나 이 방식은 대칭성은 지켜졌지만 추이성이 깨져버린다.

이런식으로 **구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 없다.**

이밖에도 다음과 같이 `getClass()`를 활용한 방법이 있으나 리스코프 치환 원칙에 위배된다.

```java
@Override
public boolean equals(Object o) {
    // 같은 구현 클래스의 객체와 비교할 때만 true를 반환한다.
    if (o == null || o.getClass() != getClass())
      return false;
    Point p = (Point) o;
    return p.x == x && p.y == y;
}
```


### 📌 일관성(consistency)

> null이 아닌 모든 참조값 x, y에 대해, x.equals(y)를 반복 호출하면 항상 같은 결과 값을 반환한다.

두 객체가 같다면 앞으로도 영원히 같아야 한다는 의미이다. 

가변객체의 경우 비교 시점에 따라 서로 다를수도 혹은 같을수도 있지만 불변 객체의 경우 한번 다르면 끝까지 달라야한다.

클래스를 작성할때 불변 클래스로 작성하는게 나은지 심사숙고하자.

또한 클래스가 불변이든 가변이든 `equals`의 판단에 신뢰할 수 없는 자원이 끼어들게 해선 안된다.


### 📌 null-아님(not-null)

> null이 아닌 모든 참조 값 x 에 대해, x.equals(null)은 false다.

모든 객체가 `null`과 같지 않아야 한다는 의미이다.

수많은 클래스가 `NullPointerException`을 회피하기 위해 `null` 검사를 명시적으로 작성한다. 

동치성을 검사하려면 `instanceof` 연산자로 올바른 타입인지 검사를 하게된다.

이 과정에서 타입이 `null` 일 경우 `false`를 반환하기 때문에 명시적으로 작성하지 않아도 된다.

```java
@Override
public boolean equals(Object o) {
  // null 일경우 false를 반환하게 된다.
  if(!(o instanceof MyType))
    return false;
  ....
}
```

## 규약에 맞춰 equals 재정의하기

일반 규약을 토대로 equals를 작성하기위한 단계를 구성하면 다음과 같다.

1. `==` 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
  - 자기 자신이면 `true`를 반환한다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
  - 그렇지 않다면 `false`를 반환한다.
  - 이때의 올바른 타입은 `equals`가 정의된 클래스일수도, 해당 클래스가 구현한 특정 인터페이스가 될수도 있다.
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다.
  - 하나라도 다르면 `false`를 반환한다.
  - 모든 필드는 `==`로 비교한다. (float, double 제외)
  - `float`은 `Float.compare(float, float)`
  - `double`은 `Double.compare(double, double)`

위의 내용들을 기반으로 핸드폰 번호 class의 `equals`를 작성해보자.

```java
public final class PhoneNumber {
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
    public boolean equals(Object o) {
        if (o == this) // 1. 자기자신 참조인지 확인
            return true;
        if (!(o instanceof PhoneNumber)) // 2. 올바른 타입인지 확인
            return false;
        PhoneNumber pn = (PhoneNumber)o; // 3. 올바른 타입으로 형변환
  
        return pn.lineNum == lineNum && pn.prefix == prefix && pn.areaCode == areaCode; // 4. 핵심 필드들이 일치하는지 모두 비교
    }
}
```

## 정리

- 꼭 필요한 경우가 아니면 `equals`를 재정의 하지 말자.
  - 대부분의 경우 기본 `Object`의 `equals`가 원하는 비교를 정확히 수행해준다.
- 재정의해야할 때는 핵심필드 및 다섯가지 규약을 확실히 지켜가며 작성해야한다.

## 📣 Reference

[Effective Java 3/E - Joshua J. Bloch](http://www.yes24.com/Product/Goods/65551284)<br/>
[WegraLee/effective-java-3e-source-code](https://github.com/WegraLee/effective-java-3e-source-code/tree/master/src/effectivejava/chapter3/item10)<br/>
[[이펙티브 자바 3판] 아이템 10. equals는 일반 규약을 지켜 재정의하라](https://madplay.github.io/post/obey-the-general-contract-when-overriding-equals)<br/>
[[EFFECTIVE JAVA] - 이펙티브자바 아이템 10 EQUALS는 일반 규약을 지켜 재정의하라](https://yhmane.tistory.com/154)<br/>
