---
title: "JPA) 연관관계 관리 - 즉시 로딩과 지연 로딩"
categories: 
    - java
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
  - 연관관계가 얽혀있는 엔티티 조회시 join 쿼리를 수행함으로써 **얽혀 있는 정도에따라 join 하는 테이블이 상당히 많아질 수 있다.**
    - 1~2개 테이블이면 괜찮으나 한번 조회할 때마다 여러 테이블을 join 해서 가져오게되면 성능 이슈가 발생하게 된다.

- 즉시 로딩은 JPQL에서 N+1 문제를 일으킨다.
  - 아래와 같이 JPQL을 사용해서 결과를 가져온다고 가정해보자.
    - `em.createQuery("select m from Member m", Member.class).getResultList();`
    - `em.find`로 객체 조회를 하는 경우 JPA가 내부적으로 최적화를 진행한다.
    - 그러나 **JPQL로 쿼리를 직접 수행한 경우**는 다음과 같은 문제가 발생한다.
      - `select m from Member m`을 `select * from Member` 로 변경하여 수행한다.
      - 만약 Member 에 연관 관계가 즉시로딩으로 설정되어 있을 경우 해당 연관관계를 한번에 모두 가져와야 한다.
        - Team이 FetchType.EAGER로 설정되어 있으므로 **관련된 Team을 조회하는 쿼리를 추가적으로 수행**한다. 
          - `select * from Team where TEAM_ID = '..'`
  - 결국 `select m from Member m` 이라는 쿼리 1개를 수행 하였는데 n개의 의도하지않은 쿼리가 추가적으로 수행되는 N+1 문제가 발생한다.
  - 이를 해결하기 위해 몇가지 방법이 존재한다.
    - 즉시로딩에서 지연로딩으로 수정
    - JQPL 사용시 FetchJoin을 사용하여 한번에 원하는 연관관계 엔티티를 가져오는 방법
    - @EntityGraph 사용하기 (관계가 조금만 복잡해져도 권장하지 않음)
    - BatchSize를 사용하여 여러개의 쿼리를 나가게 하는 대신 하나의 in 쿼리로만 나가게끔 최적화하는 방법

- `@ManyToOne`, `@OneToOne` 은 기본이 즉시 로딩이다.
  - FetchType.LAZY 를 통해 지연 로딩으로 수정해야한다!
  - `@OneToMany`, `@ManyToMany`는 기본이 지연로딩이기 때문에 괜찮다.

## 지연로딩 실무 활용 예제

<center><img src="https://user-images.githubusercontent.com/36228833/198886987-b479b3fe-7487-47ef-a231-cb238665d3ad.png"></center>

- 상황
  - Member와 Team은 자주 함께 사용한다.(즉시 로딩)
  - Member와 Order는 가끔 사용한다.(지연 로딩)
  - Order와 Product는 자주 함꼐 사용한다.(즉시 로딩)
  - 사용 빈도에 따라 예를 들었으나 실무에서는 무조건 지연로딩으로 설정할 것을 권장한다.

<center><img src="https://user-images.githubusercontent.com/36228833/198887018-61166a08-fd8b-4f95-864d-4f4a6d740ebc.png"></center>

- Member와 Team은 즉시로딩으로 설정했기 때문에 한번에 조회 된다.
- Order는 지연로딩으로 설정했기 때문에 프록시 객체를 사용하여 조회한다.

<center><img src="https://user-images.githubusercontent.com/36228833/198887059-28f04e88-4334-4ce4-8a78-94cacc7e1047.png"></center>

- Order를 조회하는 순간 프록시 객체가 초기화 되서 조회되고
- Order와 Product는 즉시 로딩으로 설정되어있기 때문에 Order를 가져오는 시점에 Product까지 한번에 조회 된다.


  
## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
