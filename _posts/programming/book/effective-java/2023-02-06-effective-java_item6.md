---
title: "이펙티브자바 - 아이템6) 불필요한 객체 생성을 피하라."
categories: 
    - book
date: 2023-02-06
last_modified_at: 2023-02-06
toc: true
toc_sticky: true
excerpt: "기존 객체를 재활용해야 한다면 새로운 객체를 만들지 마라."
---

## 개요

동일한 기능을 하는 객체는 매번 생성하지 말고 객체 하나를 재사용하는 편이 낫다.

객체를 매번 생성하는 경우와 재사용 하는 경우를 각 예시 별 차이점을 알아보자.

## String 예시

String 인스턴스 생성 과정을 살펴보자.

```java
String s3 = new String("Hi"); // new 연산자로 생성 (Bad)
String s4 = "Hi"; // String literal로 생성 (Good)
```

문자열 인스턴스를 생성할때 다음과 같이 두가지 방식이 존재하는데 차이점은 **매번 새로운 객체를 생성하는 것에 대한 유무** 차이이다.

<center><img src="https://user-images.githubusercontent.com/36228833/216913622-1b2d3545-ba1b-4a05-ae87-1b68c7f6b17d.png"></center><br/>

`new` 연산자를 통해 생성할 경우 JVM의 Heap 메모리 영역에 객체가 새롭게 생성되고 

String literal로 생성할 경우 Heap 메모리 영역 내 String Pool에 생성된다. 

String literal로 생성한 객체의 값이 이미 String Pool에 존재한다면, 해당 객체는 String Pool의 reference를 참조하여 재활용한다.(=캐싱) 

반면에 `new` 연산자로 생성한 객체는 String Pool에 같은 값이 존재하더라도 Heap 메모리 영역 내 별도의 객체를 생성하여 가리키게 된다.

이러한 차이점은 반복문 안에서 호출했을 때 더 확실히 나타난다.

```java
long beforeTime = System.currentTimeMillis();
for(int i = 0; i<Integer.MAX_VALUE; i++){
    String s = new String("Hi");
}
long afterTime = System.currentTimeMillis();
long secDiffTime = afterTime - beforeTime;
System.out.println("new 연산자 시간소요(ms) : "+secDiffTime);

beforeTime = System.currentTimeMillis();
for(int i=0; i<Integer.MAX_VALUE; i++){
    String s = "Hi";
}
afterTime = System.currentTimeMillis()
secDiffTime = afterTime - beforeTime;
System.out.println("String literal 시간소요(ms) : "+secDiffTime);
```

<center><img src="https://user-images.githubusercontent.com/36228833/216928330-9f49b7ff-e3be-490f-81a8-fa856932d337.png"></center><br/>

`new` 연산자보다 String literal 로 생성한 경우가 압도적으로 빠른것을 확인할 수 있다.

## 정규표현식 예시

다음은 정규표현식을 사용하는 경우다.

```java
public class RomanNumerals {
    static boolean isRomanNumeralSlow(String s) {
        return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
                + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    }
}
```

`String.matches` 는 정규표현식으로 문자열 형태를 확인하는 간결한 방법이다. 

하지만 내부적으로 정규표현식용 `Pattern` 인스턴스를 **한번만 쓰고 버리기 때문에** 반복해서 사용하기에는 적합하지 않다. 

`java.lang.String`<br/>
<center><img src="https://user-images.githubusercontent.com/36228833/216928501-8e0944e1-2816-4c39-9ed2-70fc3941f224.png"></center><br/>

`java.util.regex.Pattern`<br/>
<center><img src="https://user-images.githubusercontent.com/36228833/216928571-dc25b7be-6dc7-49c7-b41e-61c24fea834e.png"></center><br/>

성능을 개선하려면 Pattern 인스턴스를 미리 초기화 과정에서 생성하여 캐싱해두고 재사용하는 방식으로 수정하면 된다.

```java
public class RomanNumerals {
    // 캐싱을 위해 미리 생성해둔다.
    private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD]|D?C{0,3})"
            + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeral(String s) {
        // 캐싱되어있는 패턴 값을 가져와서 사용한다.
        return ROMAN.matcher(s).matches();
    }
}
```

실제로 속도 테스트를 진행한 결과는 다음과 같다.

```java
long beforeTime = System.currentTimeMillis();
for(int i = 0; i<1000000; i++){
    RomanNumerals.isRomanNumeralSlow("test");
}
long afterTime = System.currentTimeMillis();
long secDiffTime = afterTime - beforeTime;
System.out.println("정규표현식 시간소요(ms) : "+secDiffTime);

beforeTime = System.currentTimeMillis();
for(int i=0; i<1000000; i++){
    RomanNumerals.isRomanNumeral("test");
}
afterTime = System.currentTimeMillis();
secDiffTime = afterTime - beforeTime;
System.out.println("정규표현식 개선후 시간소요(ms) : "+secDiffTime);

```

<center><img src="https://user-images.githubusercontent.com/36228833/216928706-5f3f4b79-2d91-49d6-b83d-55352e447670.png"></center><br/>

캐싱을 활용하여 개선한 쪽이 훨씬 빠르다.

## 오토박싱 (Auto Boxing) 예시

마지막으로 오토박싱을 예로 들수있다.

> 오토박싱이란? 프로그래머가 기본 타입(primitive)과 박싱된 기본 타입을 섞어 쓸 때 자동으로 상호 변환해주는 기술이다.

오토박싱으로 인하여 **새로운 객체를 생성할 때** 비용이 발생한다.

다음은 모든 양의 정수 총합을 구하는 메서드 이다.

```java
Long sum = 0L;
for(long i = 0; i < Integer.MAX_VALUE; i++){
    sum += i; // Long에 long을 더할 때 새로운 Long 객체가 생성되기 때문에 속도 차이가 발생한다.
}
```

`Long` 타입인 `sum`에 `i`를 계속해서 더함으로써 의미 없는 `Long` 인스턴스가 계속해서 생성되는 것이다.

`sum`의 타입을 `long` 으로 수정한다면 의미 없는 인스턴스를 생성하지 않게되어 성능이 개선될 것이다.

실제로 개선 전과 개선 후의 속도차이를 비교해보자.

```java
long beforeTime = System.currentTimeMillis();
Long sumSlow = 0L;
for(long i =0; i<Integer.MAX_VALUE; i++){
    sumSlow += i;
}
long afterTime = System.currentTimeMillis();
long secDiffTime = afterTime - beforeTime;
System.out.println("오토박싱 시간소요(ms) : "+secDiffTime);


beforeTime = System.currentTimeMillis();
long sum = 0L;
for(long i =0; i<Integer.MAX_VALUE; i++){
    sum += i;
}
afterTime = System.currentTimeMillis();
secDiffTime = afterTime - beforeTime;
System.out.println("개선 후 시간소요(ms) : "+secDiffTime);
```

<center><img src="https://user-images.githubusercontent.com/36228833/216928782-f139e594-0f70-4415-99e9-7f7f8754d2f3.png"></center><br/>

박싱된 기본 타입보다는 기본 타입(primitive)을 사용하고, **의도치 않은 오토박싱이 숨어들지 않도록 신경쓰자.**

## 📣 Reference

[Effective Java 3/E - Joshua J. Bloch](http://www.yes24.com/Product/Goods/65551284)<br/>
[WegraLee/effective-java-3e-source-code](https://github.com/WegraLee/effective-java-3e-source-code/tree/master/src/effectivejava/chapter2/item6)<br/>
[Java String Pool](https://www.javastring.net/java/string/pool)<br/>
[[JAVA] String = " " vs new String(" ") 의 차이](https://ict-nroo.tistory.com/18)<br/>
[String Constant Pool이란? | Java String Pool](https://starkying.tistory.com/entry/what-is-java-string-pool)<br/>
[Java Autoboxing 자동 변환 주의점](https://johngrib.github.io/wiki/use-java-primitive-type-for-performance/)<br/>
