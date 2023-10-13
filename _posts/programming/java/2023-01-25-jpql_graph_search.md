---
title: "JPA) 객체지향 쿼리 언어 - 경로표현식"
categories: 
    - java
date: 2023-01-25
last_modified_at: 2023-01-25
toc: true
toc_sticky: true
excerpt: "객체지향 쿼리 언어의 경로표현식"
---

## 경로표현식이란
경로표현식이란 .을 찍어서 객체 그래프를 탐색하는 것을 의미한다.

이를테면 다음과 같다.

```
select m.username  // 상태 필드:
from member m
  join m.team t    // 단일 값 연관 필드: Member의 연관 엔티티 필드
  join m.orders o  // 컬렉션 값 연관 필드: Member의 컬렉션 필드 
where t.name = '팀A'
```

- 상태 필드(state field)
  - 단순히 값을 저장하기 위한 필드
  - 경로 탐색의 끝을 의미하며 더이상 탐색할 수 없다.
  ```java
  // username 속성(상태필드)을 호출한다. 그 이상의 경로는 존재하지 않기때문에 탐색할 수 없다.
  String query = "select m.username from Member m"; 
  ```

- 연관 필드(association field)
  - 연관관계를 위한 필드이다.
  - 단일 값 연관 필드
    - @ManyToOne, @OneToOne, 대상이 엔티티
    - 묵시적 내부 조인(inner join)이 발생하며 탐색할 수 있다.
      ```java
      // 다음의 예에선 Member와 Team은 N:1 관계를 가진다 (Member의 입장에서 @ManyToOne)
      // team 단일 연관필드 속성을 호출한다. 
      // team의 상태 필드인 name 까지 추가로 탐색할 수 있다.
      String query = "select m.team.name from Member m"; 

      List<Team> result = em.createQuery(query, Team.class).getResultList();

      // 위의 쿼리를 실행할 경우 묵시적으로 다음과 같은 inner join 쿼리가 수행된다.
      /*
        select team1.id, team1.name
        from Member member1 inner join Team team1 on member1.TEAM_ID = team1.id
      */
      ```

  - 컬렉션 값 연관 필드
    - @OneToMany, @ManyToMany, 대상이 컬렉션
    - 묵시적 내부 조인(inner join)이 발생하며 더이상 탐색할 수 없다.
      ```java
      // 다음의 예에선 Member와 Team은 N:1 관계를 가진다 (Team의 입장에서 @OneToMany)
      // team의 컬렉션 연관필드 속성을 호출한다. 
      // 값이 필드가 아닌 컬렉션이기 때문에 추가적인 탐색은 불가능하다. (size는 가능)
      String query = "select t.members from Team t"; 

      // 그러나 다음과 같이 명시적 join 후 별칭을 통한 상태필드 탐색 가능
      String query = "select m.username from Team t join t.members m"; 

      Collection result = em.createQuery(query, Collection.class).getResultList();

      // 위의 쿼리를 실행할 경우 묵시적으로 다음과 같은 inner join 쿼리가 수행된다.
      /*
        select members.id, members.username, members.age
        from Team team inner join Member members on team.id = members.TEAM_ID
      */
      ```

- 묵시적으로 내부 조인이 발생하게끔 개발하는 것은 **지양**해야 한다.
  - 쿼리를 예측할수 없다.
  - 쿼리를 예측할 수 없으니(어디서 쿼리가 수행된지 찾기 힘들다) 성능 튜닝에 있어서 안좋은 영향을 미친다.
  - **묵시적 내부 조인이 발생하지 않도록 명시적으로 join을 작성하여 구현하자.**

## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
