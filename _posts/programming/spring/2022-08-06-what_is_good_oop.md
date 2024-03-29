---
title: "스프링, 객체지향 프로그래밍의 핵심 - 다형성"
categories: 
    - spring
date: 2022-08-06
last_modified_at: 2022-08-08
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "스프링과 객체지향 프로그래밍의 관계에 대해 알아보자"
---

## 💻 객체 지향 프로그래밍이란
객체 지향 프로그래밍은 컴퓨터 프로그램을 명령어의 목록으로 보는 시각에서 벗어나 여러개의 독립된 단위, 즉 **"객체"**들의 모임으로 파악하고자 하는것이다. 

각각의 객체는 메세지를 주고받고, 데이터를 처리할 수 있다.

객체지향 프로그래밍은 **다형성**이라 불리는 특징으로 인하여 프로그램을 유연하고 변경이 용이하게 만들기 때문에 대규모 소프트웨어 개발에 많이 사용된다.
- 유연하고 변경 용이하다. 예를 들면,
  - 레고 블럭을 조립하다.
  - 컴퓨터의 키보드, 마우스를 교체한다.
  - 컴퓨터 부품을 교체한다.

즉, 컴포넌트를 쉽고 유연하게 변경하면서 개발할 수 있는 방법을 뜻한다.

다형성에 대하여 좀 더 자세히 알아보자.

## 🎯 다형성 (Polymorphism)
> 프로그램 언어의 다형성(多形性, polymorphism; 폴리모피즘)은 그 프로그래밍 언어의 자료형 체계의 성질을 나타내는 것으로, 프로그램 언어의 각 요소들(상수, 변수, 식, 오브젝트, 함수, 메소드 등)이 다양한 자료형(type)에 속하는 것이 허가되는 성질을 가리킨다. 반댓말은 단형성(monomorphism)으로, 프로그램 언어의 각 요소가 한가지 형태만 가지는 성질을 가리킨다. - [위키백과](https://ko.wikipedia.org/wiki/%EB%8B%A4%ED%98%95%EC%84%B1_(%EC%BB%B4%ED%93%A8%ED%84%B0_%EA%B3%BC%ED%95%99))

즉, 다형성은 하나의 객체가 여러 가지의 타입을 가질 수 있다는 것을 뜻한다.

이게 객체지향 프로그래밍에 있어서 왜 중요한 개념일까? 한가지 예를 들어보자.

## 💡 역할과 구현의 분리 

### 운전자와 자동차의 관계
![image](https://user-images.githubusercontent.com/36228833/183387241-9a24c5d1-3edf-401c-b28e-d123de1e36e3.png)

운전자와 자동차의 관계로 예를 들어보자.

기아, 현대, 테슬라 등의 회사에서 새로운 자동차 모델들이 출시 되어 운전자가 해당 차량들의 내부 구조를 몰라도 바로 운전을 할 수 있다. 

왜? 각 회사 차량들의 차체와 성능은 전부 다를 수 있어도 **자동차** 라는 **역할**은 변하지 않기 때문이다.

자동차는 핸들이 있으며 악셀, 브레이크등 페달이 있고 기어를 조작할 수 있는 **공통적인 특징(설계)**가 있기 때문이다.

이러한 경우 자동차의 역할을 유지한 채 무한히 자동차 세계를 확장할 수 있다. (SUV, 세단, 전기자동차, 하이브리드 등등)

다르게 표현하면 **운전자에게 영향을 주지 않으면서 새로운 자동차 및 기능을 제공할 수 있다는 의미 (확장)**가 된다.


### 로미오와 줄리엣 공연
![image](https://user-images.githubusercontent.com/36228833/183387413-19af387b-4e31-44b5-a779-7dc49a9517db.png)

다른 예로 로미오와 줄리엣 공연을 한다고 가정해보자.

각 역할에 주어진 대본이 존재하므로 로미오든 줄리엣이든 해당 **역할**에 어떤 배우가 담당하든(구현) 역할을 수행할 수 있다는 뜻이다.

로미오와 줄리엣의 서로 담당하는 배우가 바뀐다 하더라도 본 공연의 내용에는 전혀 지장이 없다.

이처럼 자유롭게 대체가 가능하다는 의미로 **유연하고 변경이 용이하다** 라는것을 알 수 있다.


### 정리

이처럼 역할과 구현으로 세상을 구분하면 단순해지고, 유연해지며 변경도 편리해진다.

다형성의 특징을 실세계에 비유한 위의 2가지 예시를 토대로 다음과 같은 내용들을 알 수 있다.

- **클라이언트**는 대상의 **역할(인터페이스)만 알면** 된다.
- **클라이언트**는 구현 대상의 **내부 구조를 몰라도** 된다.
- **클라이언트**는 구현 대상의 **내부 구조가 변경**되어도 영향을 받지 않는다.
- **클라이언트**는 구현 **대상 자체를 변경**해도 영향을 받지 않는다.
- 즉, **클라이언트** 를 변경하지 않고, 서버의 구현 기능을 유연하게 변경할 수 있다.

이러한 특성을 이용하여 객체 설계시 역할(인터페이스)을 먼저 부여하고, 그 역할을 수행하는 구현 객체(혹은 클래스)를 만든다.
- 그렇기 때문에 구현보다 역할이 더 중요하다!!!


### 한계

하지만 역할-구현 관계의 경우 **역할** 에 너무 의존적, 종속적이기 때문에 다음과 같은 한계점이 존재한다.
- 역할(인터페이스) 자체가 변하면 클라이언트, 서버 모두에 큰 변경이 발생한다.
- 자동차를 비행기로 변경해야 한다면?
- 로미오와 줄리엣 공연에서 대본 자체가 변경된다면?
- USB 인터페이스가 변경된다면?

이러한 관계를 이해하고 역할을 안정적으로 잘 설계하는 것이 중요하다. (최대한 변화가 필요하지 않게끔)

## 🔑 스프링과 객체 지향의 관계
- 다형성이 가장 중요하다.
- 스프링은 **다형성을 극대화**해서 이용할 수 있게 도와준다.
- 스프링에서 이야기하는 제어의 역전(IoC), 의존관계 주입(DI)은 다형성을 활용해서 역할과 구현을 편리하게 다룰 수 있도록 **지원**한다.
- 스프링을 사용하면 마치 레고 블럭 조립하듯이, 공연 무대의 배우를 선택하듯이 구현을 편리하게 변경할 수 있다.


## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리한 내용입니다.

> [스프링 핵심 원리 - 기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8)
