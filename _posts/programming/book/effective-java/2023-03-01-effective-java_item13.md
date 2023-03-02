---
title: "이펙티브자바 - 아이템12) clone 재정의는 주의해서 진행하라. (작성중)"
categories: 
    - book
date: 2023-03-01
last_modified_at: 2023-03-02
toc: true
toc_sticky: true
excerpt: "clone()을 재정의할때 고려 사항"
---

## 개요

`Cloneable`은 복제해도 되는 클래스임을 명시하는 용도의 `mixin interface`이다.

> 믹스인(mixin)이란? 다른 클래스에서 '사용'할 목적으로 만들어진 클래스 혹은 구현된 메서드가 포함된 인터페이스 이다. 다른 클래스의 부모 클래스가 되지 않으면서 다른 클래스에서 사용할 수 있는 메서드를 포함하기 때문에 '상속'이 아닌 '포함'으로 표현한다.

<center><img src="https://user-images.githubusercontent.com/36228833/222418323-a315eefc-be90-462d-8436-c0e32341bf34.png"></center><br/>

`Cloneable` 내부를 보면 아무런 메소드가 보이지 않지만 명시된 내용에 따르면 해당 인터페이스를 구현한 클래스는 `Object.clone()`를 재정의 해야한다고 나와있다.

(주의. `Cloneable`을 구현하지 않고 `Object.clone()`을 사용할 시 `CloneNotSupportedException`을 던진다.)

<center><img src="https://user-images.githubusercontent.com/36228833/222418418-8c456131-b1b2-4c7e-ad20-ad86d1891e4a.png"></center><br/>

`Object.clone()`은 기본적으로 대상 객체의 모든 필드 및 메서드를 얕은 복사하여 반환하는 동작을 수행한다.

> 얕은 복사(Shallow copy)란? 값 자체를 복사하는것이 아니라 객체의 주소 값을 복사한다. 즉, 원본 객체가 수정되었을 시 얕은 복사를 통해 반환된 객체또한 수정된다.

`Cloneable` 인터페이스를 구현하여 `Object.clone()`을 재정의하는 방법과 유의사항에 대해 알아보자.

## clone() 구현하기

### 일반 규약 명세

`Object.clone()`의 일반 규약 명세는 다음과 같다.

- `x.clone() != x` 식은 참이어야 한다.
  - 복사된 객체가 원본이랑 같은 주소를 가지면 안된다는 뜻이다. (얕은 복사가 아닌 깊은 복사를 해야한다.)
- `x.clone().getClass() == x.getClass()` 식도 참이어야 한다.
  - 복사된 객체가 같은 클래스(타입)여야 한다는 뜻이다.
- `x.clone().equals(x)`는 참이어야 하지만, 필수는 아니다.
  - 복사된 객체가 논리적 동치는 일치해야 한다는 뜻이다. (다만, 필수는 아니다.)

### super.clone()

또한 `clone()`을 구현할 때 `super.clone()`을 사용하여 호출해야한다. 

이를 어길시, 상속받은 하위 클래스에서 `super.clone()`을 호출할 때 전혀 다른 결과가 나올수 있다.

(ex. 만약 상위 클래스의 clone이 자신의 생성자로 생성한 객체를 반환하게 되면 하위 클래스 또한 자신의 타입이 아닌 상위 클래스 타입 객체를 반환할 수 밖에 없게 된다.)

상위 클래스가 `super.clone()`을 사용하지 않았을 때 예시를 살펴보자.

```java
public class Test implements Cloneable {
    public Test() {
    }

    @Override
    public Test clone() throws CloneNotSupportedException {
        // super.clone을 사용하지 않고 new를 통해 인스턴스 반환
        return new Test();
    }
}
```

```java
public class TestA extends Test {
    public TestA() {
    }

    @Override
    public TestA clone() throws CloneNotSupportedException {
        // 형변환이 되지 않아서 java.lang.ClassCastException을 던지며 예외 발생
        return (TestA) super.clone(); 
    }
}
```

```java
TestA testA = new TestA();
TestA clone = testA.clone();
System.out.println("testA = " + testA);
System.out.println("clone = " + clone);
```

<center><img src="https://user-images.githubusercontent.com/36228833/222418660-84028898-713e-4691-9c59-2ca00856bc78.png"></center><br/>

자바는 공변 반환 타이핑을 지원하기 때문에 `super.clone()`을 통해 하위 클래스가 자기 자신의 타입을 반환해도 문제가 생기지 않는다.

> 공변 반환 타입(covariant return type)이란? 재정의한 메서드의 반환타입은 상위 클래스의 메서드가 반환하는 타입의 하위 타입일 수 있다.

** 단, `final`로 선언된 클래스라면 하위 클래스를 가질수 없기때문에 이러한 문제를 고민하지 않아도 된다. **

```java
public class Test implements Cloneable {
    public Test() {
    }

    @Override
    public Test clone() throws CloneNotSupportedException {
        return (Test) super.clone();
    }
}
```

```java
public class TestA extends Test {
    public TestA() {
    }

    @Override
    public TestA clone() throws CloneNotSupportedException {
        // 공변 반환 타입 지원
        // 이를 통해 클라이언트는 Object 타입을 TestA 타입으로 형변환 하지 않아도 된다.
        return (TestA) super.clone(); 
    }
}
```

### throws 대신 try-catch 사용

`throws` 대신 `try-catch`를 사용하여 예외를 위임하지말고 내부적으로 처리하자.

`Cloneable` 인터페이스를 구현한 이상 클라이언트는 `CloneNotSupportedException` 만날일이 절대 없다.

그러나 `CloneNotSupportedException`은 `checked exception`으로 구현되어 정상적으로 인터페이스를 구현한 경우에도 예외 처리를 위한 불필요한 코드를 추가해야한다.

```java
@Override
public TestA clone() {
    try {
        return (TestA) super.clone(); 
    } catch (CloneNotSupportedException e) {
        // Cloneable 인터페이스를 구현했으므로 해당 구문은 실행되지 않음.
        throw new AssertionError();
    }
}
```

`CloneNotSupportedException`가 발생할 일이 없으므로 불필요한 위임을 할 필요가 없다.

## 가변 상태 객체를 참조하는 클래스

모든 필드가 기본 타입(`primitive`)이거나 불변(`final`)이라면 위의 방법만으로 충분하다.

그러나 가변 상태 객체가 존재하는 경우 다음과 같은 문제가 발생한다.

> 가변 상태 객체란 인스턴스 생성 이후에도 내부 상태 변경이 가능한 객체를 뜻한다. (Array, List, Set 등)

다음은 자료구조중 하나인 `Stack`을 구현한 클래스 예시이다.

```java
public class Stack implements Cloneable {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if(size == 0) {
            throw new EmptyStackException();
        }

        Object result = elements[--size];
        elements[size] = null;
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }

    @Override
    public Stack clone() {
        try {
            return (Stack) super.clone(); // 가변 객체를 얕은 복사
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }

    // 확인을 위해 getter 추가
    public Object[] getElements() {
        return elements;
    }
}
```

여기서 `super.clone()`으로만 복제를 한다면 가변 객체인 `elements[]`는 얕은 복사가 일어나며 문제가 발생한다.

```java
Stack stack = new Stack();
stack.push("a");
stack.push("b");
stack.push("c");
Stack stackClone = stack.clone();

stack.push("d");
stack.push("e");

System.out.println("stack.pop() = " + stack.getElements()[4]); // e를 출력 한다.
System.out.println("stackClone.pop() = " + stackClone.getElements()[4]); // a, b, c 까지만 저장되있으므로 null이 나와야한다.
```

<center><img src="https://user-images.githubusercontent.com/36228833/222418780-9ec6f85b-1574-4644-8472-a4e800dc9050.png"></center><br/>

우리가 원하는 `clone()`은 원본 객체에 영향을 주지 않으면서 객체간의 불변식을 보장해야한다.

이를 해결할 가장 쉬운 방법은 가변 객체의 `clone()`을 재귀적으로 호출해 주는것이다.

```java
@Override
public Stack clone() {
    try {
        Stack result = (Stack) super.clone();
        result.elements = elements.clone();  // 원본 객체의 가변 객체 clone()을 사용하여 복제한 객체 내 가변 객체로 복사 (얕은복사)
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

단, 이와 같은 방법은 가변 객체가 `final`로 선언되어 있으면 새로운 값을 할당할 수 없기 때문에 불가능하다.

<center><img src="https://user-images.githubusercontent.com/36228833/222418878-85bacad3-a1df-45ac-9ad6-b90b7c394dd8.png"></center><br/>

위의 테스트를 그대로 진행한 결과 의도대로 결과가 출력된다.

하지만 이같은 방법도 결국은 얕은 복사이기 때문에 복잡한 가변 객체의 경우 완전한 독립을 보장하지 못한다.

다음은 복잡한 가변객체를 가진 클래스를 구현한 예시이다.


```java
public class TestC implements Cloneable {
    private int id;
    private TestC[] tests;

    public TestC(final int id, final TestC[] tests) {
        this.id = id;
        this.tests = tests;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public TestC[] getTests() {
        return tests;
    }

    public void setTests(final TestC[] tests) {
        this.tests = tests;
    }

    @Override
    public TestC clone() {
        try {
            TestC result = (TestC) super.clone();
            result.tests = tests.clone();
            return result;
        } catch (final CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

```java
TestC original = new TestC(
    11, new TestC[]{
        new TestC(111, null),
        new TestC(222, null)
    }
);
TestC clone = original.clone();
original.getTests()[0].setId(5);

System.out.println("original.getTests()[0].getId() = " + original.getTests()[0].getId());
System.out.println("clone.getTests()[0].getId() = " + clone.getTests()[0].getId()); // 111이 나와야 한다.
```

<center><img src="https://user-images.githubusercontent.com/36228833/222420038-042a03f8-8b1a-41a6-9cfd-713ed395e504.png"></center><br/>

이럴 경우 내부 가변 객체를 하나씩 비교해서 반환하는 방식으로 깊은 복사를 구현해야한다.

```java
@Override
public TestC clone() {
    try {
        TestC result = (TestC) super.clone();
        result.tests = deepCopy(result.tests);
        return result;
    } catch (final CloneNotSupportedException e) {
        throw new AssertionError();
    }
}

private TestC[] deepCopy(final TestC[] parent) {
    TestC[] clone = parent.clone();
    return Arrays.stream(clone)
        .map(original -> new TestC(original.id, original.hasTests() ? deepCopy(original.tests) : null))
        .toArray(TestC[]::new);
}

private boolean hasTests() {
    return tests != null && tests.length != 0;
}
```

수정한뒤 위의 테스트를 실행해보면 의도한대로 출력되는것을 확인할 수 있다.

<center><img src="https://user-images.githubusercontent.com/36228833/222420093-cd9b3925-d0d4-4e87-88ac-eb490987b24f.png"></center><br/>

그러나 재귀 함수를 사용시 가변 객체의 깊이가 깊을수록 쌓인 스택프레임으로 인한 스택 오버플로우를 유발할수 있다는 단점이 존재한다.

따라서 반복문으로 구현이 가능하다면 최대한 반복문으로 구현하는 것이 좋다.


## 주의할점

- 상속용 클래스에서는 `Cloneable`을 구현해서는 안된다.
  - `clone()`를 재정의하여 강제로 `CloneNotSupportedException`을 던지게하자.

  ```java
  @Override
  protected final Object clone() throws CloneNotSupportedException {
     throw new CloneNotSupportedException();
  }
  ```

- `Closeable` 인터페이스를 구현한 `thread-safe` 클래스를 작성할 때는 재정의하여 동기화를 해줘야한다.
  - `thread-safe` 하지 않기때문에 동시성 문제가 발생할 수 있다.
  - 필요하다면 직접 재정의하고 동기화 구문을 작성해야한다.

## clone() 대체 방법 (권장)

앞서 말한 과정들은 너무 복잡하고 위험한 부분들이 많다.

`Cloneable` 인터페이스를 이미 구현한 클래스는 어쩔수 없이 위의 방식대로 구현해야하지만

그렇지 않은 경우 더 나은 객체 복사 방식을 사용할 수 있다.

### 복사 생성자

말그대로 자신과 같은 클래스의 인스턴스를 매개변수로 받는 생성자를 말한다.

```java
public Test(Test test) {
    //....
}
```

### 복사 팩터리

복사 팩터리는 복사 생성자를 모방한 정적 팩터리 구현 방식이다. (아이템1 참고)

```java
public static Test newInstance(Test test){
    // ...
}
```

이 둘의 방식은 `clone()`을 사용하는 것보다 이러한 장점이 있다.

- 생성자를 쓰지않는 방식의 위험 천만한 객체 생성 메커니즘이 아니다.
  - `clone()`은 생성자를 통하지 않고 인스턴스를 생성한다.
- 엉성한 일반 규약에 기대지 않는다.
  - 여러 위험한 상황에 비해 일반 규약의 제약이 너무 약하다.
- 정상적인 `final` 필드 용법과 충돌하지 않는다.
- 불필요한 예외를 던지지 않는다.
  - 절대 걸리지 않을 `CloneNotSupportedException`를 던지지 않는다.
- 불필요한 형변환을 하지 않는다.
  - `super.clone()`으로 인한 형변환을 하지 않아도 된다.

## 정리

- 새로운 인터페이스를 만들때 `Cloneable` 확장을 해서는 안된다.
  - 새로운 클래스도 이를 구현해서는 안된다.
  - 대신 복사 생성자, 복사 팩터리 방법을 사용하자.
- 성능 최적화 관점에서 검토 후 별다른 문제가 없을때만 드물게 허용하자.
- 단, 배열만은 `clone()` 방식이 가장 깔끔한 방식이다.


## 📣 Reference

[Effective Java 3/E - Joshua J. Bloch](http://www.yes24.com/Product/Goods/65551284)<br/>
[WegraLee/effective-java-3e-source-code](https://github.com/WegraLee/effective-java-3e-source-code/tree/master/src/effectivejava/chapter3/item13)<br/>
[(이펙티브 자바 3판) 3장 - 모든 객체의 공통 메서드, clone 재정의는 주의해서 진행해라](https://perfectacle.github.io/2018/12/16/effective-java-ch03-item13-clone-method/)<br/>
[이펙티브 자바, 쉽게 정리하기 - item 13. clone 재정의는 주의해서 진행하라](https://jake-seo-dev.tistory.com/31)<br/>
