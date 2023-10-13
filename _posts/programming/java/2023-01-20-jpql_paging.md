---
title: "JPA) 객체지향 쿼리 언어 - 페이징(Paging)"
categories: 
    - java
date: 2023-01-20
last_modified_at: 2023-01-20
toc: true
toc_sticky: true
excerpt: "객체지향 쿼리 언어의 페이징"
---

객체지향 쿼리 언어에서 페이징의 사용 방법을 알아보자.

JPA는 페이징을 다음 2개의 API로 추상화했다.

- `setFirstResult(int startPosition)`: 조회시작 위치(0부터 시작)
- `setMaxResults(int MaxResults)`: 조회할 데이터 수

```java

List<Member> resultList = entityManager.createQuery("select m from Member m order by m.age desc", Member.class)
              .setFirstResult(0) // 0번째 row부터 
              .setMaxResults(10) // 10개의 row를 가져온다.
              .getResultList();

```

- 한가지 주의할 점은 사용자가 설정한 SQL 방언에 따라 수행되는 쿼리가 달라진다.
  - Oracle -> rownum 활용한 서브쿼리 조회
  - H2 -> limit, offset
  - MySQL -> Limit ?, ?

## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
