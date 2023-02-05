---
title: "이펙티브자바 - 아이템4) 인스턴스화를 막으려거든 private 생성자를 사용하라."
categories: 
    - book
date: 2023-02-05
last_modified_at: 2023-02-05
toc: true
toc_sticky: true
excerpt: "생성자를 통한 인스턴스화 방지"
---

## 개요

util용 클래스 처럼 정적 메서드, 정적 필드만을 담은 클래스를 만들 때가 있다. (ex. `java.lang.Math`, `java.util.Arrays`,..)

이러한 유틸성 클래스들은 인스턴스로 만들어 쓰려고 설계한게 아니기 때문에 생성자가 필요없다. 그러나 생성자를 명시하지 않으면 컴파일러가 자동으로 내부에 기본 public 생성자를 만들어 준다. 

사용자 입장에선 생성자가 의도적으로 있는것인지 컴파일러에 의해 자동생성된 것 인지 알길이 없다.

```java
public class ExcelUtil {
    ...
}
```
<center><img src="https://user-images.githubusercontent.com/36228833/216815843-1a993737-24f8-43d2-b5cf-a2ca83c5668c.png"></center><br/>
<center><img src="https://user-images.githubusercontent.com/36228833/216815965-45311922-50b0-457d-8c30-09671aae8311.png"></center><br/>

이런 혼동을 막기위해 의미없는 인스턴스화를 막을 방법이 필요하다.

## private 생성자 사용

생성자의 접근제한자를 private 으로 추가하면 클래스의 인스턴스화를 막을 수 있다.

```java
public class ExcelUtil {
    
    // 인스턴스화 방지용
    private ExcelUtil(){}
}
```

<center><img src="https://user-images.githubusercontent.com/36228833/216815899-72ae4b3f-0f51-4138-bc01-44c00f237606.png"></center><br/>
<center><img src="https://user-images.githubusercontent.com/36228833/216815911-585d7901-6a78-4f7e-b904-e8705548afaa.png"></center><br/>

코드의 직관성을 위해 private 생성자 코드 앞에 적절한 주석을 추가하자.

다음 결과와 같이 명시적 생성자가 private 이므로 더이상 클래스 외부에서는 접근할 수 없다. 또한 생성자를 접근할 수 없게 되었으니 상속을 불가능하게 하는 효과도 있다. (상속하게되면 상위 클래스의 생성자를 호출해야 하므로)

## 📣 Reference

[Effective Java 3/E - Joshua J. Bloch](http://www.yes24.com/Product/Goods/65551284)<br/>
