---
title: "이펙티브자바 - 아이템17) 변경 가능성을 최소화하라."
categories: 
    - book
date: 2023-04-05
last_modified_at: 2023-04-05
toc: true
toc_sticky: true
excerpt: "변경자 최소화하기"
---

## 개요

클래스를 가변으로 둘 이유가 없다면 불변으로 만들어서 변경 가능성을 최소화 하자.

불변 클래스는 가변 클래스보다 설계, 구현, 사용에 용이하며 오류가 생길 여지도 적고 훨씬 안전하다.

## 클래스를 불변으로 만들기

클래스를 불변으로 만들때 다음의 다섯가지 규칙을 따르자.

- 객체 상태를 변경하는 메서드를 제공하지 않는다
  - `setter` 등
- 클래스를 확장할 수 없도록 만든다.
  - 확장으로 인하여 객체의 상태가 변할수도 있기 때문이다.
- 모든 필드를 `final`로 선언한다.
- 모든 필드를 `private`으로 선언한다.
- 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 구성한다.

이상 다섯가지 규칙을 적용하여 만든 예시 불변 클래스이다.

```java
// 복소수를 표현한 클래스
public final class Complex {
    private final double re;  // 실수부
    private final double im;  // 허수부

    public static final Complex ZERO = new Complex(0, 0);
    public static final Complex ONE  = new Complex(1, 0);
    public static final Complex I    = new Complex(0, 1);

    private Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    // private 생성자와 함께 사용해야 한다. (정적 팩토리 메서드)
    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }

    public double realPart()      { return re; }
    public double imaginaryPart() { return im; }

    // 인스턴스 자신을 수정하지 않고 새로운 인스턴스를 만들어 반환한다.
    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                re * c.im + im * c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                (im * c.re - re * c.im) / tmp);
    }

    @Override 
    public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;
        return Double.compare(c.re, re) == 0
                && Double.compare(c.im, im) == 0;
    }
    @Override 
    public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override 
    public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```

## 불변 객체는 스레드 안전(thread-safe)하다

불변 객체는 근본적으로 스레드 안전하여 따로 동기화할 필요가 없다.

때문에 안심하고 공유할 수 있으며 자주 사용하는 값들은 다음과 같이 상수로 제공하여 재활용 할 수 있다.

```java
public static final Complex ZERO = new Complex(0, 0);
public static final Complex ONE  = new Complex(1, 0);
public static final Complex I    = new Complex(0, 1);
```

이러한 개념으로 public 생성자 대신 정적 팩터리 메서드를 제공하여 캐시 기능으로 활용할 수 있다.

## 불변 객체끼리는 내부 데이터를 공유할 수 있다.

불변 객체끼리는 가변 객체여도 복사(clone)하지 않고 원본 인스턴스와 공유해도 된다. (어차피 외부에 의해 상태가 변할 일이 없으니)

예시로 `BigInteger` 클래스의 `negate()` 메서드는 다음과 같이 원본 인스턴스의 내부 배열을 그대로 가르킨다.

```java
public class BigInteger extends Number implements Comparable<BigInteger> {
  final int signum; // 부호
  final int[] mag;  // 크기(절댓값)
  ...

  BigInteger(int[] magnitude, int signum) {
    // package-private 으로 생성자가 구성되어 있어서 외부에서 접근 불가능
    this.signum = (magnitude.length == 0 ? 0 : signum);
    this.mag = magnitude;
    if (mag.length >= MAX_MAG_LENGTH) {
        checkRange();
    }
  }

  public BigInteger negate() {
    // 크기는 같고 부호만 반대인 새로운 BigInteger를 반환하는 메서드
    // this 를 통해 원본 인스턴스의 부호와 크기를 그대로 가져다 쓴다.
    return new BigInteger(this.mag, -this.signum);
  }
  ..
}
```

## 객체를 만들때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다.

내부 구조가 복잡한 객체여도 구성요소들이 불변객체로 이뤄져 있으면 불변식을 유지하기 훨씬 수월하다.

예를 들어, `Map`의 Key나 `Set`의 구성요소로 활용 가능하다. 

해당 컬렉션의 구성요소가 바뀌면 불변식이 무너지는 경우가 발생하여 문제를 야기하지만 불변 객체는 그럴일이 없다.

(ex. 불변식이 보장되지 않아 동일한 의미의 `Map`의 Key나 `Set`의 구성요소가 2개이상이 될수 있다면..?)

## 불변 객체는 그 자체로 실패 원자성을 제공한다

> 실패 원자성이란? 메서드에서 예외가 발생한 후에도 그 객체는 전과 동일하게 여전히 유효한 상태여야 한다는 의미다.

상태가 변하지 않으니 잠깐이라도 불일치 상태에 빠질 가능성이 없다는 의미다.

## 불변 객체는 값이 다르면 반드시 독립된 객체로 만들어야 한다.

값을 만들수 있는 가짓수가 많다면 (상태를 변경할 수 없으니) 모두 각각 만들어야 하므로 큰 비용이 발생한다.

해당 문제를 대처하는데 두가지 방법이 있다.

- 흔히 쓰이는 다단계 연산들을 미리 예측하여 기본 기능으로 제공한다.
  - 단계 별로 객체를 생성하여 낭비되는 자원을 최소화 할 수 있다.
- 가변 동반 클래스를 함께 구성한다.
  - (내부에서만 접근 가능한) 연산을 돕는 가변 동반 클래스를 함께 구성하여 성능을 높인다.
  - `String` 클래스의 가변 동반 클래스는 `StringBuilder` 다.

## 객체를 신뢰할 수 없는 경우 방어적 복사를 사용하자.

신뢰할 수 없는 클라이언트로부터 객체를 인수로 받을 때 필요하다면 방어적으로 복사하여 사용해야한다.

(ex. 인수로 받는 객체가 반드시 불변이어야 할 경우)

```java
public static BigInteger safeInstance(BigInteger val) {
  return val.getClass() == BigInteger.class ? val : new BigInteger(val.toByteArray());
}
```

## 불변 객체를 상속하지 못하게 하는 다른 방법

생성자를 `private`(`package-private`)으로 구성하고 `public` 정적 팩터리를 제공하면 된다.

```java
private Complex(double re, double im) {
    this.re = re;
    this.im = im;
}

public static Complex valueOf(double re, double im) {
    return new Complex(re, im);
}
```

이렇게 구성함으로써 확장의 가능성을 완벽하게 차단할수 있고 동시에 유연성을 제공한다.

## 정리

- `Getter`가 있다고 해서 반드시 `Setter`를 제공할 필요는 없다.
- 클래스는 꼭 필요한 경우가 아니면 불변이어야 한다.
- 모든 클래스를 불변으로 만들 수는 없지만 그렇더라도 변경 가능한 부분을 최소한으로 줄이자.
  - 예측할 수 없는 경우의 수를 줄임으로써 오류가 생길 가능성을 줄일 수 있다.
- 합당한 이유가 없다면 모든 필드는 `private final`로 구성하자.
- 생성자의 경우, 내부 필드를 포함하여 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다.
  - 그 외에는 `public`으로 제공해서는 안된다.


## 📣 Reference

[Effective Java 3/E - Joshua J. Bloch](http://www.yes24.com/Product/Goods/65551284)<br/>
[WegraLee/effective-java-3e-source-code](https://github.com/WegraLee/effective-java-3e-source-code/tree/master/src/effectivejava/chapter4/item17)<br/>