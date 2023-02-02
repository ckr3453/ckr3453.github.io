---
title: "이펙티브자바 - 아이템2) 생성자에 매개변수가 많다면 빌더를 고려하라."
categories: 
    - book
date: 2023-02-01
last_modified_at: 2023-02-02
toc: true
toc_sticky: true
excerpt: "생성자 대신 빌더를 고려하라"
---

## 개요

클래스를 작성할 때 "선택적" 매개변수가 많다면 생성자로 모든 경우의 수를 작성해야 하기 때문에 관리하기 힘들다.

실제로 생성자를 작성하게 된다면 다음과 같을 것이다.

```java
// 영양정보 클래스
public class NutritionFacts {
    private final int servingSize;  // (ml, 1회 제공량 정보)   필수
    private final int servings;     // (회, 총 n회 제공량 정보) 필수
    private final int calories;     // (1회 제공량당 칼로리)    선택
    private final int fat;          // (g/1회 제공량 지방)     선택
    private final int sodium;       // (mg/1회 제공량 나트륨)  선택
    private final int carbohydrate; // (g/1회 제공량 탄수화물)  선택

    // 점층적 생성자 패턴
    public NutritionFacts(int servingSize, int servings){
        this(servingSize, servings, 0);
    }

    // 점층적 생성자 패턴
    public NutritionFacts(int servingSize, int servings, int calories){
        this(servingSize, servings, calories, 0);
    }

    // 점층적 생성자 패턴
    public NutritionFacts(int servingSize, int servings, int calories, int fat){
        this(servingSize, servings, calories, fat, 0);
    }

    // 점층적 생성자 패턴
    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium){
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    // 점층적 생성자 패턴
    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate){
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}

// 각 매개변수가 의미하는 바가 무엇인지 식별하기 어렵다.
NutritionFacts coke = new NutritionFacts(240, 8, 100, 0, 35, 27); 

```

위의 같이 점층적으로 선택 매개변수를 모두 받게끔 작성하는 방식을 점층적 생성자 패턴(telescoping constructor pattern)이라고 부른다.

그러나 이러한 방식으로 생성자를 작성할 경우 **매개변수가 많아지면 많아질수록 코드를 작성하거나 읽기 어려워질것**이다.

그렇다면 setter를 활용하는 방식인 자바빈즈 패턴(JavaBeans pattern)은 어떨까?

```java
// 영양정보 클래스
public class NutritionFacts {
    private int servingSize = -1;  // (ml, 1회 제공량 정보)   필수
    private int servings    = -1;     // (회, 총 n회 제공량 정보) 필수
    private int calories    = 0;     // (1회 제공량당 칼로리)    선택
    private int fat         = 0;          // (g/1회 제공량 지방)     선택
    private int sodium      = 0;       // (mg/1회 제공량 나트륨)  선택
    private int carbohydrate= 0; // (g/1회 제공량 탄수화물)  선택

    public NutritionFacts(){ }

    public void setServingSize(int value) {servingSize = value;}
    public void setServings(int value) {servings = value;}
    public void setCalories(int value) {calories = value;}
    public void setFat(int value) {fat = value;}
    public void setSodium(int value) {sodium = value;}
    public void setCarbohydrate(int value) {carbohydrate = value;}
}

NutritionFacts coke = new NutritionFacts(); 
coke.setServingSize(240);
coke.setServings(8);
coke.setCalories(100);
coke.setSodium(35);
coke.setCarbohydate(27);
```

점층적 생성자 패턴을 사용했을 때에 비해 가독성은 좋아졌으나 **메서드를 여러개 호출해야하고 분리된 상태이기 때문에 일관성이 무너지는 단점**이 있다.

## 빌더 패턴을 사용해보자

점층적 생성자 패턴과 자바빈즈 패턴의 장점을 모두 취한 방법인 빌더 패턴을 사용해보자.

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    // 빌더 패턴용 정적 멤버 클래스
    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수 - 기본값으로 초기화한다.
        private int calories      = 0;
        private int fat           = 0;
        private int sodium        = 0;
        private int carbohydrate  = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings    = servings;
        }

        public Builder calories(int val) { 
            calories = val;
            return this; // 메서드 체이닝 가능
        }

        public Builder fat(int val) { 
            fat = val;
            return this; // 메서드 체이닝 가능
        }

        public Builder sodium(int val){ 
            sodium = val;
            return this; // 메서드 체이닝 가능
        }

        public Builder carbohydrate(int val) { 
            carbohydrate = val;
            return this; // 메서드 체이닝 가능
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        // 정적 멤버 클래스인 Builder 로부터 값을 받아와 초기화 함.
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
        sodium       = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}

// 필수값에 대한 일관성과 setter 메서드 체이닝을 활용하여 식별이 용이하다.
NutritionFacts coke = new NutritionFacts.Builder(240, 8) // 필수 매개변수
                .calories(100)      // 선택 매개변수
                .sodium(35)         // 선택 매개변수
                .carbohydrate(27)   // 선택 매개변수
                .build();
```

클래스 내부에 정적 멤버 클래스를 만들어두어 생성자와 setter를 동시에 활용함으로써 코드를 읽고 쓰는게 훨씬 쉬워졌다.

## 계층 구조에 활용 가능한 빌더 패턴

> 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다.

```java
import java.util.*;

public abstract class Pizza {
    public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
    final Set<Topping> toppings;

    // 재귀적 타입 한정 (Recursive type parameter)
    // 즉, 임의의 타입 T 와 Builder<T>를 상속 받는 객체까지만 허용한다.
    abstract static class Builder<T extends Builder<T>> { 
        
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();          // 하위 클래스에 메서드 체이닝이 가능하게끔 유도
        }

        abstract Pizza build();

        // 시뮬레이트한 셀프 타입(simulated self-type) 관용구인 self()를 활용한다.
        // 하위 클래스는 이 메서드를 재정의(overriding)하여 "this"를 반환하도록 해야 한다.
        protected abstract T self();
    }
    
    // Builder 타입을 매개변수로 받고있는 Pizza 생성자
    // Pizza를 상속받은 하위클래스는 Pizza.Builder 타입의 인자를 전달해야한다.
    // 그렇기 때문에 자식 클래스는 Pizza.Builder 타입을 상속받는 Builder 클래스를 만들어야 한다.
    Pizza(Builder<?> builder) {    
        toppings = builder.toppings.clone(); // 자기의 빌더 타입으로부터 생성한 토핑 set을 클론하여 저장한다.
    }
}
```

```java
import java.util.Objects;

// 뉴욕 피자 - 계층적 빌더를 활용한 하위 클래스
public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size; 

    // Pizza.Builder<Builder> 에서 <Builder> 는 NyPizza의 Builder를 의미
    public static class Builder extends Pizza.Builder<Builder> {    
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size); // 뉴욕 피자는 크기가 필수 매개변수이다.
        }

        @Override 
        public NyPizza build() { 
            return new NyPizza(this); 
        }

        @Override 
        protected Builder self() { 
            // 부모 클래스에서 addToping 이후 self()를 호출하고 있기 때문에 상속 받은후 자기의 Builder로 return 받게 재정의하여
            // 메서드 체이닝이 가능하도록 구현하고 있다.
            return this; 
        }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }

    @Override 
    public String toString() {
        return toppings + "로 토핑한 뉴욕 피자";
    }
}
```

```java
// 칼초네 피자 - 계층적 빌더를 활용한 하위 클래스
public class Calzone extends Pizza {
    private final boolean sauceInside; 

    // Pizza.Builder<Builder> 에서 <Builder> 는 Calzone의 Builder를 의미
    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false; 

        public Builder sauceInside() {
            sauceInside = true; // 칼초네 피자는 소스 유무가 선택 매개변수이다.
            return this;
        }

        @Override 
        public Calzone build() {
            return new Calzone(this);
        }

        @Override 
        protected Builder self() { 
            // 부모 클래스에서 addToping 이후 self()를 호출하고 있기 때문에 상속 받은후 자기의 Builder로 return 받게 재정의하여
            // 메서드 체이닝이 가능하도록 구현하고 있다.
            return this; 
        }
    }

    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }

    @Override 
    public String toString() {
        return String.format("%s로 토핑한 칼초네 피자 (소스는 %s에)", toppings, sauceInside ? "안" : "바깥");
    }
}
```

```java
NyPizza pizza = new NyPizza.Builder(SMALL)  // 뉴욕 피자는 피자 size를 필수 매개변수로 받는다.
                .addTopping(SAUSAGE)        // 상속받은 부모클래스인 Pizza의 공통 메서드로써 선택 매개변수로 받는다.
                .addTopping(ONION)          // 상속받은 부모클래스인 Pizza의 공통 메서드로써 선택 매개변수로 받는다.
                .build();
Calzone calzone = new Calzone.Builder()
                .addTopping(HAM)            // 상속받은 부모클래스인 Pizza의 공통 메서드로써 선택 매개변수로 받는다.
                .sauceInside()              // 칼초네 피자는 소스 유무를 선택 매개변수로 받는다.
                .build();
```

## 빌더 패턴의 단점
- 빌더 객체를 무조건 생성해야 하기 때문에 생성 비용이 발생한다.
- 점층적 생성자 패턴에 비해 코드가 장황하여 매개변수가 최소 4개이상은 되어야 값어치를 한다.
  - 그러나 시간이 지날수록 매개변수는 많아질 확률이 높기때문에 애초에 빌더 패턴으로 시작하는것이 낫다.

## 정리
> **생성자나 정적 팩토리가 처리해야할 매개변수가 많다면 빌더 패턴을 선택하는게 더 낫다.** 매개변수 중 다수가 필수가 아니거나 같은 타입이면 특히 더 그렇다. 빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고 자바빈즈보다 훨씬 안전하다.


## 📣 Reference

[Effective Java 3/E - Joshua J. Bloch](http://www.yes24.com/Product/Goods/65551284)<br/>
[WegraLee/effective-java-3e-source-code](https://github.com/WegraLee/effective-java-3e-source-code/tree/master/src/effectivejava/chapter2/item2)<br/>
[생성자에 매개변수가 많으면 빌더를 고려하라 #2](https://github.com/java-squid/effective-java/issues/2#issuecomment-694834862)<br/>
[#03 #02의 보충설명 - 계층적으로 설계된 클래스와 빌더패턴](https://debaeloper.tistory.com/35)<br/>
[자바 제네릭에서 T extends Builder T는 어떤 의미인가요](https://okky.kr/articles/842260)<br/>

