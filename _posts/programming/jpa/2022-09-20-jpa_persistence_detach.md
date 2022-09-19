---
title: "JPA 내부구조 - 준영속 상태"
categories: 
    - jpa
date: 2022-09-20
last_modified_at: 2022-09-20
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "JPA의 준영속 상태에 대해 알아보자."
---

## 준영속 상태란?

- 영속상태가 되는 조건
  - `em.persist(entity)`
    - `Entity`를 영속성 컨텍스트에 직접 등록
  - `em.find(Member.class, 1L)`
    - 영속성 컨텍스트에서 `Entity`를 찾을 때 1차 캐시에 존재하지 않으면 DB에서 찾아와 1차 캐시에 등록하게 된다.

- 영속 상태의 `Entity`가 영속성 컨텍스트에서 분리(detach) 되는것을 뜻한다.

- 준영속 상태가 되면 영속성 컨텍스트가 제공하는 기능을 사용하지 못한다.
  - 동일성 보장, 변경 감지(Dirty Checking) 등

## 준영속 상태로 만드는 방법

- `em.detach(entity)`
  ```java
  tx.begin();
  Member member = em.find(Member.class, 10L);
  member.setName("AAAA");

  em.detach(member); // 준영속 상태

  tx.commit();  // 변경 감지가 일어나지 않음
  ```
  - 특정 `Entity`만 준영속 상태로 전환
  - 변경 감지에 의해 `UPDATE` SQL이 전송되어야 하지만 준영속 상태가 되었기 때문에 아무일도 일어나지 않는다.

- `em.clear()`
  ```java
  tx.begin();
  Member member = em.find(Member.class, 10L);
  member.setName("AAAA");

  em.clear(); // 영속성 컨텍스트 초기화

  tx.commit();  // 변경 감지가 일어나지 않음
  ```
  - 영속성 컨텍스트(1차 캐시)를 완전히 초기화 
  - 영속성 컨텍스트 내 모든 `Entity`를 지우므로 사실상 준영속 상태가 되버림.

- `em.close()`
  ```java
  tx.begin();
  Member member = em.find(Member.class, 10L);
  member.setName("AAAA");

  em.close(); // 강제로 영속성 컨텍스트를 종료

  tx.commit();  // 변경 감지가 일어나지 않음
  ```
  - 영속성 컨텍스트를 종료
  - 영속성 컨텍스트가 종료되며 1차 캐시가 소멸하므로 영속성 컨텍스트 내 모든 `Entity`는 준영속 상태가 됨.


## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
