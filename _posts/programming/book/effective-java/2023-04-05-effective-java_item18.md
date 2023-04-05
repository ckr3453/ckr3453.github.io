---
title: "이펙티브자바 - 아이템18) 상속보다는 컴포지션을 사용하라."
categories: 
    - book
date: 2023-04-05
last_modified_at: 2023-04-05
toc: true
toc_sticky: true
excerpt: "상속보다는 컴포지션을 사용하라."
---

## 개요

우리가 일상에서 사용하던 상속은 몇가지 치명적인 단점을 가지고 있다.

이번 아이템에서는 상속의 문제점들을 알아보고 상속의 대안인 컴포지션을 사용하는 방법을 알아본다.

## 상속은 캡슐화를 깨뜨린다.

- 상위 클래스의 구현이 하위 클래스에게 노출되기 때문이다.
- 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작이 달라질수 있다.
- 결국 자식 클래스는 상위 클래스에게 높은 결합도와 의존성을 지니게 된다.

예시를 통해 알아보자.

```java
public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;

    public InstrumentedHashSet() {
    }

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override 
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override 
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
        s.addAll(List.of("하나", "둘", "셋", "야!"));
        System.out.println(s.getAddCount()); // 4? 8?
    }
}
```

`InstrumentedHashSet`는 `HashSet`을 상속받은 클래스로써 `add`와 `addAll` 메서드를 재정의하였다.

main 메서드에서 다음과 같이 4개의 원소를 지닌 리스트를 `addAll` 을 통해서 더한다면 `addCount`는 값이 어떻게 될까?

4가 나와야 할것 같지만 8이 나온다.

그 이유는 재정의한 `addAll` 메서드가 내부적으로 `HashSet`이 상속받는 `AbstractCollection` 클래스가 정의한 `addAll` 메서드를 호출하기 때문이다.

```java
public abstract class AbstractCollection<E> implements Collection<E> {
  // .. 생략

  public boolean add(E e) {
      // addAll이 얘를 호출하는게 아님
      throw new UnsupportedOperationException();
  }

  public boolean addAll(Collection<? extends E> c) {
      boolean modified = false;
      for (E e : c)
          if (add(e)) // HashSet을 상속받아 재정의한 add 메서드를 호출함
              modified = true;
      return modified;
  }
}
```

여기에서 `addAll`은 내부적으로 `add(e)`를 호출하게 되는데 해당 `add(e)`는 추상 클래스를 구현한 메서드를 호출하게 되므로 

`InstrumentedHashSet`이 재정의한 `add`를 호출함으로써 `addCount`가 결과적으로 한번더 더해지게 되어 원하지 않는 결과가 나오게 된다.

이처럼 상속을 할 경우 상위 클래스의 구현에 따라 하위 클래스의 동작이 의도치 않게 바뀔수 있음을 확인했다.

그렇다면 이러한 문제점은 어떻게 해결할 수 있을까?

- 문제가 되는 `addAll`을 재정의하지 않는다?
  - 그러나 부모 클래스인 `HashSet`의 내용이 변경이 된다면 원하지 않는 동작을 수행할 수 있다.
- addAll을 다른 방식으로 재정의한다?
  - 상위 클래스의 메서드를 다시 구현해야 하므로 그에 대한 어려움이 동반된다.
  - 잘못 구현했을경우 오류가 발생하거나 성능이 저하될 수 있다.
  - 또한 상위 클래스가 `addAll`과 관련하여 `private` 필드를 사용할 경우 구현자체가 불가능해진다.
- 하위 클래스에 새로운 메서드를 추가하여 사용한다?
  - 다음 릴리즈때 재수없게(?) 상위 클래스에서 추가한 메서드가 새롭게 추가한 메서드와 시그니처가 같고 반환 타입이 다르다면 컴파일 조차 되지않는다.
    > 메서드 시그니처란? 메서드명, 파라미터의 순서, 타입, 개수를 의미한다. 

해당 문제들을 피하여 개발하기 위한 묘안으로 컴포지션을 권장한다.

## 상속 대신 컴포지션을 사용하자.

컴포지션을 구성하기 위해 전달 클래스를 작성한다.

새로 만들 클래스인 전달 클래스는 private 필드로 상위 클래스의 인스턴스를 참조하게 하고 

내부에는 상위 클래스의 대응하는 메서드들을 호출하여 단순히 그 결과를 반환하는 메서드들로 구성한다.

기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 뜻에서 이러한 설계를 컴포지션이라고 한다.

이로써 기존 클래스는 전달 클래스를 통하여 기능을 호출하게 되므로 **상위 클래스의 내부 구현 방식에서 벗어날 수 있게된다.**

```java
public class ForwardingSet<E> implements Set<E> { // 유연성을 위해 HashSet의 모든 기능을 정의한 Set을 구현

    private final Set<E> s; // private 으로 인스턴스 변수를 가지고 생성자에서 매개변수로 받는다.
    public ForwardingSet(Set<E> s) {this.s = s;}

    public void clear()               { s.clear();            }
    public boolean contains(Object o) { return s.contains(o); }
    public boolean isEmpty()          { return s.isEmpty();   }
    public int size()                 { return s.size();      }
    public Iterator<E> iterator()     { return s.iterator();  }
    public boolean add(E e)           { return s.add(e);      }
    public boolean remove(Object o)   { return s.remove(o);   }
    public boolean containsAll(Collection<?> c)
                                   	  { return s.containsAll(c); }
    public boolean addAll(Collection<? extends E> c)
    								                  { return s.addAll(c);      }
    public boolean removeAll(Collection<?> c)
                                   	  { return s.removeAll(c);   }
    public boolean retainAll(Collection<?> c)
                                   	   { return s.retainAll(c);   }
    public Object[] toArray()          { return s.toArray();  }
    public <T> T[] toArray(T[] a)      { return s.toArray(a); }
    @Override public boolean equals(Object o)
                                       { return s.equals(o);  }
    @Override public int hashCode()    { return s.hashCode(); }
    @Override public String toString() { return s.toString(); }

}
```

전달 클래스는 `Set`을 구현하고 있기 때문에 `Set`을 구현하는 모든 객체가 활용할 수 있다. (재사용성, 유연성)

`InstrumentedHashSet`는 컴포지션을 위해 생성한 전달 클래스를 상속받아 다음과 같이 구현한다.

```java
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        // 컴포지션을 위해 Set 객체를 상위 클래스에 전달
        super(s);
    }

    @Override 
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override 
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        // HashSet 말고도 Set을 구현한 구현체는 모두 가능
        // InstrumentedSet<String> s = new InstrumentedSet<>(new TreeSet<>()); 
        InstrumentedSet<String> s = new InstrumentedSet<>(new HashSet<>()); 

        s.addAll(List.of("하나", "둘", "셋", "야!"));
        System.out.println(s.getAddCount()); // 4
    }
}
```

새롭게 작성한 `InstrumentedSet`는 이제 `addAll`를 호출했을 때 `AbstractCollection`의 `addAll`이 아닌 생성자로 넘겨받은 `HashSet`의 `addAll`을 호출하게 된다.

다른 `Set` 인스턴스를 감싸고 있다는 뜻에서 `InstrumentedSet` 같은 클래스를 **래퍼 클래스**라고 하며,

다른 `Set`에 계측 기능을 덧씌운다는 의미로 **데코레이터 패턴**이라고도 한다.

## 정리

- 상속은 캡슐화를 깨뜨리므로 취약점을 피하기 위해 컴포지션을 사용하자.
- 그럼에도 상속을 사용해야 한다면 다음 질문을 자문해보자.
  - 확장하려는 클래스의 API에 아무런 결함이 없는가?
  - 결함이 있다면 이 결함이 클래스의 API까지 전파돼도 괜찮은가?
  - 상속은 특성상 상위 클래스의 API를 **결함까지 포함**하여 그대로 승계한다.


## 📣 Reference

[Effective Java 3/E - Joshua J. Bloch](http://www.yes24.com/Product/Goods/65551284)<br/>
[WegraLee/effective-java-3e-source-code](https://github.com/WegraLee/effective-java-3e-source-code/tree/master/src/effectivejava/chapter4/item18)<br/>