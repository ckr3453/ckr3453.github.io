---
title: "이펙티브자바 - 아이템16) public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라."
categories: 
    - book
date: 2023-03-06
last_modified_at: 2023-03-06
toc: true
toc_sticky: true
excerpt: "getter, setter 활용하기"
---

## 개요

공개 api의 경우 내부 필드를 `public`으로 열어 놓으면 캡슐화([아이템15](https://ckr3453.github.io/book/effective-java_item15/#%EA%B0%9C%EC%9A%94))의 장점들을 취할 수 없다.

```java
public class Member {
  public Long id;
  public String name;
  public int age;
}
```

이런 경우 불변식을 보장할수 없고 필드에 접근할 때 부수 작업을 수행하도록 강제할 수 없다. (외부에서 필드에 직접 접근이 가능하므로)

이번 아이템에선 내부 필드 접근권한을 `public`으로 열어놓는 대신 `private`으로 바꾸고 접근자(`getter`)를 활용하는 것을 권장한다.

## 접근자(+변경자) 활용하기

방법은 매우 간단하다. 위에서 기술한것과 같이 `public` 으로 선언된 필드의 접근 권한을 `private`으로 변경한 뒤 접근자인 `getter`를 추가해 주자.

필요에 따라 필드 별 변경자(`setter`)를 추가하여 활용할 수 있다.

```java
public class Member {
  // public -> private으로 변경
  private Long id;
  private String name;
  private int age;

  // 필드 별 getter 추가
  public Long getId(){
    return id;
  }

  public String getName(){
    return name;
  }

  public int getAge(){
    return age;
  }

  // 필드 별 setter 추가
  public Long setId(){
    return id;
  }

  public String setName(){
    return name;
  }

  public int setAge(){
    return age;
  }
}
```

접근자와 변경자를 제공함으로써 클래스 내부 표현 방식을 언제든 바꿀수 있는 유연함을 가져갈 수 있다.

## 불변으로 선언한 필드

그렇다면 필드를 불변으로 선언하면 해결될까? 그렇지 않다.

`final`로 필드를 선언하여 불변임을 보장한다고 하더라도 api를 변경하지 않고는 표현 방식을 바꿀수없고, 

필드에 접근할때 특정 작업을 수행할 수 없다는 단점은 여전히 존재한다.

```java
public class Member {
  
  // 불변이어도 외부에서 직접 접근이 가능하므로 여전히 단점은 존재한다!
  public final Long id;

  private String name;
  private int age;

  public String getName(){
    return name;
  }

  public int getAge(){
    return age;
  }
}
```

## 필드 공개가 가능한 경우

`package-private` 클래스 혹은 `private` 중첩 클래스라면 데이터 필드를 노출한다고 해도 문제가 되지 않는다.

위의 경우에 한해 오히려 이 방식이 `getter` 방식보다 훨씬 깔끔하다.

`package-priavte` 클래스의 경우, 이 클래스를 포함하는 패키지 내부에서만 동작 가능하며 

`private` 중첩 클래스 또한 해당 클래스를 포함하는 가장 바깥의 클래스까지로 수정 범위가 한정되기 때문에

이는 곧 클라이언트의 코드 범위가 그만큼 제한된다는 의미이므로 경우에 따라선 `public`으로 필드를 선언하는게 더 나은 경우가 있다.

```java
class Member {

    public Long id;
    public String name;
    public int age;
    
    private static class SocialSecurityNumber {
        public Long number;
    }
    
    public void doSomething(){
        SocialSecurityNumber socialSecurityNumber = new SocialSecurityNumber();
        socialSecurityNumber.number = id * age * 12345;
    }
}
```


## 정리

- 공개 api 즉, `public` 클래스라면 절대 가변 필드를 `public`으로 선언하면 안된다.
  - 불변 필드가 상대적으로 덜 위험하긴 하지만 여전히 단점은 존재한다.

## 📣 Reference

[Effective Java 3/E - Joshua J. Bloch](http://www.yes24.com/Product/Goods/65551284)<br/>
[WegraLee/effective-java-3e-source-code](https://github.com/WegraLee/effective-java-3e-source-code/tree/master/src/effectivejava/chapter3/item16)<br/>