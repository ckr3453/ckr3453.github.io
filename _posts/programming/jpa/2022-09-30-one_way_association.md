---
title: "단방향 연관관계"
categories: 
    - jpa
date: 2022-09-28
last_modified_at: 2022-09-29
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "객체와 테이블의 단방향 연관관계를 이해하자"
---

JPA에서의 단방향 연관관계를 예제를 통해 이해하자.

## 시나리오
- 회원과 팀이 있다.
- 회원은 하나의 팀에만 소속될 수 있다.
- 회원과 팀은 다대일(N:1) 관계다
  - 하나의 팀에 여러명의 회원이 소속될 수 있다.

## 객체를 테이블에 맞춰서 모델링 했을 때 (객체간 연관관계 X)
<center><img src="https://user-images.githubusercontent.com/36228833/193238545-c124e888-9fbb-4883-8b57-229039ff7851.png"></center>

- 테이블 연관관계 적용
  - `Entity` 모델링 시 `Team` 객체 참조 대신 FK인 `teamId`를 구현 한다.  

```java
@Entity
public class Member { 
  @Id @GeneratedValue
  private Long id;

  @Column(name = "USERNAME")
  private String name;

  @Column(name = "TEAM_ID")
  private Long teamId;    // FK

  //getter and setter
}
```
```java
@Entity
public class Team {
  @Id @GeneratedValue
  private Long id;

  private String name;

  //getter and setter 
}
```

- **`Entity`를 테이블에 맞춰서 모델링 했을 때 문제점 발생**
  - 정보를 저장할 때
    ```java
    //팀 저장
    Team team = new Team();
    team.setName("TeamA");
    em.persist(team);

    //회원 저장
    Member member = new Member();
    member.setName("member1");
    member.setTeamId(team.getId()); // Team Entity의 FK를 직접 핸들링
    em.persist(member);
    ```
    - 위 예제와 같이 회원을 저장할 때 팀의 FK 식별자를 `직접` 저장해야 한다.
    - 객체지향 관점에서 봤을 때 `member.setTeamId(team.getId())`가 아닌 `member.setTeam(team)`가 되어야 하지 않을까?
  - 정보를 조회할 때
    ```java
    // 회원 조회
    Member findMember = em.find(Member.class, member.getId());

    // 팀 조회 (회원과 연관관계 없음)
    Team findTeam = em.find(Team.class, team.getId());
    ```
    - 회원이 소속된 팀을 조회하려면 팀의 id를 `직접` 구해야 한다.
    - 객체지향 관점에서 봤을 때 `em.find(Team.class, team.getId())`가 아닌 `findMember.getTeam()`이 되어야 하지 않을까?
  - 결국 **객체간의 협력 관계**를 만들 수 없다.
    - **테이블은 외래키로 조인**을 해서 연관된 테이블을 찾는다.
    - 하지만 **객체는 참조**를 사용해서 연관된 객체를 찾는다.
    - 테이블과 객체 사이에는 이런 큰 간격이 존재한다.

## 객체 지향을 적용한 모델링 (객체간 연관관계 O)
<center><img src="https://user-images.githubusercontent.com/36228833/193238678-a29802d1-75a2-4559-a8db-22a67ea42e0d.png"></center>

- 객체 연관관계 적용
  - 테이블 연관관계와는 달리 FK를 구현하지 않고 `Team` 객체를 참조하여 구현한다.  

```java
@Entity
public class Member { 
  @Id @GeneratedValue
  private Long id;

  @Column(name = "USERNAME")
  private String name;

  @ManyToOne // 객체 간의 N:1 연관관계를 위해 적용 (Member가 N, Team이 1)
  @JoinColumn(name="TEAM_ID") // 실제 쿼리를 날릴 때 join 대상 컬럼을 지정
  private Team team; 

  //getter and setter
}
```

- **객체 지향 모델링** 적용 후
<center><img src="https://user-images.githubusercontent.com/36228833/193238746-dba1e309-bdda-4d7c-87b4-abf322618ccf.png"></center>

  - 정보를 저장할 때
    ```java
    //팀 저장
    Team team = new Team();
    team.setName("TeamA");
    em.persist(team);

    //회원 저장
    Member member = new Member();
    member.setName("member1");
    member.setTeam(team); // 단방향 연관관계 설정, 참조한 객체를 저장
    em.persist(member);
    ```
    - `FK`가 아닌 `Team` 객체를 그대로 저장한다. (객체 참조로 연관관계 저장)
    - 또한, 저장된 기존 팀에서 새로운 팀으로 수정할 때도 연관관계가 적용되어 DB에서 `FK`에 대한 `UPDATE` 쿼리가 적용된다. 

  - 정보를 조회할 때
    ```java
    //조회
    Member findMember = em.find(Member.class, member.getId());

    //참조를 사용해서 연관관계 조회
    Team findTeam = findMember.getTeam();
    ```
    - `findMember.getTeam()`을 통해 해당 회원이 소속된 팀을 바로 조회할 수 있다. (객체 참조로 연관관계 조회)

## 정리
- 객체를 테이블에 맞춰 모델링 했을 때 발생 가능한 문제들을 파악했다.
  - 그러므로 객체 연관관계를 적용하여 모델링하자.
- 객체 연관관계를 적용하여 객체지향적으로 설계가 가능하며, 객체 그래프 탐색도 중간에 끊기지 않는다.

## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
