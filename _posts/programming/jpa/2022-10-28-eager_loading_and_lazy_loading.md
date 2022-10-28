---
title: "연관관계 관리 - 즉시 로딩과 지연 로딩 (작성중)"
categories: 
    - jpa
date: 2022-10-28
last_modified_at: 2022-10-28
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "연관관계에서 즉시 로딩과 지연 로딩이 하는 역할에 대해 알아보자."
---

## 지연로딩 (Lazy Loading)

<center><img src="https://user-images.githubusercontent.com/36228833/198076062-13069e79-a6d5-4114-abb6-13c43f246e1f.png"></center><br/>

저번에 이어 그림처럼 Member와 Team의 관계가 N:1 이라고 했을 때 Member의 경우 **자신의 속성만 조회하고 싶을 때는 Team의 조회는 필요없다.**

그러나 연관관계로 인하여 Member entity 는 원하지 않아도 Team 객체를 조회하기 위해 join 쿼리를 수행한다. 이때 지연로딩을 통한 최적화를 적용할 수 있다.

JPA는 다음과 같이 연관관계 어노테이션 속성으로 프록시 조회를 통한 지연로딩을 할 수 있도록 지원한다.

<center><img width="870" alt="image" src="https://user-images.githubusercontent.com/36228833/198597554-2f8afdf0-82e5-4ca3-bb02-9aa20452d8f0.png"></center>


```java
@Entity
public class Member {
    @Id
    @GeneratedValue
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @ManyToOne(fetch = FetchType.LAZY) // 지연로딩
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    ..
}

....

Member member = entityManager.find(Member.class, 1L); 
System.out.println(member.getName());           // 지연 로딩으로 인해 team 을 조회하지 않는다. (team은 프록시 객체로 적용된다)
System.out.println(member.getTeam().getName()); // 이때 프록시 객체에 초기화가 일어나며 join 쿼리를 수행한다.
```

## 즉시로딩 (Eager Loading)

그러나 지연로딩의 상황과 반대로 Member와 Team을 자주 함께 사용하게 되는 상황이 오면 호출 할때마다 따로따로 조회하는 것보다 한번에 전부 조회하는게 성능상 유리하다.

JPA는 지연로딩에 이어 연관관계 어노테이션 속성으로 즉시로딩을 할 수 있도록 지원한다.

<center><img width="864" alt="image" src="https://user-images.githubusercontent.com/36228833/198598094-bf4cefde-c2c9-47fe-810a-2e3059ec6c76.png"></center>

```java
@Entity
public class Member {
    @Id
    @GeneratedValue
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @ManyToOne(fetch = FetchType.EAGER) // 즉시로딩
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    ..
}

....

Member member = entityManager.find(Member.class, 1L); // 즉시 로딩으로 join을 통해 Member와 Team을 한번에 가져온다.
System.out.println(member.getName());           
System.out.println(member.getTeam().getName());
```

JPA 구현체는 가능하면 조인을 사용하여 SQL을 한번에 조회하려고 한다. (default)

## 프록시와 즉시로딩 사용 시 주의사항
- 가급적 지연 로딩만 사용하자.(특히 실무에서)
  - 해외 여러 레퍼런스들을 찾아보면 가급적 지연로딩을 사용하라고 권고한다. 왜일까?
  - 즉시로딩 사용시 아래와 같은 일이 벌어진다.

- 즉시 로딩을 적용하면 예상하지 못한 SQL이 발생한다.

- 즉시 로딩은 JPQL에서 N+1 문제를 일으킨다.

- `@ManyToOne`, `@OneToOne` 은 기본이 즉시 로딩이다.
  - FetchType.LAZY 를 통해 지연 로딩으로 수정해야한다!
  - `@OneToMany`, `@ManyToMany`는 기본이 지연로딩이기 때문에 괜찮다.


  
## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
