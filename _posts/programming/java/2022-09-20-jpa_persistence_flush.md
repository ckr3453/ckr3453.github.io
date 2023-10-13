---
title: "JPA) 내부구조 - 플러시(Flush)"
categories: 
    - java
date: 2022-09-20
last_modified_at: 2022-09-20
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "JPA의 flush에 대해 알아보자."
---

## 플러시(Flush)란?

- 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영하는 것을 뜻한다.
  - 영속성 컨텍스트의 내용을 비우는게 아니다!

- `transaction commit`이 일어날 때 자동적으로 `flush`가 선행으로 수행 된다.
  - 쓰기 지연 SQL 저장소에 있던 `INSERT`, `UPDATE`, `DELETE` SQL이 데이터베이스에 날라가는 것이다.
  - `commit`과 `flush`는 다른 개념이다.
    - `flush`를 통해 DB에 반영이 되었다 하더라도 현재 트랜잭션에서만 유효한 것 이므로 최종적으로 `commit`을 하지 않는 이상 실제 DB에 동기화가 되지 않는다.
    - `commit`은 트랜잭션의 내용을 실제 DB에 동기화 하는 작업이라는 점에서 차이가 있다는 것을 명심하자. 

- 영속성 컨텍스트와 데이터베이스의 상태를 맞추는 작업이다.

## flush 발생 순서

1. 변경 감지(Dirty Checking)를 통해 상태가 변경된 `Entity`에 대한 SQL문이 생성된다.
2. 해당 SQL문이 쓰기 지연 SQL 저장소에 저장된다.
3. 쓰기 지연 SQL 저장소에 저장된 SQL문을 데이터베이스에 전송한다.

## 영속성 컨텍스트를 flush 하는 방법

- 직접 호출
  - `em.flush()`
    ```java
    Member member = new Member(200L, "member");
    em.persist(member);

    em.flush(); // flush 직접 호출
    ```
- 자동 호출
  - `transaction commit`
    ```java
    tx.begin();
    Member member = new Member(200L, "member");
    em.persist(member);

    tx.commit();  // flush 자동 호출
    ```
  - `JPQL`
    ```java
    em.persist(memberA);
    em.persist(memberB);
    em.persist(memberC);

    //중간에 JPQL 실행
    query = em.createQuery("select m from Member m", Member.class);
    List<Member> members= query.getResultList();
    ```
    - `JPQL` 실행 시 `flush()`가 자동으로 호출되는 이유
      - 영속성 컨텍스트에 저장 후 `flush`가 되지 않은 상태에서 `JPQL`를 통해 조회하면 당연히 조회되지 않는다.
      - 때문에 조회 결과가 다르게 나오는 상황이 발생한다. (문제가 될 수 있음)
      - 이러한 상황을 방지하기 위해 `JPQL` 조회시 default로 `flush()`가 호출 되도록 되어있다. (`FlushModeType.AUTO`)

## flush 모드 옵션

- `em.setFlushMode(FlushModeType.AUTO)`
  - `commit`이나 쿼리를 실행할 때 플러시를 수행한다. (default)
  - 가급적 기본값을 유지하는걸 권장한다.

- `em.setFlushMode(FlushModeType.COMMIT)`
  - 커밋할 때만 플러시를 수행한다.


## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
