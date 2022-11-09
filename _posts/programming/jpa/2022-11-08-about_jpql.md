---
title: "객체지향 쿼리 언어 - 개요 (작성중)"
categories: 
    - jpa
date: 2022-11-08
last_modified_at: 2022-11-08
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "객체지향 쿼리 언어에 대한 소개"
---

JPA는 다음과 같이 다양한 쿼리 방법을 지원한다.

- JPQL
  - JPA의 객체지향 쿼리 언어로 관계형 데이터베이스의 엔티티에 대한 쿼리를 만드는데 사용된다.
- JPA Criteria
  - JPQL을 자바 코드로 작성하도록 도와주는 빌더 클래스 API
- QueryDSL
  - JPA Criteria와 마찬가지로 JPQL을 자바 코드로 작성할수 있도록 도와주는 오픈소스 라이브러리
- Native SQL
  - 데이터베이스 표준 문법에서 벗어난 종속적인 쿼리가 필요할 때 사용
  - 예) Oracle의 connect by, partition by 등
- JDBC API, MyBatis, SpringJdbcTemplate,..

우리는 그중 JPQL을 알아보자.

## JPQL 소개

지금까지 우리는 엔티티를 조회할 때 다음과 같이 조건이 없는, 가장 단순한 방법을 사용했다.

- 엔티티매니저 활용
  - `entityManager.find()`
- 객체 그래프 탐색
  - `entity.getA().getB()`

그렇다면 `나이가 18살 이상`인 회원을 모두 검색과 같은 조건으로 검색할 때는 어떻게 해야될까?

데이터양이 많을 경우 위의 두가지 방식만으로는 한계가 있을 것이다.

JPA를 사용하면 엔티티 객체를 중심으로 개발해야 한다. (테이블은 매핑만 할 뿐이다.)

검색을 할때도 테이블이 아닌 엔티티 객체를 대상으로 검색을 해야한다. 그러나 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능하기 때문에 **DB로부터 필요한 데이터만을 가져오려면 검색조건이 포함된 SQL이 필요**하다.
(모든 DB 데이터를 들고오게되면 과도한 네트워크 비용 및 퍼포먼스 이슈가 발생하기 때문)

그래서 JPA는 **SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어를 제공**한다.

JPQL은 다음과 같은 특징을 지닌다.

- SQL 문법과 유사하며 ANSI 표준 문법인 `SELECT`, `FROM`, `WHERE`, `GROUP BY`, `HAVING`, `JOIN`을 지원한다.
- 테이블이 아닌 엔티티 객체를 대상으로 쿼리를 수행하는 객체 지향 쿼리이다.
  - SQL을 추상화하기 때문에 특정 데이터베이스 SQL에 의존하지 않는다.
  - 데이터베이스에 맞는 SQL 문법으로 번역이 되어 실제 테이블에 적용된다.


## JPQL 사용방법

사용 방법은 간단하다. 다음과 같이 `entityManager`가 제공하는 `createQuery`와 `createNativeQuery`를 사용한다.

### entityManager.createQuery()

`createQuery()`는 엔티티를 대상으로 쿼리를 수행할 수 있게 도와준다.

jpql문을 작성할 때는 다음과 같은 규칙이 있다.

- 엔티티와 엔티티의 속성은 대소문자를 구분한다.
- `SELECT`, `FROM`, `JOIN` 와 같은 JPQL 키워드는 대소문자 구분을 하지 않는다.
- alias(별칭)은 필수로 작성해야 하고 as는 생략 가능하다.

예를 들어 이름이 홍길동이며 나이는 18살 이상인 Member 엔티티를 조회할 때는 다음과 같이 작성한다.

`select m from Member m where m.name='홍길동' and m.age >= 18`



Member라는 Entity가 있다고 가정했을 때 아래와 같은 방법으로 사용할 수 있다.

```java
@Entity
public class Member {
  @Id
  @GeneratedValue
  private Long id;

  private String name;

  ...
}

...


// Member 엔티티를 대상으로 jpql문을 작성한다. 
String jpql = "select m from Member m where m.name like 'kim%'";

//createQuery의 첫번쨰 인자는 jpql 문자열이 들어가고 두번째 인자는 대상 엔티티 클래스 객체를 넣어준다.
List<Member> result = entityManager.createQuery(jpql, Member.class).getResultList();
```



### entityManager.createNativeQuery()

## Criteria 소개

JPQL의 경우 쿼리를 문자열로 작성해야하기 때문에 동적으로 작성하려면 문자열을 자르고 붙이는 작업이 동반되어 매우 지저분해지고 복잡해진다. 

jpql 동적쿼리 예) 

```java
String jpql = "select m from Member m";

if(username != null){
  String where = " where m.name = '" + username + "'"; // alias와 where 절을 떨어뜨려야 하기 때문에 앞부분 공백도 신경써야한다.
  jpql += where; 
}

List<Member> result = entityManager.createQuery(jpql, Member.class).getResultList();

```

이러한 단점을 보완하기 위해 Criteria가 등장했다. 

Criteria는 동적 쿼리를 작성하기 위한 메서드를 제공해주고 문자열이 아닌 자바 코드로 작성하기 때문에 컴파일 시점에 미리 오류를 잡아서 확인할 수 있다는 장점을 지닌다.

그러나 자바코드로 작성되기 때문에 SQL문과 다르고 

## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
