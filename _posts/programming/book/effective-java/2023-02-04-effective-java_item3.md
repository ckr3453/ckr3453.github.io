---
title: "이펙티브자바 - 아이템3) private 생성자나 열거 타입으로 싱글턴임을 보증하라."
categories: 
    - book
date: 2023-02-04
last_modified_at: 2023-02-05
toc: true
toc_sticky: true
excerpt: "싱글턴 패턴을 보장하는 방식들"
---

## 싱글턴이란

인스턴스를 오직 하나만 생성할 수 있는 클래스를 의미하며 인스턴스가 필요할 때 **다른 인스턴스를 만들지 않고 기존의 인스턴스를 재활용** 함으로써 메모리 낭비를 방지할 수 있다.

싱글턴을 만드는 방식은 보통 `public static final` 필드를 활용한 방식과 정적 팩터리 메서드를 활용한 방식 두가지로 나뉜다.

두가지 방식을 사용했을 때의 단점과 그 단점을 보완한 세번째 방식을 소개한다.

## public static final 필드 방식

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {} // 클래스 내부에서만 접근가능
}
```

```java
public class Main {
    public static void main(String[] args) {
        Elvis test1 = Elvis.INSTANCE;
        Elvis test2 = Elvis.INSTANCE;

        System.out.println("test1 = " + test1);
        System.out.println("test2 = " + test2);
        System.out.println(test1.equals(test2));
    }
}
```

<center><img src="https://user-images.githubusercontent.com/36228833/216814951-89a753d6-92a8-4563-9f98-74b98d3f5ba7.png"></center><br/>


생성자가 private 이므로 Elvis.INSTANCE 객체를 초기화 할때 딱 한번만 호출된다. 그로인해 Elvis.INSTANCE는 전체 시스템에서 단 하나뿐인 객체임이 보장된다.

단, 예외는 있다. 

권한이 있는 클라이언트가 리플렉션 API인 `AccessibleObject.setAccessible`을 사용하면 private 생성자를 임의로 호출할 수 있다. 

```java
public class Main {
    public static void main(String[] args) {
        Elvis elvis = Elvis.INSTANCE;
        System.out.println("elvis = " + elvis);

        try {
            Constructor<Elvis> c = Elvis.class.getDeclaredConstructor(); // 클래스 내부에 생성자 객체를 가져온다.
            c.setAccessible(true); // private 접근 제한을 해제
            Elvis newElvis = c.newInstance();
            System.out.println("newElvis = " + newElvis);
        } catch (NoSuchMethodException | InstantiationException | IllegalAccessException | InvocationTargetException e) {
            throw new RuntimeException(e);
        }
    }
}
```

<center><img src="https://user-images.githubusercontent.com/36228833/216814974-2ba2e3a2-bb43-423e-959c-a14c65a29238.png"></center><br/>

이러한 예외를 방지하려면 두번째 객체가 생성되려고 할때 생성자에서 예외를 던지게 하면 된다.

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {
        if(INSTANCE != null){
            throw new IllegalStateException("이미 생성된 인스턴스가 존재합니다.");
        }
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Elvis elvis = Elvis.INSTANCE;
        System.out.println("elvis = " + elvis);

        try {
            Constructor<Elvis> c = Elvis.class.getDeclaredConstructor();
            c.setAccessible(true);
            Elvis newElvis = c.newInstance(); // InvocationTargetException 발생
            System.out.println("newElvis = " + newElvis);
        } catch (NoSuchMethodException | InstantiationException | IllegalAccessException | InvocationTargetException e) {
            throw new RuntimeException(e);
        }
    }
}
```

<center><img src="https://user-images.githubusercontent.com/36228833/216814991-7cb40f6e-4d75-4bbd-a25e-a220cd7b8a8c.png"></center><br/>

이러한 방식의 장점은 다음과 같다.

- public static 필드가 final 이므로 해당 클래스가 싱글턴임이 API에 명백히 드러난다.
- 복잡하지 않고 간결하다.

## 정적 팩터리 메서드 방식

다음은 정적 팩터리 메서드를 활용한 두번째 방식이다.

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() {}
    public static Elvis getInstance() { return INSTANCE; }
}
```

Elvis.getInstance() 를 통해 항상 같은 객체의 참조를 반환하므로 다른 인스턴스는 만들어지지 않는다. (리플렉션 API 예외 또한 동일)

정적 팩터리 메서드 방식의 장점은 다음과 같다.

- API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.

    ```java
    public class Elvis {
        private static final Elvis INSTANCE = new Elvis();
        private Elvis() {}
        public static Elvis getInstance() { return new Elvis(); } // INSTANCE가 아닌 new 로 반환하여 싱글턴을 해제한다.
    }
    ```

- 정적 팩터리를 제네릭 싱글턴 팩터리로 만들수 있다.

    > 제네릭 싱글턴 팩터리란? 제네릭으로 타입설정 가능한 인스턴스를 만들어두고, 반환 시에 제네릭으로 받은 타입을 이용해 타입을 결정하는 것

- 정적 팩터리의 메서드 참조를 공급자(Supplier)로 사용할 수 있다.
    ```java
    public class Elvis {
        private static final Elvis INSTANCE = new Elvis();
        private Elvis() {}

        public static Elvis getInstance() { return INSTANCE; }

        public static void main(String[] args) {
            // 함수형 인터페이스로, 'get()' 추상 메서드를 사용할 수 있다.
            // get() 은 지연 연산을 지원한다.
            Supplier<Elvis> instance = Elvis::getInstance; 
            Elvis elvis = instance.get();
        }
    }
    ```

이러한 장점들이 필요하지 않다면 public static final 필드 방식을 사용하는 것이 좋다.

## 직렬화

위의 두가지 방식으로 만든 싱글턴 클래스를 직렬화하면 싱글턴을 보장하지 않는 상태로 변한다. (직렬화를 통해 초기화해둔 인스턴스가 아닌 다른 인스턴스가 반환되기 때문)

역직렬화 과정에서 만들어진 인스턴스(readObject) 대신에 **기존에 생성된 싱글톤 인스턴스를 반환하도록** readResolve 메서드를 직접 정의하면 된다.

```java
public class Elvis implements Serializable {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() {}
    public static Elvis getInstance() { return INSTANCE; }

    // 접근 지정자는 반드시 private으로 해야한다.
    // 역직렬화 과정에서 호출되는 readObject로 생성되는 인스턴스는 가비지 컬렉터의 대상이 된다.
    private Object readResolve() throws ObjectStreamException {
        return INSTANCE;
    }
}
```

## 열거 타입(Enum) 방식 (권장)

```java
public enum Elvis {
    INSTANCE;
}
```

```java
public class Main {
    public static void main(String[] args) {
        Elvis elvis = Elvis.INSTANCE;
        System.out.println("elvis = " + elvis);
    }
}
```

가장 간단하고 쉬운 방식이며 추가적인 로직 없이 직렬화를 할 수 있고 리플렉션 공격에도 제2의 인스턴스가 생기는 일을 완벽하게 막아준다.

단, 상속받아야 하는 클래스가 존재할 경우에는 사용할 수 없으니 주의하자.

## 📣 Reference

[Effective Java 3/E - Joshua J. Bloch](http://www.yes24.com/Product/Goods/65551284)<br/>
[WegraLee/effective-java-3e-source-code](https://github.com/WegraLee/effective-java-3e-source-code/tree/master/src/effectivejava/chapter2/item3)<br/>
[[Java] 리플렉션 API : 클래스, 필드, 메서드 정보 조회](https://sas-study.tistory.com/275)<br/>
[Constructor 클래스의 getConstructor 와 getDeclaredConstructor 차이 비교](https://emflant.tistory.com/52)<br/>
[Java Pattern: Use Atomic Boolean to Return Single Usage Object](https://helloacm.com/java-pattern-use-atomic-boolean-to-return-single-usage-object/)<br/>
[[이펙티브 자바] 아이템 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라](https://web2eye.tistory.com/228)<br/>
[직렬화 프록시 패턴이란](https://pamyferret.tistory.com/m/58)<br/>
[자바 직렬화: readResolve와 writeReplace](https://madplay.github.io/post/what-is-readresolve-method-and-writereplace-method)<br/>
