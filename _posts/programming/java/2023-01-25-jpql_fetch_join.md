---
title: "JPA) 객체지향 쿼리 언어 - 페치 조인(fetch join)"
categories: 
    - java
date: 2023-01-25
last_modified_at: 2023-01-29
toc: true
toc_sticky: true
excerpt: "객체지향 쿼리 언어의 페치 조인"
---

## 페치조인 이란
- SQL 조인 종류가 아니다.
- JPQL에서 성능 최적화를 위해 제공하는 기능이다.
- 연관된 엔티티나 컬렉션을 **SQL 한번에 함께 조회**하는 기능이다.
- join fetch 명령어를 사용한다.
  - `LEFT [OUTER | INNER] JOIN FETCH 조인 경로`

예를들어 회원을 조회하면서 연관된 팀도 함께 조회하고 싶다.

- 다음과 같이 JPQL 쿼리문을 작성한다.
  ```sql
  select m 
  from Member m join fetch m.team
  ```

- 그러면 실제 쿼리는 다음과 같이 수행된다.
  ```sql
  select m.*, t.* 
  from Member m inner join Team t on m.TEAM_ID = t.id
  ```
    - 회원 뿐만 아니라 팀도 함께 SELECT를 한다.
    - 마치 즉시 로딩(Eager Loading) 전략으로 수행하듯 진행된다.
    - fetch를 통해 명시적으로 즉시 로딩(연관 엔티티를 한번에 모두 조회) 전략을 사용할 수 있다.

## 일반조인, 페치조인 차이

회원 1과 회원 2가 팀A 소속이고, 회원 3이 팀B 소속일때 

회원과 회원이 속한 팀을 조회하고 싶다.

<center><img src="https://user-images.githubusercontent.com/36228833/214607363-b1f95bf7-5c10-4c34-b032-69b73cbdc0a9.png"></center>
<br/>

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory(...);
EntityManager em = emf.createEntityManager();

Team teamA = new Team();
teamA.setName("팀A");
em.persist(teamA);

Team teamB = new Team();
teamB.setName("팀B");
em.persist(teamB);

Member member1 = new Member();
member1.setUsername("회원1");
member1.setTeam(teamA);
em.persist(member1);

Member member2 = new Member();
member2.setUsername("회원2");
member2.setTeam(teamA);
em.persist(member2);

Member member3 = new Member();
member3.setUsername("회원3");
member3.setTeam(teamB);
em.persist(member3);

em.flush();
em.clear();

// 일반 조인
String query = "select m from Member m";
List<Member> members = em.createQuery(query, Member.class).getResultList();

for(Member member : members){
   System.out.println(member.getUsername() + ", " + member.getTeam().getName());
   // 팀 이름을 가져오기 위해 팀 조회 쿼리를 날리는 횟수 
   // 회원1, 팀A(영속성 컨텍스트에 없기 때문에 SQL로 조회)
   // 회원2, 팀A(영속성 컨텍스트내에 팀A가 있기때문에 1차 캐시로 조회)
   // 회원3, 팀B(영속성 컨텍스트에 없기 때문에 SQL로 조회)

   // 만약 최악의 경우, 회원 컬렉션 n사이즈 대로 팀 조회 쿼리가 n번 수행됨.
   // N+1 문제 라고도함. (회원 조회 쿼리 1번 수행 + 회원에 대한 팀 쿼리 조회 n번 수행)
}

// 페치 조인
String query = "select m from Member m join fetch m.team";
List<Member> members = em.createQuery(query, Member.class).getResultList();

for(Member member : members){
   System.out.println(member.getUsername() + ", " + member.getTeam().getName());
   // 일반 조인 방식과 다르게 쿼리 한번에 연관 엔티티까지 전부 조회한다. (즉시 로딩)
   // 회원과 팀에대한 모든 결과가 영속성 컨텍스트에 이미 담겨져 있다.
   // 그러므로 추가적인 조회 쿼리가 수행되지 않는다.
}
```

## 컬렉션 페치 조인

일대다 관계의 경우, 컬렉션 페치 조인 작성 예시

팀A에 속한 회원들을 조회하고 싶다.

- 작성한 JQPL 쿼리문
```sql
select t 
from Team t join fetch t.members 
where t.name = '팀A'
```

- 실제 수행된 쿼리
```sql
select T.*, M.* 
from Team T inner join Member M on T.id = M.TEAM_ID 
where T.name = '팀A' 
```

주의할 점은 일대다 관계인 컬렉션 페치 조인(다대일은 상관없음)을 수행할 경우 팀에속한 회원 수 만큼 row가 중복되서 나올 수 있다.<br/>
(예를 들어 member1, member2가 팀A 소속일 경우 팀A를 조회할 때 2개의 row가 조회됨. -> 수행한 쿼리를 통해 나온 개수 만큼 조회하기 떄문)

<center><img src="https://user-images.githubusercontent.com/36228833/214607682-8fae1d69-c16b-4d99-b6d0-a7206a3dba99.png"></center>

이럴 경우 JPQL의 DISTINCT를 사용하면 된다. JPQL의 DISTINCT는 다음과 같은 2가지 기능을 제공한다.
1. SQL에 DISTINCT를 추가하여 row 중복 제거 후 가져온다.
2. 1번 수행 결과를 가져와서 애플리케이션에서 엔티티 중복 제거를 추가로 수행한다.

```sql
select distinct t
from Team t join fetch t.members
where t.name = '팀A'
```

- JPQL은 결과를 반환할때 연관관계를 고려하지 않는다.
- 단지 SELECT 절에 지정한 엔티티만 조회할 뿐이다.
- 여기서는 팀 엔티티만 조회하고 회원 엔티티는 조회하지 않는다.
  - 컬렉션도 단일건과 동일하게 일반 조인으로 조회 후 loop로 하나하나 꺼낼때 회원 엔티티를 조회하기 위해 매번 쿼리를 수행하게된다.
- **대부분의 N+1 문제는 fetch join으로 해결 가능하다!**

## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
