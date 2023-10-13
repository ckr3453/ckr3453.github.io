---
title: "JPA) 객체지향 쿼리 언어 - 프로젝션(SELECT)"
categories: 
    - java
date: 2023-01-17
last_modified_at: 2023-01-17
toc: true
toc_sticky: true
excerpt: "객체지향 쿼리 언어의 프로젝션"
---

프로젝션이란 SELECT 절에 조회할 대상을 지정하는것을 의미한다. 기 SQL의 경우 기본 타입들만 선택 가능했으나 JPQL의 경우 엔티티, 임베디드 타입, 스칼라 타입을 선택할 수 있다.

## 엔티티

엔티티 자신 혹은 연관관계의 엔티티를 호출할 수 있다.

- 엔티티 자신을 호출 
  - `SELECT m FROM Member m`
  - 선언한 엔티티의 alias 를 select 절에 넣는다.
  - 엔티티의 경우 쿼리 결과가 영속성 컨텍스트에 의해 관리된다.

    ```java
      List<Member> result = entityManager.createQuery("select m from Member m", Member.class).getResultList();

      Member member = result.get(0);
      member.setAge(11) // 영속성 컨텍스트에 의해 관리가 되고 있으므로 update 쿼리가 수행된다.
      ...
    ```

- 엔티티 연관관계를 호출
  - `SELECT m.team FROM Member m`
  - 이렇게 작성할 경우 작성한 쿼리와 다르게 **실제 쿼리는 join 쿼리로 수행**되어서 헷갈릴 수 있으며 결과를 예측하기 힘들다.
  - 귀찮더라도 다음과 같이 명시적으로 작성하자.
  - `SELECT t from Member m join m.team t`

## 임베디드 타입
임베디드 타입은 복합 값 타입이라고도 부르며 호출할 경우 해당 컬럼들만 조회한다.

- `SELECT m.address FROM Member m`
- address는 임베디드 타입이며 호출할 경우 address에 해당하는 컬럼들만 조회한다.

  ```java
  @Embeddable
  @Getter
  @Setter
  @AllArgsConstructor
  public class Address {
    private String city;
    private String street;
    private String zipcode;
  }

  ....

  @Entity
  @Getter
  @Setter
  public class Member {
    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @Embedded // 임베디드 값 타입을 사용
    private Address address;
  }

  ...

  /* SELECT city, street, zipcode FROM Member */
  List<Address> result = em.createQuery("SELECT m.address FROM Member m", Member.class).getResultList();
  
  ```

## 스칼라 타입 (숫자, 문자와 같은 기본 데이터 타입)
일반 SQL과 동일하게 숫자, 문자와 같은 기본 데이터 타입을 조회할 수 있다.

- `SELECT m.username, m.age FROM Member m`
- 일반 SQL을 사용하듯 distinct 를 사용하여 중복제거도 가능하다.
  - `SELECT distinct m.username, m.age FROM Member m`

만약 타입이 통일되지 않고 **여러 타입을 가진 결과를 조회**하려 할때 다음의 방법을 사용할 수 있다.

- Query 타입으로 조회
  - 타입이 명확한 경우
    - `TypedQuery<Member> query = em.createQuery("SELECT m FROM Member m", Member.class)`

  - 타입이 명확하지 않은 경우
    - `Query query = em.createQuery("SELECT m.username, m.age)`

- Object[] 타입으로 조회
  - `List<Object[]> resultList = em.createQuery("select m.username, m.age from Member m").getResultList();`

- new 명렁어로 조회 
  - `SELECT new jpa.jpql.common.MemberDTO(m.username, m.age) FROM Member m`
  - 단순 값을 DTO로 바로 조회
  - 캐스팅을 할 필요가 없기 떄문에 제일 깔끔하다.
  - 패키지 명을 포함한 전체 클래스 명을 입력해야한다.
  - 순서와 타입이 일치하는 생성자가 필요하다.

## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
