---
title: "JPA) 내부구조 - 영속성 컨텍스트의 이점"
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
excerpt: "JPA의 영속성 컨텍스트가 주는 장점들에 대해 알아보자"
---

저번 포스팅에 이어 영속성 컨텍스트가 주는 이점들에 대해 알아보자.

## 영속성 컨텍스트의 이점

영속성 컨텍스트는 다음과 같은 이점으로 application과 DB 사이에 중간계층으로 존재한다.

### 1차 캐시
- ![image](https://user-images.githubusercontent.com/36228833/191070952-667d8625-8a28-438e-ae2e-ecf67b75437e.png)
- 영속성 컨텍스트 내부에는 1차 캐시가 존재한다. (1차 캐시 자체를 영속성 컨텍스트 라고 이해해도 된다.)
- `Entity`가 영속성 컨텍스트에 의해 영속 상태가 되면 1차 캐시에 저장된다.
- 1차 캐시 내부에는 `Entity` 의 `@Id`로 등록된 필드가 식별자가 되고 `Entity` 객체가 해당 식별자의 값이 된다. (Key-Value)
- 1차 캐시에 저장된 `Entity`를 조회할 경우 `DB Connection pool`을 거치지 않고 **캐시에 있는 값을 조회**하기 때문에 성능상의 이점이 있다.
  - 만약에 1차 캐시에 `Entity`가 없다면?
    - ![image](https://user-images.githubusercontent.com/36228833/191071009-6f68a82e-21e6-49fa-9451-685f3e3c2f66.png)
    - `Entity` 객체를 DB에서 조회한 후 1차 캐시에 저장하여 1차 캐시에서 조회 한 후 반환한다.

- 영속성 컨텍스트는 트랜잭션이 끝날 때 같이 종료됨.
  - 즉, 트랜잭션이 종료될 때 1차 캐시도 같이 소멸되기 때문에 **찰나의 순간에서만 이득이 있다.** 그렇게 큰 이득은 아니다.
  - 그러나 비즈니스 로직이 매우 복잡한 경우에는 효과가 있다.

### 영속 Entity의 동일성(identity) 보장
- 영속 상태의 `Entity`에 대해서 동일성을 보장한다. (== 비교시)
  - 예를 들어, 다음과 같이 `member1`이라는 영속 상태의 `Entity`를 여러번 호출하여 비교해도 결국 1차 캐시에 있는 동일한 `Entity`를 호출한 것임으로 true를 반환한다.
  ```java
  Member a = entityManager.find(Member.class, "member1");
  Member b = entityManager.find(Member.class, "member1");
  System.out.println(a == b); // true
  ```
  - a와 b 객체 둘다 같은 reference다.
- 1차 캐시로 반복 가능한 읽기(REPEATABLE READ) 등급의 트랜잭션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공한다.


### 트랜잭션을 지원하는 쓰기 지연 (Transactional write-behind)
- 영속성 컨텍스트가 영속화 할때마다 일일이 SQL을 데이터베이스에 보내지 않고 트랜잭션 커밋 시점에 한번에 보낸다.
  ```java
  EntityManager em = emf.createEntityManager();
  EntityTransaction transaction = em.getTransaction();

  //엔티티 매니저는 데이터 변경시 트랜잭션을 시작해야 한다.
  transaction.begin(); // [트랜잭션] 시작

  em.persist(memberA);
  em.persist(memberB);
  //여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.

  //커밋하는 순간 데이터베이스에 INSERT SQL을 보낸다.
  transaction.commit(); // [트랜잭션] 커밋
  ```
  - `EntityManager`가 SQL을 생성하여 `쓰기 지연 SQL 저장소`에 저장했다가 트랜잭션 커밋 시점에 한번에 보내게 된다. (`flush`가 이루어짐)
    - ![image](https://user-images.githubusercontent.com/36228833/191071261-82be7ea0-9901-4e20-9877-f86539c77df8.png)
    - 일종의 버퍼링 기능이며 `hibernate.jdbc.batch_size` 옵션을 통해 설정할 수 있다. (한 번에 몇 건까지 모았다가 보낼건지)
      - 여러 번의 `network`를 통하지 않고 **한 번의 `network`를 통해 전부 보낼 수 있기 때문에** 성능을 개선할 수 있다.


### 변경 감지(Dirty checking)
- 영속 상태의 `Entity`를 마치 객체의 속성을 수정하듯 `setter`를 통해 수정한 뒤 커밋을 하면 알아서 DB에 반영된다.

  ```java
  EntityManager em = emf.createEntityManager();
  EntityTransaction transaction = em.getTransaction();

  transaction.begin(); // [트랜잭션] 시작

  // 영속 엔티티 조회
  Member memberA = em.find(Member.class, "memberA");

  // 영속 엔티티 데이터 수정
  memberA.setUsername("hi");
  memberA.setAge(10);

  //em.update(member) 이런 코드가 있어야 하지 않을까? setter만으로 수정이 가능하다?

  transaction.commit(); // [트랜잭션] 커밋
  ```

- ![image](https://user-images.githubusercontent.com/36228833/191071384-9df17b61-0f3a-4999-b4d4-efc5026bd65f.png)
- 영속성 컨텍스트의 스냅샷 활용
  - 영속성 컨텍스트는 1차 캐시에 `Entity`가 들어오는 순간 해당 시점의 스냅샷을 저장해놓는다.
  - 트랜잭션 커밋 이후 `flush()`가 일어날 때 **1차 캐시에 저장된 스냅샷과 `Entity`의 상태를 비교**한다.
  - 만일 저장된 스냅샷과 `Entity`의 상태가 다르다면 `UPDATE SQL`을 생성한다 (변경 감지 발생!)
  - 그 후 `flush` -> `commit`을 통해 DB에 반영된다.


## 마무리
다음 포스팅에선 영속성 컨텍스트의 변경내용을 DB에 반영하는 `flush()`에 대해 알아보자

## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
