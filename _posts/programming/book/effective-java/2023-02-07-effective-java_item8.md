---
title: "이펙티브자바 - 아이템8) finalizer와 cleaner 사용을 피하라."
categories: 
    - book
date: 2023-02-06
last_modified_at: 2023-02-07
toc: true
toc_sticky: true
excerpt: "finalizer와 cleaner 사용을 지양하자."
---

## 개요

자바의 객체 소멸은 가비지 컬렉터가 담당한다. 그 외에도 `finalizer`(java 8)와 `cleaner`(java 9부터)라는 자원 반납을 위한 두개의 객체 소멸자를 따로 제공한다. 

그러나 두가지 모두 **예측할 수 없고, 느리고, 상황에따라 위험할 수 있기 때문**에 기본적으로 '사용하지 말것'을 권장하고 있다.

왜 기본적으로 쓰지 말아야 하는지 알아보자.

## 수행 시점 및 수행 여부를 보장하지 않는다.

소멸 대상이 된 객체(객체에 접근 할 수 없게됨)를 대상으로 `finalizer`나 `cleaner`가 실행되기까지 얼마나 걸릴지 알 수 없다.

즉, 제때 실행되어야 하는 작업은 절대 할 수 없다. (ex. 파일 리소스 반납 등)

`finalizer`와 `cleaner`의 수행 속도는 전적으로 **가비지 컬렉터 알고리즘**에 달렸다.(가비지 컬렉터 구현마다 천차만별)

`finalizer`와 `cleaner`가 수행되리라는 보장도 없다. 시스템이 종료될때까지 소멸대상 객체를 소멸 시키지 못할수도 있다는 의미다.

> `finalizer`의 스레드 우선순위가 기본적으로 낮기때문에 다른 로직이 수행되면 실행 순서가 자연스레 뒤로 밀린다. `cleaner`는 별도의 스레드로 동작하여 우선순위를 높게할 수 있어서 `finalizer` 보다는 나은 상황이지만 백그라운드에서 실행된다는 점은 변함이 없기때문에 언제 처리 될지는 장담할 수 없다.

결국 최악의 경우 자원을 반납하지 못한 인스턴스가 계속 쌓이다가 `OutOfMemoryException`이 발생할 수 있다.

실행을 보장하는 `System.runFinalizersOnExit`와 `Runtime.runFinalizersOnExit` 메서드가 존재하지만 치명적 결함으로 인하여 deprecated 되었다.

> System.runFinalizersOnExit( )는 deprecated 되었다. 가장 큰 이유는 rechable한 객체를 finalize를 하는게 말도 안된다는 것이고, 또다른 이유는 finalize의 순서가 보장되지 않기 떄문이다.

```java
public class FinalizerExample {

    @Override
    protected void finalize() throws Throwable {
        System.out.println("clean up"); // 언제 수행될지 보장하지 않음. (=실행이 아예 안될수도 있음)
    }

    public void printHello(){
        System.out.println("hello");
    }
}
```

```java
public class Main {

    public static void main(String[] args) throws InterruptedException {
        Main main = new Main();
        main.run();

        TimeUnit.SECONDS.sleep(10); // 일정시간 대기해도 finalizer가 수행되지 않는다.
    }

    private void run(){
        FinalizerExample finalizerExample = new FinalizerExample();
        finalizerExample.printHello();
    }
}
```

<center><img src="https://user-images.githubusercontent.com/36228833/217519681-14c6b78b-c4b3-4be4-a8cd-a34614e6b3f7.png"></center><br/>

## 예외 발생 무시 (`finalizer` 한정)

`finalizer` 동작 중 발생한 예외는 무시되며 처리할 작업이 남아있어도 그 순간 바로 종료된다.

보통 예외가 발생하면 스레드를 중단 시키고 stack trace를 출력하지만, `finalizer`에서 예외 발생 시 경고 조차 출력하지 않는다.

잡지 못한 예외 때문에 훼손된 객체가 남아있어서 어떻게 동작할 지 예측할 수 없게 된다.

(단, `cleaner`는 사용하는 라이브러리가 자신의 스레드를 통제하기 때문에 이런 문제가 발생하지 않는다.)

## 성능 문제

`AutoCloseable` 객체를 생성하고 `try-with-resource`로 자원 반납까지 걸린시간 : 12ns

`finalize()`를 구현한 객체가 자원 반납까지 걸린시간 : 550ns

속도가 무려 50배 정도 차이난다. (`cleaner`도 클래스의 모든 인스턴스를 수거하는 형태이므로 `finalizer`와 비슷한 성능을 낸다.)

## 보안 문제 (`finalizer` 공격)

`finalizer`를 사용한 클래스는 심각한 보안 문제를 일으킬 수 있다. `finalizer`를 상속받은 클래스가 생성자나 직렬화 과정에서 예외가 발생하면 해당 객체의 `finalizer` 가 강제로 수행될 수 있다.

이 `finalizer`는 정적 필드에 자신의 참조를 할당하여 가비지 컬렉터가 수집하지 못하게 막을 수 있다.

```java
public class Dashboard {

    int value;

    public Dashboard(int value) {
        if(value < 1)
            throw new IllegalStateException("1보다 작은 숫자는 허용하지 않습니다."); // 예외가 발생하여 객체 생성 실패 즉, gc의 대상이 됨.
        this.value = value;
    }

    public int getValue() {
        return value;
    }

    public void print() {
        System.out.println("Hi!");
    }
}
```

```java
public class FinalizerAttack extends Dashboard {

    public static Dashboard dashboard;

    public FinalizerAttack(int value) {
        super(value);
    }

    @Override
    protected void finalize() throws Throwable {
        // Finalizer 로 인하여 Dashboard 객체를 주입받을 수 있게 된다.
        // static 변수에 주입된다 -> 객체가 다시 접근 가능하게 됨 -> Dashboard가 GC의 대상에서 벗어나게 되어서 소멸되지 못함.
        dashboard = this;
    }

    public static void main(String[] args) {
        try {
            // 일부러 생성자 예외를 발생시킨다.
            FinalizerAttack finalizerAttack = new FinalizerAttack(-1);
        } catch (Exception e) {
            System.out.println(e);
        }

        System.gc();        // 가비지 컬렉터 강제 수행
        System.runFinalization(); // finalizer 강제 수행

        if (dashboard != null) {
            // 소멸 되어야할 객체가 소멸 되지 않고 메서드를 수행하게 된다.
            System.out.println("dashboard = " + dashboard);
            System.out.println("dashboard.getValue() = " + dashboard.getValue());
            dashboard.print();
        }
    }
}
```

<center><img src="https://user-images.githubusercontent.com/36228833/217519826-3cd7640f-6610-4652-b9ab-d05bf58ea469.png"></center><br/>

이 공격으로부터 방어를 하려면 아무 동작을 하지않는 `finalize()`에 `final` 키워드를 선언함으로써 하위 클래스가 오버라이딩 하려는것을 막으면 된다.

```java
public class Dashboard {

    int value;

    public Dashboard(int value) {
        if(value < 1)
            throw new IllegalStateException("1보다 작은 숫자는 허용하지 않습니다.");
        this.value = value;
    }

    @Override
    protected final void finalize() throws Throwable {
        // 하위 클래스가 상속받더라도 finalize는 오버라이딩 하지 못한다.
    }

    public int getValue() {
        return value;
    }

    public void print() {
        System.out.println("Hi!");
    }
}
```

## 자원 반납을 위해 `AutoCloseable`을 사용하자 (권장)

앞서 설명한 `finalizer`, `cleaner` 대신 파일이나 스레드 등 자원 반납을 위해 `AutoCloseable`을 사용하자.

자원 반납이 필요한 클래스에 `AutoCloseable` 인터페이스를 구현하고 `close()`를 명시적으로 호출하면 된다. 

```java
public class Test implements AutoCloseable{

    @Override
    public void close() throws Exception {
        System.out.println("Test.close");
    }

    public void hi(){
        System.out.println("Test.hi");
    }
}
```

```java
public class Main {
    public static void main(String[] args) throws Exception {
        Test test = null;
        try {
            test = new Test();
            test.hi();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if(test != null){
                test.close();
            }
        }
    }
}
```

`try-with-resource`를 사용하면 `close()`를 명시하지 않아도 `try` 블록이 끝날때 자동으로 호출해준다.

```java
public class Main {
    public static void main(String[] args) throws Exception {
        try(Test test = new Test()){
            test.hi();
        } // 이후 test.close() 실행됨
    }
}
```

<center><img src="https://user-images.githubusercontent.com/36228833/217519947-cd066fad-96c4-4b50-8343-ca972f095dbd.png"></center>


## finalizer와 cleaner를 자원 반납의 안전망으로 활용

그렇다면 `finalizer`와 `cleaner`은 어떤 상황에서 쓸 수 있을까?

사용자가 (`try-with-resource`를 사용하지 않았을 때) `close()`를 명시하지 않았을 때 `finalize()` 를 구현하여 `close()`를 강제로 수행하는 방법이다.

사용자가 미처 자원을 회수하지 못한 경우를 대비하여 `finalize()`가 대신 `close()`를 수행해 줌으로써 자원에 대한 안전망 역할을 하는것이다.

추가로, 안전망을 구현할 때 자원이 반납이 되었는지에대한 여부를 추적하여 예외처리를 하는것이 좋다.

```java
public class Test implements AutoCloseable {

    // 자원이 반납되었는지 여부
    private boolean closed;

    @Override
    public void close() throws Exception {
        if(this.closed)
            throw new IllegalStateException("이미 자원이 반납되었습니다.");
            
        closed = true;
        System.out.println("Test.close");
    }

    @Override
    protected void finalize() throws Throwable {
        if(!this.closed) close();
    }

    public void hi(){
        System.out.println("Test.hi");
    }
}
```

`cleaner`나 `finalizer`가 끝까지 호출되리라는 보장은 없지만 사용자가 하지않은 자원 회수를 늦게나마 해주는 편이 아예 회수를 안하는 것보다 낫다.

실제로 자바에서 제공하는 `FileInputStream`, `FileOutputStream`, `ThroeadPoolExecutor`, `java.sql.Connection` 에는 안전망 역할의 `finalizer`가 있다.

## `finalizer`와 `cleaner`를 `native` 객체를 정리하는데에 활용

`native` 객체는 일반적인 객체가 아니므로 가비지 컬렉터가 그 존재를 모른다. `native` 객체가 들고있는 리소스가 중요하지 않고 성능상 영향이 크지 않은 자원이라면 `cleaner`나 `finalizer`를 사용해서 해당 자원을 반납할 수 있다.

만약 중요한 리소스인 경우에는 `AutoCloseable`의 `close()`를 사용하는 것이 좋다.

## cleaner 예제

cleaner를 직접 구현한 예시를 살펴보자.

```java
public class CleanerExample implements AutoCloseable {

    private static final Cleaner CLEANER = Cleaner.create();

    private final CleanerRunner cleanerRunner; // clean 작업을 수행할 별도의 쓰레드가 필요함.

    private final Cleaner.Cleanable cleanable;

    public CleanerExample(int resources) {
        this.cleanerRunner = new CleanerRunner(resources);
        this.cleanable = CLEANER.register(this, cleanerRunner); // 인스턴스 clean 을 실제로 수행할 runner 를 등록함.
    }

    @Override
    public void close() throws Exception {
        cleanable.clean();
    }

    public void helloWorld(){
        System.out.println("CleanerExample.helloWorld");
    }

    private static class CleanerRunner implements Runnable {
        // 실제로 정리할 resource 가 여기 있어야함.
        // 단, 내부에 CleanerExample 타입의 인스턴스를 가져오면 순환 참조 오류가 발생하므로 유의할 것.
        int resources;

        public CleanerRunner(int resources) {
            this.resources = resources;
        }

        @Override
        public void run() {
            // 해당 인스턴스가 필요 없고 GC의 대상이 되었을 때 호출됨.
            System.out.println("Clean 작업 수행 (자원 반납)");
            resources = 0;
        }
    }
}
```

`cleaner`는 별도의 쓰레드가 필요하기 때문에 `Runnable`을 상속받아서 `run()`을 구현해야 한다.

실직적으로 자원을 반납하는 역할을 하는것은 `CleanerRunner`이며 내부 클래스 타입 인스턴스를 참조하면 순환 참조 오류가 발생하게 되므로 정적 중첩 클래스로 구현한다.

```java
public class Main {
    public static void main(String[] args) throws Exception {
        Main main = new Main();
        main.run(10);
        System.gc(); // 가비지 컬렉터를 강제로 수행시켜도 cleaner가 수행 되리란 보장은 없다. (이번엔 수행되었음)
    }

    private void run(int resources) {
        CleanerExample cleanerExample = new CleanerExample(resources);
        cleanerExample.helloWorld();
    }
}|
```

<center><img src="https://user-images.githubusercontent.com/36228833/217520092-bae818ad-a5c0-4b62-9840-49b0d926fb13.png"></center><br/>


결과에는 가비지 컬렉터를 강제로 수행시키면 `cleaner`가 동작하는것 처럼 보이나, 실제로는 수행을 보장하지 않으니 의존하지 말아야한다. (안전망 역할임을 유의하자.)

## 정리
- cleaner(자바 8까지는 finalizer)는 안전망 역할이나 중요하지 않은 네이티브 자원 회수용으로만 사용하자.
- 물론 이마저도 불확실성과 성능 저하에 유의해야한다.

## 📣 Reference

[Effective Java 3/E - Joshua J. Bloch](http://www.yes24.com/Product/Goods/65551284)<br/>
[WegraLee/effective-java-3e-source-code](https://github.com/WegraLee/effective-java-3e-source-code/tree/master/src/effectivejava/chapter2/item8)<br/>
[[이팩티브 자바] #8 Finalizer와 Cleaner 쓰지 마세요](https://www.youtube.com/watch?v=sdPdpMYqW_k)<br/>
[Finalizer attack](https://yangbongsoo.tistory.com/8)<br/>
[Effective Java - 객체의 생성과 소멸](https://brunch.co.kr/@oemilk/122)<br/>
[[아이템 8] finalizer와 cleaner 사용을 피하라](https://javabom.tistory.com/74)<br/>
[[Effective Java] 아이템8 - finalizer와 cleaner 사용을 피하라](https://mongsil1025.github.io/book/effective-java/item8/)<br/>
[어떻게 이런 FINALIZE()를 쓰란 말이에요](http://mkseo.pe.kr/blog/?p=1029)<br/>
[Why is the runFinalizersOnExit method in class System deprecated?](https://coderanch.com/t/368114/java/runFinalizersOnExit-method-class-System-depricated)<br/>