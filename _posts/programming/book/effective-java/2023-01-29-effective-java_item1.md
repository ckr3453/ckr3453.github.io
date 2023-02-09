---
title: "이펙티브자바 - 아이템1) 생성자 대신 정적 팩터리 메서드를 고려하라."
categories: 
    - book
date: 2023-01-29
last_modified_at: 2023-01-31
toc: true
toc_sticky: true
excerpt: "생성자 대신 정적 팩터리 메서드를 사용했을 때 장단점"
---

## 개요

클래스의 객체(인스턴스)를 생성할때 일반적인 수단은 다음과 같이 public 생성자를 통해 얻는 것이다. 

```java
public class Job {
    private int salary;
    private int workHours;

    public Job(int salary, int workHours){
        this.salary = salary;
        this.workHours = workHours;
    }
}
```

그러나 클래스는 생성자와는 별도로 **정적 팩터리 메서드(static factory method)**를 제공할 수 있다. <br/>

정적 팩터리 메서드의 역할은 직접적으로 생성자를 통해 객체를 생성하는 것이 아닌 **메서드를 통해서 객체를 생성**하는 것이다.

```java
public class Job {
    
    private int salary;
    private int workHours;

    private Job(int salary, int workHours){
        this.salary = salary;
        this.workHours = workHours;
    }

    // 정적 팩토리 메서드
    public static Job newDoctor(){
        return new Job(15000, 10);
    }

    // 정적 팩토리 메서드
    public static Job newDeveloper(){
        return new Job(5000, 13);
    }
}
```

그렇다면 생성자 대신 정적 팩토리 메서드를 사용했을 때 얻을 수 있는 장점과 단점에 대해 알아보자.

## 장점
### 1. 이름을 가질 수 있다.
생성자로 객체를 만드는 경우에는 생성자명은 필연적으로 해당 클래스명을 따라가게 되어있다. 때문에 생성자 자체 만으로는 반환될 객체의 특성을 제대로 설명할 수 없다.

```java
// 생성자명만 봤을 때 어떤 직업인지 구분할 수 없다.
Job doctor = new Job(15000, 10);     
Job developer = new Job(5000, 13); 
```

물론 생성자의 매개변수를 바꿔서 생성자를 늘리는 방식도 있지만 갯수가 많아질수록 기억하기 어렵고 결국엔 설명 문서를 보지않고선 의미를 알 수 없게 된다.

그러나 정적 팩토리 메서드는 다음과 같이 각각의 특성이 잘 드러나는 이름으로 지어줄 수 있다.

```java
Job doctor = Job.newDoctor();
Job developer = Job.newDeveloper();
```

### 2. 호출할 때마다 인스턴스를 새로 생성하지 않아도 된다.
정적 팩토리 메서드는 애플리케이션 시작부터 종료까지 메모리 `static` 영역에 메모리가 할당되어 있으므로 **인스턴스를 따로 생성할 필요가 없다.**

생성 비용이 큰 객체가 자주 요청되는 상황이라면 이같은 방식은 성능 향상에 큰 효과를 누릴 수 있다.

```java
public static Boolean valueOf(boolean b){
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```

```java
Boolean bool = new Boolean(true);       // 매번 생성할 필요 X
Boolean bool = Boolean.valueOf(true);   // O
```

이와 같이 반복되는 요청에 같은 객체를 반환하는 식으로 인스턴스를 철저히 통제하는 클래스를 **인스턴스 통제(instance-controlled)** 클래스라고 한다.

인스턴스를 통제하면 다음과 같은 이점이 있다.

1. 클래스를 싱글턴 혹은 인스턴스화 불가로 만들 수 있다.
  - private을 활용하여 외부에서 생성자 접근을 못하도록 막는다.
2. 불변 값 클래스(String, Boolean, Integer,..)에서 동일값을 가진 인스턴스가 단 하나뿐임을 보장할 수 있다.
  - a == b 일때만 a.equals(b)가 성립한다.
  - 즉, a와 b의 주소값이 같을 때만 실제 값도 같을 수 있다.

### 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

정적 팩토리 메소드를 사용하면 하위 클래스의 인스턴스를 반환할 수 있다.<br/>

새로운 인터페이스와 수많은 구현 클래스가 있을 때, 구현 클래스의 생성자로 인스턴스를 만드는게 아니라 인터페이스의 정적 팩토리 메소드로 인스턴스를 만들어서 <br/>

개발자가 **수많은 구현 클래스들을 이해하지 않고도 인터페이스를 사용할 수 있도록** 할 수 있다.

```java
// Collections를 통해서 list, map, set 객체 생성이 가능함.
List<String> list = Collections.singletonList("list");
Map<Object, Object> map = Collections.emptyMap();
Set<Object> set = Collections.emptySet();
```

### 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다.

클래스의 생성자는 해당 클래스의 인스턴스만 만들 수 있지만 정적 팩토리 메소드를 사용하면 **같은 메소드라도 상황에 따라 다른 클래스 인스턴스**를 반환할 수 있다. 

매개 변수에 맞는 적절한 인스턴스를 반환하여 자원이 낭비되는 것을 막을 수 있다.

예시로 `java.util.EnumSet` 클래스의 `noneOf` 메소드를 살펴보자.

```java
// 지정한 elementType을 사용하여 빈 enumSet을 만듭니다.
public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
    Enum<?>[] universe = getUniverse(elementType);
    if (universe == null)
        throw new ClassCastException(elementType + " not an enum");

    if (universe.length <= 64)
        return new RegularEnumSet<>(elementType, universe); // long 변수 하나로 관리 하는 RegularEnumSet
    else 
        return new JumboEnumSet<>(elementType, universe); // long 배열로 관리하는 JumboEnumSet
}
```

```java
public enum RAINBOW {
    RED, ORANGE, YELLOW, GREEN, BLUE
}
```

```java
EnumSet<RAINBOW> = EnumSet.noneOf(RAINBOW.class);
```

> 클라이언트는 팩토리가 건네주는 객체가 어느 클래스의 인스턴스인지 알 수도 없고 알 필요도 없다. 해당 코드에서는 EnumSet의 하위 클래스이기만 하면 된다.

### 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
생성자는 클래스가 존재해야 하지만 정적 팩토리 메서드를 작성할 때는 타입만 적고 실제 반환될 클래스는 나중에 구현해도 된다.

```java
public static CardApp payment(String cardType){
    if("국민".equals(cardType)){
        return new KookminCardApp();
    } else if("하나".equals(cardType)){
        return new HanaCardApp();
    } else if("신한".equals(cardType)){
        CardApp shinhanApp = null;
        try {
            // 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다. (Java Reflection 활용)
            Class<?> class = Class.forName("com.app.test.ShinhanApp")
            Constructor<?> constructor = class.getConstructor();
            shinhanApp = (CardApp) constructor.newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return shinhanApp;
    }

    throw new IllegalArgumentException();
}
```

## 단점
### 1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
정적 팩토리 메서드"만" 사용하려면 기존 생성자를 private으로 막아야하고 그렇게 되면 상속을 받을수 없게 된다. 

### 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.
생성자의 경우 기존 클래스명을 따라가고 또한 용도가 명확히 드러나기 때문에 사용하기 쉽다. 하지만 정적 팩터리 메서드의 경우 해당 클래스 내에 존재한다는 것을 작성하지 않으면 찾기 힘들다. 

때문에 정적 팩토리 메서드명을 널리 알려진 규약에 따라 짓는 식으로 문제를 완화해야한다.

## 정적 팩터리 메서드 명명 방식들
```java
// from: 매개변수 하나를 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드
Date d = Date.from(param);

// of: 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);

// valueOf: from과 of의 더 자세한 버전
BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE)

// instance, getInstnce: 매개변수에 맞는 인스턴스를 반환하지만 같은 인스턴스임을 보장하지 않는다.
StackWalker luke = StackWalker.getInstance(param);

// create, newInstance: 매번 새로운 인스턴스를 생성해서 반환하는 것을 보장한다.
Object newArray = Array.newInstance(classObject, arrayLen);

// getType: getInstnce와 같으나 다른 클래스의 인스턴스를 반환 (getType에서 Type은 팩터리 메서드가 반환할 객체의 타입)
FileStore fs = Files.getFileStore(path);

// newType: newInstnce와 같으나 다른 클래스의 인스턴스를 반환 (newType에서 Type은 팩터리 메서드가 반환할 객체의 타입)
BufferedReader br = Files.newBufferedReader(path);

// type: getType, newType와 유사
List<ParamType> list = Collections.list(param);
```

## 정리
> 정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있으니 상대적인 장/단점을 이해하고 사용하는것이 좋다. 그렇다 하더라도 정적 팩터리를 사용하는게 유리한 경우가 더 많으므로 **무작정 public 생성자를 제공하던 습관이 있으면 고치자.**

## 📣 Reference
[Effective Java 3/E - Joshua J. Bloch](http://www.yes24.com/Product/Goods/65551284)<br/>
[[이펙티브 자바 / 예제 코드 추가] 아이템 1. 생성자 대신 정적 팩토리 메소드를 고려해라.](https://jaeseongdev.github.io/development/2021/01/05/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C_%EC%9E%90%EB%B0%94_%EC%95%84%EC%9D%B4%ED%85%9C_1/)<br/>
[[Java 궁금증] Class.forName()은 어떻게 동작할까?](https://kyun2.tistory.com/23)<br/>
[[Java] Class.forName()에 대해서](https://jongminlee0.github.io/2019/06/29/reflection/)<br/>
[2-1) 생성자 대신 정적 팩토리 메소드를 고려하라.](https://nankisu.tistory.com/87)<br/>
[자바 클래스 동적 로딩 - Java Class.forName newInstance, 이클립스(Eclipse)](https://carrotweb.tistory.com/53)<br/>
