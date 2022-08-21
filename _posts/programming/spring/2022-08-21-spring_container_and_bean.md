---
title: "스프링 컨테이너와 빈 (IoC, DI)"
categories: 
    - Spring
date: 2022-08-21
last_modified_at: 2022-08-21
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
---

## 스프링 컨테이너란? (Spring Container)
스프링 프레임워크는 빈(Bean)을 생성하고 관리하는 컨테이너를 제공한다. 여기서 말하는 빈이란 스프링 컨테이너에 의해 관리되는 자바 객체를 의미한다.

그렇다면 왜 스프링 컨테이너를 사용하는 것일까? 스프링 컨테이너는 제어의 역전(IoC)이라는 개념과 이러한 개념을 구현한 의존관계 주입(DI)을 통해 장점들을 제공합니다.

### 제어의 역전 (Inversion of control)
- 문자 그대로 객체의 호출을 개발자가 아닌 외부에서 결정하는 것을 의미하는 "개념"이다.
- IoC는 스프링 프레임워크에만 적용된 개념이 아니라 서블렛 컨테이너 등 다른 프레임워크에서도 많이 사용된다.
- 객체의 생성 및 직접적인 호출을 컨테이너가 담당함으로써 개발자는 **의미없는 변경을 최소화 할 수 있고 비즈니스 로직에만 집중**할 수 있다.
- **변경에 유연한 코드 구조를 설계하여 적용할 수 있다.**

### 의존관계 주입 (Dependency injection)
- IoC의 "개념"을 스프링 컨테이너가 "구현" 하여 제공한다. (IoC != DI)
- 객체의 의존관계를 외부에서 주입시키는 디자인 패턴을 의미한다.
- 이러한 의존관계 주입은 3가지의 방법이 있다. (생성자 주입, Setter 주입, 인터페이스 주입)
- 모의 객체를 주입할 수 있기때문에 단위 테스트가 쉬워진다.
- 가독성과 재사용성이 높아진다.
- 결과적으로 **모듈 간의 결합도가 낮아지고 유연성이 높아진다.**

#### DI 적용 전
```java
// DI 적용 전 (new 키워드로 직접 객체를 생성함으로써 Test는 ExClassA에게 의존하고있음.)
public class Test {
    private Ex ec;
    
    public Test() {
        // new 키워드를 사용하여 의존하고 있으므로 수정을 하려면 이와같이 직접 소스코드를 수정해야함.
        // ec = new ExClassA();
        ec = new ExClassB();
    }
}
```

#### DI 적용 후
```java
// 생성자로 주입 (Constructor Injection) -> (스프링 권장 방식) 생성자를 통한 주입
public class Test {
    private Ex ec;
    
    public Test(Ex ec) {
        this.ec = ec;
    }
}
```

```java
// Setter로 주입 (Setter Injection) -> setter 메소드로 주입
public class Test {
    private Ex ec;
    
    public Test() {}

    public void setEc(Ex ec) {
        this.ec = ec;
    }
}
```

```java
// 인터페이스로 주입 (Constructor Injection) -> 인터페이스의 구현체로 주입
public interface ExInterface {
    void inject(Ex ec);
}

public class Test implements ExInterface{
    private Ex ec;
    
    public Test() {}

    @Override
    public inject(Ex ec) {
        this.ec = ec;
    }
}
```

이러한 특징들을 가지고 있어서 스프링 컨테이너는 IoC 컨테이너 혹은 DI 컨테이너 라고도 불린다. (최근에는 IoC가 아닌 DI컨테이너라고 불림.)


## BeanFactory와 ApplicationContext
스프링 컨테이너는 이러한 특징들 외에도 빈 관리를 위한 더 많은 기능들을 제공하는데 최상위 인터페이스인 `BeanFactory`와 `BeanFactory`를 상속받아 추가 기능을 제공하는 `ApplicationContext`로 나뉜다.

![image](https://user-images.githubusercontent.com/36228833/185787059-f7ec113f-53b3-4fb9-95ea-d045799e8aaa.png)

### BeanFactory
![image](https://user-images.githubusercontent.com/36228833/185787172-eb918f22-cd0f-4f05-a360-d14f3f05873a.png)
- 스프링 컨테이너의 최상위 인터페이스다.
- 스프링 빈을 관리하고 조회하는 역할을 담당한다. (`getBean()` 메소드를 제공)

### ApplicationContext
![image](https://user-images.githubusercontent.com/36228833/185787436-e75a8f29-7bf9-40c7-bd36-c4c567cb858d.png)
- `BeanFactory` 기능을 모두 상속받아서 제공한다. (`BeanFactory`를 상속받는 `HierarchicalBeanFactory`를 상속받는다.)
- 빈을 관리하고 검색하는 기능을 `BeanFactory`가 제공해주는데, 그러면 둘의 차이가 뭘까?
- 애플리케이션을 개발할 때는 빈을 관리하고 조회하는 기능은 물론이고, 수 많은 부가기능이 필요하다.

**ApplicatonContext가 제공하는 부가기능**<br/>
![image](https://user-images.githubusercontent.com/36228833/185787228-258a3084-2a98-4ba7-be97-54a8a42847e4.png)
- 메시지소스를 활용한 국제화 기능 (MessageSource)
  - 예를 들어서 한국에서 들어오면 한국어로, 영어권에서 들어오면 영어로 출력
- 환경변수 (EnvironmentCapable)
  - 로컬, 개발, 운영등을 구분해서 처리
- 애플리케이션 이벤트 (ApplicationEventPublisher)
  - 이벤트를 발행하고 구독하는 모델을 편리하게 지원
- 편리한 리소스 조회 (ResourceLoader)
  - 파일, 클래스패스, 외부 등에서 리소스를 편리하게 조회

## 빈 등록 방법
- 스프링 컨테이너는 다양한 형식의 설정 정보를 받아들일 수 있게 유연하게 설계되어 있다.
  - 자바 코드, XML, Groovy 등

![image](https://user-images.githubusercontent.com/36228833/185787642-700d8378-1781-407e-8f8b-51b04bf9bf97.png)

- 애노테이션 기반 자바 설정 클래스(.class) 혹은 레거시의 경우 XML기반(.xml)을 통해 스프링 컨테이너에 빈을 등록할 수 있다.
- 스프링이 이런 다양한 설정형식을 지원할 수 있었던 것은 `BeanDefinition` 추상화를 통해 빈의 메타정보를 등록하기 때문이다.

### BeanDefinition
![image](https://user-images.githubusercontent.com/36228833/185788051-bb511af1-2312-4ad8-a6a3-234281ae7e9f.png)
- `AnnotationConfigApplicationContext` 는 `AnnotatedBeanDefinitionReader` 를 사용해서 `AppConfig.class` 를 읽고 `BeanDefinition` 을 생성한다.
- `GenericXmlApplicationContext` 는 `XmlBeanDefinitionReader` 를 사용해서 `appConfig.xml` 설정정보를 읽고 `BeanDefinition` 을 생성한다.
- 새로운 형식의 설정 정보가 추가되면, `XxxBeanDefinitionReader` 를 만들어서 `BeanDefinition` 을 생성하면 된다.
- `BeanDefinition`은 빈과 관련한 설정 정보를 담고있는 객체다.
- 자바 코드로 만들던 XML을 만들던 스프링 내부에서 `BeanDefinition`의 설계대로 정보를 읽기 때문에 가능하다.

하지만 `BeanDefinition`을 실무에서 직접 사용할 일은 거의 없다고 한다..

## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.


[스프링 핵심 원리 - 기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8)<br/>
[IoC, DI란 무엇일까](https://biggwang.github.io/2019/08/31/Spring/IoC,%20DI%EB%9E%80%20%EB%AC%B4%EC%97%87%EC%9D%BC%EA%B9%8C/#undefined)<br/>
[[Spring] IoC, DI 란?](https://jobc.tistory.com/30)<br/>
[[SPRING] 컨테이너 기본 개념](https://salty-computer-until-night.tistory.com/34)
