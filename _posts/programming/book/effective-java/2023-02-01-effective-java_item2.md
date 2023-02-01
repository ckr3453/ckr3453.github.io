---
title: "이펙티브자바 - 아이템2) 생성자에 매개변수가 많다면 빌더를 고려하라. (작성중)"
categories: 
    - book
date: 2023-02-01
last_modified_at: 2023-02-01
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
    private final int servingSize;  // (ml, 1회 제공량 정보)   필수
    private final int servings;     // (회, 총 n회 제공량 정보) 필수
    private final int calories;     // (1회 제공량당 칼로리)    선택
    private final int fat;          // (g/1회 제공량 지방)     선택
    private final int sodium;       // (mg/1회 제공량 나트륨)  선택
    private final int carbohydrate; // (g/1회 제공량 탄수화물)  선택

    public NutritionFacts(){ }

    public void setServingSize(int value) {servingSize = value;}
    public void setServings(int value) {servings = value;}
    public void setCalories(int value) {calories = value;}
    public void setFat(int value) {fat = value;}
    public void setSodium(int value) {sodium = value;}
    public void setCarbohydrate(int value) {carbohydrate = value;}
}
```

## 빌더를 사용하자


## 📣 Reference
[Effective Java 3/E - Joshua J. Bloch](http://www.yes24.com/Product/Goods/65551284)<br/>
[WegraLee/effective-java-3e-source-code](https://github.com/WegraLee/effective-java-3e-source-code/tree/master/src/effectivejava/chapter2/item2)<br/>
