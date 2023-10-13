---
title: "JPA) 내부구조 - 영속성 컨텍스트"
categories: 
    - java
date: 2022-09-19
last_modified_at: 2022-09-19
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "JPA 내부에서 영속성 관리를 위해 어떻게 동작하는지 알아보자"
---

## JPA 에서 가장 중요하게 생각해야 할 것 2가지

- 객체와 관계형 데이터베이스 매핑하기 (Object Relational Mapping)
  - DB와 객체를 어떻게 설계해서 둘을 매핑할 것인가?
  - 설계에 대한 고민

- 영속성 컨텍스트 (Persistence Context)
  - 실제 JPA 내부 동작의 핵심 개념

## EntityManager와 EntityManagerFactory
- `EntityManager`
  - 내부에 영속성 컨텍스트라는 공간으로 `Entity`를 관리하는 역할을 수행하기 때문에 영속성 컨텍스트라고도 불린다. (하단에 자세히 설명)
  - `transaction`을 수행할 때마다 생성된다.
    - `transaction`: 하나의 작업 단위
  - thread 간에 공유하면 안된다.
  - 영속성 컨텍스트 안에 `쓰기 지연 SQL 저장소`라는 공간이 존재한다.
  - 생성된 `EntityManager`는 `transaction` 수행 후 `close()`를 해서 내부적으로 `DB Connection pool` 객체를 반환하게끔 해야한다.

- `EntityManagerFactory`
  - `EntityManager`를 관리한다.
  - DB당 하나의 `EntityManagerFactory`만 생성되어야 한다.
  - 여러 thread가 `EntityManager`에 접근할 때 발생하는 동시성(Concurrency) 문제를 해결해준다.

- `EntityManagerFactory`와 `EntityManager` 설명을 위해 다음과 같이 웹 어플케이션을 구성한다고 가정하자.
  - ![image](https://user-images.githubusercontent.com/36228833/191070059-8a80d21d-1c88-4302-b923-5eefb5d623d5.png)
  - `EntityManagerFactory`를 통해 고객의 요청이 올때마다 `EntityManager`를 생성함 (transaction 발생)
  - `EntityManager`는 내부적으로 `DB Connection pool`을 통해 DB에 접근한다.

## 영속성 컨텍스트 (Persistence Context)
- JPA를 이해하는데 가장 중요한 용어

- `Entity`를 영구 저장하는 환경이라는 뜻
  - DB에 직접 저장하는 개념이 아니라 영속성 컨텍스트라는 곳에 저장한다(영속화한다)는 의미
  - DB에 저장하려면 트랜잭션 커밋을 해야함.

- 논리적인 개념이며 눈에 보이지 않는다.

- `EntityManager`를 통해 접근 가능하다.
  - <img src="https://user-images.githubusercontent.com/36228833/191070174-c273b8cb-166c-46a4-92b2-a373a82efc4e.png" width="50%" height="50%">
  - `EntityManager`와 영속성 컨텍스트와의 관계는 환경에 따라 달라질 수 있다.
  - 기본적으로 `EntityManager`를 생성하면 영속성 컨텍스트라는 공간이 생긴다 라고 이해하자.

## Entity의 생명 주기
### 전체 생명주기 흐름
<center><img src="https://user-images.githubusercontent.com/36228833/191070256-a5bf4529-da7f-4297-bbeb-72f560c942d1.png"></center>
  
### 비영속 (new/transient)
<center><img src="https://user-images.githubusercontent.com/36228833/191070683-536f1639-710d-493a-abf8-2e590ff440bf.png"></center>
<br/>
```java
//객체를 생성한 상태(비영속) 
Member member = new Member(); 
member.setId("member1"); 
member.setUsername("회원1");
```
- 영속성 컨텍스트와 전혀 관계가 없는 **새로운** 상태
- 말 그대로 **객체를 생성만** 한 상태


### 영속 (managed)
<center><img src="https://user-images.githubusercontent.com/36228833/191070741-4741ae9c-6667-4fb8-8029-a264b1709253.png"></center>
<br/>
```java
//객체를 생성한 상태(비영속) 
Member member = new Member(); 
member.setId("member1"); 
member.setUsername(“회원1”);
EntityManager em = emf.createEntityManager();
em.getTransaction().begin();
//객체를 저장한 상태(영속)
em.persist(member);
```
- 영속성 컨텍스트에 **관리**되는 상태
- `Entity`를 영속성 컨텍스트에 저장하면 그 순간 부터 **영속성 컨텍스트에 의해 *관리**되는 상태가 됨.
- DB에 저장하려면 트랜잭션 커밋을 해야한다.
  - 트랜잭션 커밋 시 영속성 컨텍스트에 저장된 정보들을 DB 쿼리로 날린다.

### 준영속 (detached)
  - 영속성 컨텍스트에 저장되었다가 **분리**된 상태
  - 비영속과 준영속의 차이는 **한번이라도 영속성 컨텍스트에 저장이 되었느냐**의 차이가 있다.
    - 준영속의 경우 한번은 저장이 되었기 때문에 1차 캐시에 식별자가 남아있다.
  ```java
  em.detach(entity);
  ```

### 삭제 (removed)
  - DB에 **삭제**를 요청한 상태 (영속성 컨텍스트에서 영구 삭제)
  ```java
  em.remove(entity);
  ```

## 마무리

다음장에서는 영속성 컨텍스트가 주는 이점에 대해서 정리하겠다.

## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
