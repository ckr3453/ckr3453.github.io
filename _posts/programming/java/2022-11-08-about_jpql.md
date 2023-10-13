---
title: "JPA) 객체지향 쿼리 언어 - 개요"
categories: 
    - java
date: 2022-11-08
last_modified_at: 2022-11-10
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


각 쿼리 방법에 대한 간략한 소개와 사용 예를 알아보자.

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


## JPQL 사용 예

사용 방법은 간단하다. 다음과 같이 `entityManager`가 제공하는 `createQuery`를 사용한다.

`createQuery()`는 엔티티를 대상으로 쿼리를 수행할 수 있게 도와준다.

jpql문을 작성할 때는 다음과 같은 규칙이 있다.

- 엔티티와 엔티티의 속성은 대소문자를 구분한다.
- `SELECT`, `FROM`, `JOIN` 와 같은 JPQL 키워드는 대소문자 구분을 하지 않는다.
- alias(별칭)은 필수로 작성해야 하고 as는 생략 가능하다.

예를 들어 이름이 홍길동이며 나이는 18살 이상인 Member 엔티티를 조회할 때는 다음과 같이 작성한다.

```java
@Entity
public class Member {
  @Id
  @GeneratedValue
  private Long id;

  private String name;

  private Integer age;
}

...


// Member 엔티티를 대상으로 jpql문을 작성한다. 
String jpql = "select m from Member m where m.name ='홍길동' and m.age >= 18 ";

//createQuery의 첫번쨰 인자는 jpql 문자열이 들어가고 두번째 인자는 대상 엔티티 클래스 객체를 넣어서 결과 타입을 정해준다.
List<Member> result = entityManager.createQuery(jpql, Member.class).getResultList();
```

실제 sql문은 다음과 같이 번역되어 테이블에 날린다.

```sql
select
  m.id as id,
  m.age as age,
  m.name as name
from
  Member m
where
  m.age >= 18
```

## JPA Criteria 소개

JPQL의 경우 쿼리를 문자열로 작성해야하기 때문에 동적으로 작성하려면 문자열을 자르고 붙이는 작업이 동반되어 매우 지저분해지고 복잡해진다. 

JPQL의 동적쿼리 예) 

```java
String jpql = "select m from Member m";

if(username != null){
  String where = " where m.name = '" + username + "'"; // alias와 where 절을 떨어뜨려야 하기 때문에 앞부분 공백도 신경써야한다.
  jpql += where; 
}

List<Member> result = entityManager.createQuery(jpql, Member.class).getResultList();

```

이러한 단점을 보완하기 위해 Criteria가 등장했다. 

Criteria는 다음과 같은 특징을 가진다.

- 문자열이 아닌 자바 코드로 JPQL 쿼리를 작성한다.
- 동적 쿼리를 작성하기 위한 메서드를 제공한다.
- 자바 코드로 작성하기 때문에 컴파일 시점에 문법 오류를 찾을 수 있다.
- 그러나 `SELECT`, `FROM` 등과 같은 키워드까지 하나하나 메서드로 작성해야 하기 때문에 가독성이 떨어지며 복잡하다.
  - 쿼리 복잡도가 증가할수록 **코드 또한 복잡해지며 유지보수가 힘든 형태**가 됨
  - 그러므로 **실무에서 사용을 권장하지 않음**

### JPA Criteria 사용 예

Criteria는 `CriteriaBuilder`, `CriteriaQuery` 객체를 중심으로 쿼리를 동적으로 생성할 수 있는 환경을 구성할 수 있다.

예를 들어 이름이 홍길동인 Member 엔티티를 조회할 때는 다음과 같이 작성한다.

```java
// Criteria 사용을 위해 CriteriaBuilder, CriteriaQuery 객체 생성
CriteriaBuilder cb = entityManager.getCriteriaBuilder();
CriteriaQuery<Member> query = cb.createQuery(Member.class);

// 메서드가 전체적으로 동적 쿼리에 유리하게 구성됨
// 엔티티 조회를 위해 Root 객체를 생성 (select m from Member m)
Root<Member> member = query.from(Member.class);

// select(), where() 메서드 조합을 통해 최종 쿼리를 생성함.
// 이름이 홍길동인 Member를 전부 검색하는 쿼리 생성 (select m from Member where m.name = '홍길동')
CriteriaQuery<Member> resultQuery = query.select(member).where(cb.equal(member.get("name"), "홍길동"));

List<Member> resultList = entityManager.createQuery(resultQuery).getResultList();
```

이외에도 조건에따라 `CriteriaQuery.groupBy()`, `CriteriaQuery.orderBy()` 등을 사용하여 쿼리를 구성할 수 있고 

`CriteriaBuilder.max()`, `CriteriaBuilder.min()`, `CriteriaBuilder.greaterThan()` 등 쿼리 조건에 들어가는 값들을 객체로 만들어 쉽게 동적 쿼리를 구성할 수 있다.

자세한 내용은 [하이버네이트 공식문서](https://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html#criteria)를 참고하자.

동적 쿼리 작성을 위한 수많은 메서드를 제공하지만 **오히려 그게 독이 되어 유지보수가 어려워지는 단점**이 있다. 그래서 실무에서는 잘 쓰이지 않는다.

## QueryDSL 소개

JPA에서 제공하는 Criteria가 너무 복잡하여 그에 대한 대안으로 등장한 것이 QueryDSL 이다.

QueryDSL의 특징은 다음과 같다.

- QueryDSL은 오픈소스 라이브러리다.
- 사용하기 위한 초기 설정 세팅이 필요하다.
- 문자열이 아닌 자바 코드로 JPQL 쿼리를 작성한다.
- 동적 쿼리를 작성하기 위한 메서드를 제공한다.
  - 수많은 메서드 조합으로 복잡하게 구성해야했던 Criteria와 달리 **직관적이며 단순한 메서드를 제공**한다.
- 자바 코드로 작성하기 때문에 컴파일 시점에 문법 오류를 찾을 수 있다.
- **실무에서 사용을 권장**한다.

### QueryDSL 사용 예

```java
// QueryDSL 사용을 위해 JPAFactoryQuery 객체 생성
JPAFactoryQuery query = new JPAQueryFactory(entityManager);
QMember m = QMember.member;

// select m from Member m where m.age > 18
List<Member> list = query.selectFrom(m)
                          .where(m.age.gt(18))
                          .orderBy(m.name.desc())
                          .fetch();
```

QueryDSL은 오픈소스 라이브러리로써 JPQL 동적 쿼리 생성을 쉽게할 수 있고 메서드 또한 복잡하지 않고 간단하다는 장점이 있다.

## Native SQL 소개

Native SQL은 JPQL로 해결할 수 없는, 즉 특정 DB에 종속된 기능(키워드)를 사용하여 쿼리를 구성해야할 때 유용하다.
(Oracle - CONNECT BY, 특정 DB의 SQL 힌트 등)

### Native SQL 사용 예

사용 방법은 간단하다. `entityManager`가 제공하는 `createNativeQuery()`를 사용하면 된다.

예를 들어 이름이 홍길동이며 나이가 18살 이상인 Member를 조회하는 쿼리를 작성할 때 다음과 같다.

```java
// 실제 SQL 쿼리 작성하듯 작성
String sql = "select * from Member where name = '홍길동' and age >= 18";

List<Member> list = entityManager.createNativeQuery(sql, Member.class).getResultList();
```

사용시 특별한 제한은 존재하지 않으며 실제 SQL 쿼리를 작성하듯 작성하면 된다.

## JDBC API 직접사용, MyBatis, SpringJdbcTemplate 등

JPA를 사용하면서 JDBC 커넥션을 직접 사용하거나, SpringJdbcTemplate, MyBatis 등을 함께 사용할 수 있다.

`createQuery()`, `createNativeQuery()`를 통해 JPQL 쿼리를 수행하면 **내부적으로 영속성 컨텍스트가 자동으로 flush를 수행**하여 최신 반영된 내용을 조회한다. (기본 전략이 auto flush로 설정 되어있음)

그러나 JDBC API의 경우 별도로 flush를 수행하지 않기 때문에 **영속성 컨텍스트를 적절한 시점에 강제로 flush** 해야한다.

```java
Member member = new Member();
member.setName("member1");
entityManager.persist(member);

Connection conn = DriverManager.getConnection(url, user, pw);
Statement st = conn.createStatement();

// 결과 0 (flush가 일어나지 않았으므로 위에서 추가한 Member가 반영되지않음)
ResultSet rs = st.executeQuery("select * from Member");
```

다음과 같이 flush를 수동으로 수행해야 한다.

```java
Member member = new Member();
member.setName("member1");
entityManager.persist(member);

entityManager.flush(); // flush를 수동으로 수행

Connection conn = DriverManager.getConnection(url, user, pw);
Statement st = conn.createStatement();

// 결과 1
ResultSet rs = st.executeQuery("select * from Member");
```

## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
