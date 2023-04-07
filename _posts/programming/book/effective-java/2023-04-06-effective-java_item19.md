---
title: "이펙티브자바 - 아이템19) 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라."
categories: 
    - book
date: 2023-04-06
last_modified_at: 2023-04-06
toc: true
toc_sticky: true
excerpt: "상속의 side effect"
---

## 개요

공개 API 클래스의 경우, 재정의가 가능한 메서드(`public`과 `protected` 면서 `final`이 아닌 모든 메서드)에 대한 상세 내용들을 문서로 제공해야한다.

(메서드를 어떤 순서로 호출하였는지, 각각의 호출 결과가 이어지는 처리에 어떤 영향을 주는지)

제공하지 않았을 경우 사용자 입장에선 예측할수 없는 문제들이 발생할수있다.

설계 시점부터 이러한점들을 고려하여 설계하고 문서화를 해야 문제를 예방할 수 있다.

## 메서드의 중간 훅(hook)을 선별하여 protected 메서드로 공개하기

내부 메커니즘을 문서로 남기는것만이 상속을 위한 설계의 전부가 아니다.

클래스 내부 동작과정 중간에 끼어들수있는 훅(hook)을 잘 선별해서 `protected` 메서드 형태로 공개해야 하수도 있다.

(ex. `AbstractList`의 `removeRange` 메서드)

```java
protected void removeRange(int fromIndex, int toIndex) {
    ListIterator<E> it = listIterator(fromIndex);
    for (int i=0, n=toIndex-fromIndex; i<n; i++) {
        it.next();
        it.remove();
    }
}
```

`List`를 구현하는 사용자는 `removeRange`에 관심이 없다.

`removeRange`는 내부에서 `clear` 메서드의 성능 최적화를 위해 존재한다.

```java
// Bulk Operations

/**
    * Removes all of the elements from this list (optional operation).
    * The list will be empty after this call returns.
    *
    * <p>This implementation calls {@code removeRange(0, size())}.
    *
    * <p>Note that this implementation throws an
    * {@code UnsupportedOperationException} unless {@code remove(int
    * index)} or {@code removeRange(int fromIndex, int toIndex)} is
    * overridden.
    *
    * @throws UnsupportedOperationException if the {@code clear} operation
    *         is not supported by this list
    */
public void clear() {
    removeRange(0, size());
}
```

`removeRange` 메서드가 없다면 하위 클래스에서 `clear` 메서드를 호출할때 원소의 크기에 비례해 성능이 느려질 것이다.

어떤 메서드를 `protected`로 공개할지는 안타깝게도 직접 하위 클래스를 만들어 테스트 해보는 방법밖에는 없다.

`protected` 메서드는 하나하나가 내부 구현에 해당되므로 가능한 그 수가 적어야한다.

## 상속용 클래스의 생성자는 재정의 가능 메서드를 호출해선 안된다.

상위 클래스 생성자가 하위 클래스 생성자보다 먼저 실행되므로 

하위 클래스가 재정의한 메서드가 하위 클래스의 생성자보다 먼저 동작하여 프로그램이 오동작 할수 있다.

```java
public class Super {
    public Super() {
        hi();       // Sub의 instant가 초기화 되지 않은 상태로 제일 먼저 실행됨
    }

    public void hi(){
    }
}

public final class Sub extends Super {
    private final Instant instant;

    Sub() {
        instant = Instant.now(); // 
    }

    @Override
    public void hi(){
        System.out.println(instant);
    }

    public static void main(String[] args) {
        Sub sub = new Sub();
        // instant 객체를 두번 출력해야 하지만 첫번째는 상위 클래스 생성자로 인해 null을 출력한다.
        sub.hi(); 
    }
}
```

재정의가 불가능한 `private`, `final`, `static` 키워드가 붙은 메서드는 상관없다.

## Cloneable, Serializable 인터페이스 상속 시 주의

둘중 하나라도 구현한 클래스를 상속할수 있게 만드는것은 일반적으로 좋지 않다.

`clone`, `readObject` 메서드는 새로운 객체를 만든다 (생성자와 비슷하다.)

상속용 클래스에서 `Cloneable`이나 `Serializable`을 구현해야 한다면 그에 대한 제약이 생성자와 비슷하다는 점에서 주의해야한다.

즉, `clone`, `readObject` 메서드에서 모두 재정의 가능 메서드를 호출해선 안된다.

> readObject는 하위 클래스 상태가 역직렬화가 되기도 전에 재정의한 메서드를 호출하게 되므로 문제 발생
> clone은 하위 클래스의 clone 메서드가 복제본 상태를 수정하기 전에 재정의한 메서드를 호출하게 되므로 문제 발생

추가로, 

`Serializable` 을 구현한 상속용 클래스가 `readResolve`나 `writeReplace`를 구현했다면 `protected`로 공개하자.

## 상속용이 아닌 클래스는 상속 금지

이러한 문제에서 벗어날수 있는 제일 좋은방법은 상속용으로 설계하지 않은 클래스는 상속을 금지하는것이다.

금지할 수 있는 방법은 두가지다.

- 클래스를 final로 선언하기
- 모든 생성자를 `private`(혹은 `package-private`)으로 선언 하고 `public` 정적 팩터리를 만들어 주기

## 정리

- 클래스 내부에서 어떻게 사용하는지 모두 문서로 남겨야한다.
- 경우에 따라 일부 메서드는 `protected`로 제공해야 할 수도 있다.
- 클래스를 확장해야할 명확한 이유가 없으면 상속을 금지시키자.

## 📣 Reference

[Effective Java 3/E - Joshua J. Bloch](http://www.yes24.com/Product/Goods/65551284)<br/>
[WegraLee/effective-java-3e-source-code](https://github.com/WegraLee/effective-java-3e-source-code/tree/master/src/effectivejava/chapter4/item19)<br/>