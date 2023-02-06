---
title: "이펙티브자바 - 아이템5) 자원을 직접 명시하지말고 의존 객체 주입을 사용하라."
categories: 
    - book
date: 2023-02-06
last_modified_at: 2023-02-06
toc: true
toc_sticky: true
excerpt: "의존 관계 주입 (Dependency Injection)"
---

## 개요

클래스가 자원에 의존하는 경우가 있다. 이럴경우 발생하는 문제점과 해결 방법에 대해 알아보자.

핸드폰 배터리 충전기를 유틸 클래스로 구현하였을 때를 예시로 들어보자.

```java
public class BatteryCharger {
    private static final SmartPhone phone = new IPhone(); // 아이폰에 의존

    private BatteryCharger() {...}
    public static void charge() {...}
    public static boolean isCharged() {...}
}
```

핸드폰 배터리를 충전하는 대상이 있고 그와 관련한 메서드 이다.

싱글톤으로 구현한다면 이렇게 될것이다.

```java
public class BatteryCharger {
    private static SmartPhone phone = new IPhone(); // 아이폰에 의존
    public static BatteryCharger INSTANCE = new BatteryCharger();

    private BatteryCharger() {...}
    public static void charge() {...}
    public static boolean isCharged() {...}
}
```

두 방식은 배터리 충전의 대상 즉, 자원을 아이폰으로 직접 명시함으로써 아이폰 외의 다른 스마트폰은 해당 유틸 클래스(BatteryCharger)를 사용할 수 없게 된다.

(클래스 내부에 final 키워드를 제거하고 다른 스마트폰으로 교체하는 메서드를 추가할 순 있지만, 멀티쓰레드 환경에서는 사용할 수 없다.)

이처럼 사용하는 자원에 따라 동작이 달라지는 클래스는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.

## 의존관계 주입

그렇다면 자원에 의존적이지 않고 필요한 자원으로 유연하게 사용할 순 없을까?

- 여러 자원 인스턴스를 지원해야한다.
- 사용자가 원하는 자원을 사용해야한다.

```java
public class BatteryCharger {
    private final SmartPhone phone; // 불변 상태 유지

    public BatteryChecker(SmartPhone phone){
        this.phone = phone; // 의존관계 주입
    }

    public static void charge() {...}
    public static boolean isCharged() {...}
}
```

다음과 같이 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식을 사용하면 된다.

(final 키워드로 불변임을 보장하기 때문에 여러 사용자가 의존 객체들을 안심하고 공유할 수 있다.)

해당 방식을 응용하여 생성자에 **자원 팩토리를 넘겨주는 방식**이 있다.

> 팩터리란? 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체를 뜻한다.

```java
public class SmartPhone {
}

public class IPhone extends SmartPhone {
}

public class Galaxy extends SmartPhone {
}

public class BlackBerry {
}
```

```java
public class BatteryCharger {
    private final SmartPhone phone;

    // 자원 팩터리를 통해 SmartPhone 하위 타입만 받도록 제한할 수 있다.
    public BatteryChecker(Supplier<? extends SmartPhone> phoneSupplier){
        this.phone = phoneSupplier.get(); 
    }

    public static void charge() {...}
    public static boolean isCharged() {...}

    public static void main(String[] args){
        // 람다식 혹은 메서드 참조로 SmartPhone의 하위 타입 인스턴스를 넣어준다.
        BatteryCharger batteryCharger = new BatteryCharger(Galaxy::new);
        BatteryCharger batteryCharger = new BatteryCharger(() -> new IPhone);

        // SmartPhone 의 하위타입이 아니므로 오류
        // BatteryCharger batteryCharger = new BatteryCharger(BlackBerry::new);
    }
}
```

자바 8의 `Supplier<T>` 인터페이스를 통해 자원에 대한 팩터리를 표현할 수 있으며 사용자는 **자신이 명시한 타입의 하위 타입**이라면 무엇이든 생성할 수 있는 팩터리를 넘길 수 있다.

해당 기법은 **클래스의 유연성, 재사용성, 테스트 용이성을 기막히게 개선**해준다!

## 정리
> 클래스가 내부적으로 하나 이상의 자원에 의존하고 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다. 이 자원들을 클래스가 직접 만들게 해서도 안된다. 대신 필요한 자원(or 자원을 만드는 팩토리)을 생성자(정적 팩토리 or 빌더)에 넘겨주자.

## 📣 Reference

[Effective Java 3/E - Joshua J. Bloch](http://www.yes24.com/Product/Goods/65551284)<br/>
