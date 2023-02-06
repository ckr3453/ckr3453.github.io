---
title: "이펙티브자바 - 아이템7) 다 쓴 객체 참조를 해제하라."
categories: 
    - book
date: 2023-02-06
last_modified_at: 2023-02-06
toc: true
toc_sticky: true
excerpt: "가비지 컬렉터의 손에 닿지않는 곳"
---

## 개요

자바 개발자들은 JVM의 가비지 컬렉터가 메모리를 알아서 관리하고 있기 때문에 C, C++ 같은 언어에 비해 상대적으로 편하게 개발할 수 있다.

그렇다고해서 메모리 관리를 아예 신경쓰지 않아도 된단 말은 아니다.

가비지 컬렉터의 손이 닿지 않는, 예상치 못한 곳에서 메모리 누수가 일어날 수 있다.

이번 아이템에선 어떤 경우에 메모리 누수가 일어날 수 있을 지 살펴보자.

## 메모리 직접 관리

다음은 자료구조 Stack을 구현한 코드이다.

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }

    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보한다.
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

해당 코드에서 메모리 누수가 일어나는 곳은 `Stack.pop()` 이다.

`pop()`에 의해서 다 쓴 참조(obsolete reference)가 된 객체들을 스택이 여전히 가지고 있기 때문이다. (스택은 `pop()`에 의해 뽑혀진 객체를 내부에서 쓸일이 없음.)

> 다 쓴 참조란? 문자 그대로 앞으로 다시 쓰지 않을 참조를 뜻한다.

가비지 컬렉터는 **객체 참조 하나를 살려두면 그 객체 뿐 아니라 그 객체가 참조하는 모든 객체를 회수해가지 못하기 때문**에 성능에 지대한 악영향을 줄 수 있다.

그래서 다 쓴 참조의 경우 `null` 처리를 해줌으로써 참조 해제를 해야한다.

```java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    elements[size] = null;  // 다 쓴 참조 해제
    return elements[--size];    
}
```

다 쓴 참조를 `null` 처리를 하면 NPE를 통해 `null` 처리된 참조에 대한 접근을 미리 예방할 수 있는 장점또한 가지고 있다.

하지만 그렇다고 해서 모든 객체에 일일이 `null` 처리를 할 필요는 없다. (변수의 범위를 최소화 해서 정의하자.)

이 Stack 클래스가 메모리 누수에 취약하게 된 근본적인 원인은 무엇일까?

해당 Stack 클래스는 elements 배열로 저장소 풀을 만들어서 관리한다 (= 메모리를 직접 관리)

```java
public class Stack {
    private Object[] elements;
    ...
}
```

Stack의 구조상 elements 배열에 '활성 영역'에 해당하는 원소들만 사용되고 '비활성 영역'은 쓰이지 않게 되어있다.

> 활성 영역: 인덱스가 배열 size보다 작은 원소들 / 비활성 영역: 그 외의 원소들 (`pop()` 으로 인해 사용되지 않는 원소들)

그러나 문제는 가비지 컬렉터 입장에선 **비활성 영역 또한 유효한 객체라고 판단**하기 때문에 객체를 회수하지 않는다. 

그러므로 개발자는 **비활성 영역이 되는순간 null 처리를 해서 해당 객체를 더이상 쓰지 않을 것임을 가비지 컬렉터에 알려야 한다.**

## 캐시

캐시 또한 메모리 누수를 일으키는 원인이 된다.

예를 들면, 객체 참조를 캐시에 넣고 다 쓴 뒤로도 한참을 놔두는 일이다.

이럴 경우 `WeakHashMap` 을 사용하여 캐시를 만들자. (캐시 외부에서 키를 참조하는 동안만 엔트리가 살아 있음)

```java
WeakHashMap<Integer, String> map = new WeakHashMap<>();

Integer key1 = 1000;
Integer key2 = 2000;

map.put(key1, "참조가 해제되어 가비지 컬렉터에 의해 수거됩니다.");
map.put(key2, "아직 남아있음.");

key1 = null;  // 참조 해제
System.gc();  // 가비지 컬렉터에 의해 수거

TimeUnit.SECONDS.sleep(5); // 가비지 컬렉터가 수거해갈때까지 잠시 대기
map.entrySet().forEach(System.out::println);
```

<center><img src="https://user-images.githubusercontent.com/36228833/216977641-6fcf78f7-0ce1-4ff3-8d47-1704bc88862e.png"></center><br/>

`WeakHashMap`을 관리할 때는 키값을 (String literal로 생성한) `String` 타입을 사용하면 안된다. (= Strong Reference 이므로 사라지지 않음)

## 리스너 혹은 콜백

메모리 누수의 세번째 원인은 리스너(Listener) 혹은 콜백(Callback) 이다.

클라이언트가 콜백을 등록만 하고 이후 명확하게 해지하지 않는다면 계속 쌓여갈 것이다.

이럴 때도 캐시와 동일하게 콜백을 약한 참조(weak reference)로 저장하면 가비지 컬렉터가 즉시 수거해 간다. (ex. `WeakHashMap`에 키로 저장)

## 정리

- 메모리 누수는 겉으로 잘 드러나지 않아 시스템에 수년간 잠복할 수도 있음.
- 이런 종류의 문제들은 예방법을 미리 익혀서 사전에 예방하는 것이 매우 중요하다.


## 📣 Reference

[Effective Java 3/E - Joshua J. Bloch](http://www.yes24.com/Product/Goods/65551284)<br/>
[WegraLee/effective-java-3e-source-code](https://github.com/WegraLee/effective-java-3e-source-code/tree/master/src/effectivejava/chapter2/item7)<br/>
[[이펙티브 자바] 아이템 7. 다 쓴 객체 참조를 해제하라](https://velog.io/@lychee/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C-%EC%9E%90%EB%B0%94-%EC%95%84%EC%9D%B4%ED%85%9C-7.-%EB%8B%A4-%EC%93%B4-%EA%B0%9D%EC%B2%B4-%EC%B0%B8%EC%A1%B0%EB%A5%BC-%ED%95%B4%EC%A0%9C%ED%95%98%EB%9D%BC#-%EC%BD%9C%EB%B0%B1-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EB%88%84%EC%88%98%EC%9D%98-%EC%A3%BC%EB%B2%94)<br/>
[이펙티브 자바 규칙 6 - 유효기간이 지난 객체는 폐기하자](https://meaownworld.tistory.com/77)<br/>
[[Effective Java] 아이템7 - 다 쓴 객체 참조를 해제하라](https://mongsil1025.github.io/book/effective-java/item7/)<br/>
[Weak reference의 이해](https://tourspace.tistory.com/37)<br/>
[[Java] 참조 유형 (Strong Reference / Soft Reference / Weak Reference / Phantom Reference)](https://jangjjolkit.tistory.com/31)<br/>
