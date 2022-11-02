---
title: "값 타입 - 기본값 타입 (작성중)"
categories: 
    - jpa
date: 2022-11-02
last_modified_at: 2022-11-02
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "JPA에서의 값 타입을 알아보자."
---

## JPA의 데이터 타입 분류

JPA는 데이터 타입을 최상위 레벨로 봤을 때 엔티티 타입과 값 타입으로 분류한다.

- **엔티티 타입**
  - @Entity로 정의하는 클래스 객체
  - 데이터가 변해도 식별자로 지속해서 추적이 가능하다.
    - 예) 회원 엔티티의 키나 나이 값을 변경해도 식별자로 인식이 가능하다.
    - @Id가 100번인 경우 엔티티 내부에 다른 속성이 바뀌더라도 @Id 100번로 인식 및 추적이 가능하다.

- **값 타입**
  - int, Integer, String처럼 단순히 값으로 사용하는 자바 기본 타입이나 객체를 의미한다.
  - 식별자가 없고 값만 있으므로 변경시 추적이 불가능하다.
    - 예) 정수 100을 200으로 변경하면 완전히 다른값으로 대체된다.
    - 바뀐 200이 과거에 어떤 값이었는지 추적할 수 없다.

## 값 타입

값 타입은 크게 기본값 타입, 임베디드 타입, 컬렉션 값 타입으로 나눌 수 있다.

### **기본값 타입** 
  - 자바 primitive 타입 (int, double 등)
  - wrapper 클래스 (Integer, Long)
  - String
  - 자바가 제공하며 기본적으로 값을 세팅하여 사용할 수 있는 타입들
  - 엔티티에 의존하는 생명주기를 가지고 있다.
    - 예) 회원 엔티티를 삭제하면 이름, 나이, 이메일 등의 필드도 함께 삭제된다.
  - 기본값 타입은 공유하면 안된다.
    - 예) 회원의 이름을 변경 시 다른 회원의 이름도 함께 변경되면 안된다.

### **임베디드 타입(embedded type, 복합 값 타입)**
  - JPA에서 정의를 해서 사용해야함
  - 예) x,y 좌표로 구성된 값이 있음. 이 값을 클래스로 묶어서 값으로 쓰고싶음.
    ```java
    public class Position {
       double x;
       double y;
    }

    ...

    @Entity
    public class Car {
       ...
       
       private Position position;
    }
    ```

### **컬렉션 값 타입(collection value type)**
  - 자바가 제공하는 컬렉션(List, Set 등)에 기본값 타입 혹은 임베디드 타입을 넣어서 사용할 수 있다.
  - 예) 좌표들의 컬렉션 = `List<Position> positions`
  - 예) 

## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
