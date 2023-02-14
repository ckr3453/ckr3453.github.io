---
title: "이펙티브자바 - 아이템9) try-finally 보다는 try-with-resources를 사용하라."
categories: 
    - book
date: 2023-02-13
last_modified_at: 2023-02-15
toc: true
toc_sticky: true
excerpt: "try-with-resources를 통한 자원 관리"
---

## 개요

자바 라이브러리는 `close()` 를 호출하여 직접 닫아줘야 하는 자원이 많다. 

(ex. `InputStream`, `OutputStream`, `java.sql.Connection` 등)

자바에서 이러한 자원들을 닫는 수단으로 `try-finally`를 많이 사용한다.

그러나 직접 호출해야하는 특성때문에 사용자가 자원을 닫지않고 놓치는 경우가 많아 성능 문제로 이어지기도 한다.

또한 안전망 역할로 `finalizer`를 활용한다고 해도 `finalizer` 자체가 안정적이지 않기때문에 믿음직스럽지 않다.

이번 아이템에선 이러한 `try-finally` 방식 대신 `try-with-resources` 방식을 통한 안전한 자원 관리 방법을 제안한다.

## 예외 발생시 마지막 stack trace만 출력

`try-finally`는 `try` 블록에서 자원을 사용하고 `finally` 블록에서 자원의 `close()`를 직접 호출하여 자원을 닫아주는 방식이다.

문제는 예외 발생에 있다.

예외는 `try` 블록과 `finally` 블록 모두 발생할 수 있으며, 만약 2개의 블록 모두 예외 발생 시 `finally` 블록에서 발생한 예외만 출력된다.

다음의 예시를 살펴보자.

```java
public class MyResource implements AutoCloseable {

    public void hello(){
        System.out.println("hello!");
        throw new IllegalArgumentException();
    }

    @Override
    public void close() {
        System.out.println("close resource.");
        throw new IllegalStateException();
    }
}
```

```java
public static void main(String[] args) {
    MyResource myResource = new MyResource();
    try {
        myResource.hello(); // 예외 발생함.
    } finally {
        myResource.close(); // 예외 발생함.
    }
}
```

<center><img src="https://user-images.githubusercontent.com/36228833/218789995-c417f88f-8bed-4e1d-b0a3-486a1d1f0556.png"></center><br/>

`MyResource.hello()`가 실행되어 `IllegalArgumentException` 예외가 발생하였으나 출력되지 않고 `close()`의 `IllegalStateException`만 출력되는 것을 확인할 수 있다.

이렇게 되면 첫번째 예외 내역이 남지 않아 문제가 발생했을 경우 찾기가 쉽지 않다.

## 코드 가독성 하락

자원 하나만 사용했을 경우에는 가독성이 나쁘지 않다. 하지만 2개 이상의 자원을 사용한다면 다음과 같은 형태가 될 것이다.

```java
public static void main(String[] args) {
    MyResource myResource = new MyResource();
    try {
        myResource.hello();
        MyResource myResource2 = null;
        try {
            myResource2 = new MyResource();
            myResource2.hello();
        } finally {
            if(myResource2 != null){ // myResource2를 초기화 하는 시점에 오류가 발생하면 NPE가 발생할 수 있으므로 null 체크 필요
                myResource2.close();
            }
        }
    } finally {
        myResource.close();
    }
}
```

단순히 2개의 자원만 사용했음에도 구조가 상당히 복잡하여 가독성이 떨어지게 된다.

## try-with-resources를 사용하자.

`try-with-resources`는 위의 나열된 단점을 모두 보안한 방식이다. 

(`AutoCloseable` 인터페이스를 구현한 클래스만 사용 가능하다.)

위의 예제를 `try-with-resources` 방식으로 바꾸면 이렇게 된다.

```java
public static void main(String[] args) {
    try(MyResource myResource = new MyResource()){
        myResource.hello();
    }
}
```

`close()`를 직접 호출할 필요가 없어지고 코드가 간결해져서 읽기 쉬워졌다.

<center><img src="https://user-images.githubusercontent.com/36228833/218790151-88d7d065-5e9b-4350-a0d0-794ca7a63c74.png"></center><br/>


또한 다음과 같이 예외에 따른 stack trace도 순서에 맞게 제대로 출력되는 것을 확인할 수 있다. 

(심지어 뒤에 나올 예외까지 `suppressed`라는 표시와 함께 전부 나온다.)

2개 이상의 자원을 다룰때도 다음과 같이 작성하면 된다.

```java
public static void main(String[] args) {
    try (MyResource myResource = new MyResource();
         MyResource myResource2 = new MyResource()){
         myResource.hello();
         myResource2.hello();
    }
}
```

<center><img src="https://user-images.githubusercontent.com/36228833/218791346-42041459-35d0-438c-84e5-b6c9beef6537.png"></center><br/>


마찬가지로 예외 발생에 따른 stack trace가 순서에 맞게 잘 출력된 것을 확인할 수 있다.

추가로 `suppressed`된 예외들은 `Throwable.getSuppressed()`를 통해 코드상에서 가져올수도 있다.

```java
public static void main(String[] args) {
    try (MyResource myResource = new MyResource();
         MyResource myResource2 = new MyResource()){
         myResource.hello();
         myResource2.hello();
    } catch (Exception e){
        if(e.getSuppressed().length > 0){
            Arrays.stream(e.getSuppressed()).forEach(System.out::println);
        }
    }
}
```

<center><img src="https://user-images.githubusercontent.com/36228833/218790223-11981996-a075-4a12-a93e-c027027d8dcb.png"></center><br/>


## 정리

> 꼭 회수해야 하는 자원을 다룰 때는 try-finally 보다 try-with-resources 방식을 사용하자.


## 📣 Reference

[Effective Java 3/E - Joshua J. Bloch](http://www.yes24.com/Product/Goods/65551284)<br/>
[WegraLee/effective-java-3e-source-code](https://github.com/WegraLee/effective-java-3e-source-code/tree/master/src/effectivejava/chapter2/item9)<br/>
[An example of how suppressed exceptions in Java work](https://www.theserverside.com/tutorial/OCPJP-OCAJP-Java-7-Suppressed-Exceptions-Try-With-Resources)<br/>