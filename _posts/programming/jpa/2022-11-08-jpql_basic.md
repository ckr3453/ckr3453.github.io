---
title: "객체지향 쿼리 언어 - 기본문법과 쿼리 API"
categories: 
    - jpa
date: 2022-11-11
last_modified_at: 2022-11-11
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "객체지향 쿼리 언어의 기본 문법 및 쿼리 API"
---

## JPQL 문법

JPQL의 기본 문법에 대해서 알아보자.

![image](https://user-images.githubusercontent.com/36228833/201143241-4012ba40-424e-4120-a432-722d0c10c35b.png)

![image](https://user-images.githubusercontent.com/36228833/201143346-d27c4dbc-d47e-4b2f-b4a4-af73c5438f37.png)

작성하는 방식은 기본적으로 ANSI SQL 표준 문법과 같으며 주의할 점은 **테이블이 아닌 엔티티 객체를 대상으로 쿼리문을 작성**해야 된다는 것이다.

## JPQL 반환 타입

JPQL은 쿼리문의 반환 타입이 명확할 때와 명확하지 않을 때 사용하는 타입이 다르다.

- `TypedQuery<T>`
  - 반환 타입이 명확한 경우
  - 예) `entityManager.createQuery("select m from Member m", Member.class)`
    - m은 Member 엔티티를 의미한다.
    - 반환해야 하는 타입이 1가지일 경우 타입을 명시하여 가져올 수 있다.<br/><br/>

- `Query`
  - 반환 타입이 명확하지 않은 경우
  - 예) `entityManager.createQuery("select m.name, m.age from Member m")`
    - m.name은 String 이고 m.age는 Integer 다.
    - 반환해야하는 타입이 2가지 이상인 경우 타입을 명시할 수 없다.

```java
@Entity
@Getter
@Builder
public class Member {
    @Id @GeneratedValue
    private Long id;

    private String name;

    private Integer age;
}

...

// 타입을 명시한 경우
TypedQuery<Member> query = em.createQuery("select m from Member m", Member.class);

// 타입을 명시하지 않은 경우
Query query = em.createQuery("select m.name, m.age from Member m");
```

## 결과 조회 API

JPQL은 쿼리의 결과 갯수에 따라 리스트 혹은 단일 객체로 반환 받을 수 있다.

- `query.getResultList()`
  - 쿼리의 **결과가 하나 이상일 때** 리스트로 반환한다.
  - 결과가 없으면 빈 리스트를 반환한다.<br/><br/>

- `query.getSingleResult()`
  - 쿼리의 **결과가 정확히 하나일 때** 단일 객체로 반환 한다.
  - 정확히 하나가 아닐 떄 다음과 같은 예외를 발생시킨다.
    - 결과가 없을때 : `javax.persistence.NoResultException`
    - 결과가 둘 이상일 때 : `javax.persistence.NonUniqueResultException`
  - 결과가 정확하게 하나가 아니면 예외가 발생되기 때문에 try-catch로 작성해야한다. (잘 사용되지 않는다.)


```java
Member member = new Member();
member.setName("member1");
member.setAge(10);
em.persist(member);

// 결과가 하나 이상인 경우 리스트로 반환
TypedQuery<Member> query = em.createQuery("select m from Member m", Member.class);
List<Member> resultList = query.getResultList();

for(Member member : resultList){
    System.out.println("member = " + member);
}

// 결과가 하나인 경우 단일 객체로 반환
TypedQuery<Member> query = em.createQuery("select m from Member m where m.name = 'member1' and m.age = 10", Member.class);

try{
    Member singleResult = query.getSingleResult();
    System.out.println("member = " + singleResult);
} catch(NoResultException e) {
    // do something
} catch(NonUniqueResultException e) {
    // do something
}
```

## 파라미터 바인딩

JPQL을 사용할 때 파라미터를 바인딩 하는 방법을 알아보자.

파라미터를 바인딩은 인덱스 혹은 직접 지정 하는 방법이 있다.

- 이름으로 직접 지정
  - 쿼리 문자열을 작성할 때 콜론(:) + 이름 조합으로 파라미터 바인딩을 받을 위치에 작성한다.
    - 예) select m from Member m where m.name = **:username**
  - 그 후 `query.setParameter("이름", 파라미터)`를 사용하여 파라미터를 바인딩 시킬 수 있다.
    - 예) query.setParameter("username", name) <br/><br/>

- 인덱스(위치)로 지정
  - 쿼리 문자열을 작성할 때 ? + 인덱스(위치) 조합으로 파라미터 바인딩을 받을 위치에 작성한다.
    - 예) select m from Member m where m.name = **?1**
  - 그 후 `query.setParameter(인덱스(위치), 파라미터)`를 사용하여 파라미터를 바인딩 시킬 수 있다.
    - 예) query.setParameter(1, name)
  - 위치 지정의 경우 중간에 파라미터가 빠지거나 빼야할 경우가 생기면 **위치가 전부 바뀌어 버리므로 권장하지 않는다.**

```java
// 이름으로 지정
TypedQuery<Member> query = em.createQuery("select m from Member m where m.name = :username and m.age = :age", Member.class)
                                .setParameter("username", "member1")
                                .setParameter("age", 10);
Member result = query.getSingleResult();

// 위치로 지정
TypedQuery<Member> query = em.createQuery("select m from Member m where m.name = ?1 and m.age = ?2", Member.class)
                                .setParameter(1, "member1")
                                .setParameter(2, 10);
Member result = query.getSingleResult();
```


## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
