---
title: "JPA) 필드와 컬럼 매핑"
categories: 
    - java
date: 2022-09-28
last_modified_at: 2022-09-29
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "Entity의 필드와 데이터베이스의 컬럼이 어떻게 매핑이 되는지 알아보자."
---

Entity 클래스 안에 필드를 대상으로 다양한 애노테이션을 적용하여 실제 테이블을 구성하듯 쉽게 정의할 수 있다.

## @Id
- 해당 애노테이션이 적용된 필드(컬럼)는 PK 제약조건이 적용된다. (PK 직접 할당)

## @GeneratedValue
- 해당 컬럼에 PK를 위한 auto increments 전략을 사용할 수 있다. (PK 자동 할당)

  |옵션|설명|사용예|
  |---|---|---|
  |`GenerationType.AUTO`|설정된 데이터베이스에 따라 자동으로 지정된다 (기본값)|`@GeneratedValue(strategy=Generation.AUTO)`|
  |`GenerationType.IDENTITY`|데이터베이스가 자동으로 auto_increment를 통해 기본키를 생성한다. 주로 MySQL, PostgreSQL, SQL Server, DB2가 사용한다.|`@GeneratedValue(strategy=Generation.IDENTITY)`|
  |`GenerationType.SEQUENCE`|@SequenceGenerator를 통해 설정된 데이터베이스의 시퀀스를 통해 자동으로 기본키를 생성한다. Oracle, PostgreSQL, DB2, H2가 사용한다.|`@GeneratedValue(strategy=Generation.SEQUENCE)`|
  |`GenerationType.TABLE`|@TableGenerator를 통해 설정된 키 생성용 테이블을 통해 데이터베이스가 자동으로 기본키를 생성한다.|`@GeneratedValue(strategy=Generation.TABLE)`|

## @Column
- 해당 애노테이션의 속성을 통하여 대상 컬럼에 제약조건 등을 설정할 수 있다.

  |속성|설명|사용예|
  |---|---|---|
  |`name`|DB 컬럼명을 정의한다.(설정하지 않을 시 필드명이 DB 컬럼명이 된다.)|`@Column(name="COLUMN_NAME")`|
  |`insertable`|`Entity`에 해당 컬럼의 값을 입력해도 DB에 반영되지 않는다.|`@Column(insertable=true)`|
  |`updatable`|`Entity`에 해당 컬럼의 값을 수정해도 DB에 반영되지 않는다.|`@Column(updatable=true)`|
  |`nullable`|NOT NULL 제약조건 여부를 설정할 수 있다.|`@Column(nullable=true)`|
  |`unique`|unique 제약조건 여부를 설정할 수 있다. (제약조건 이름 설정이 불가능하여 권장하지 않음)|`@Column(unique=true)`|
  |`columnDefinition`|문자열로 직접 제약조건을 작성할 수 있다.|`@Column(columnDefinition="varchar(10) default 'NONE'")`|
  |`length`|String 타입에만 사용되며, 문자열 길이 제약조건을 설정할 수 있다..|`@Column(length=10)`|
  |`precision`|BigDecimal 혹은 BigInteger 타입에 사용되며 소수점을 포함한 전체 자릿수를 설정할 수 있다.|`@Column(precision=5)`|
  |`scale`|BigDecimal 혹은 BigInteger 타입에 사용되며 소수의 자릿수를 설정할 수 있다.|`@Column(scale=2)`|

## @Enumerated
- 자바 Enum 타입 객체를 매핑해 DB에 저장할 수 있다. Enum의 특성에 따라 2가지 속성이 있다.

  |옵션|설명|사용예|
  |---|---|---|
  |`EnumType.ORDINAL`|입력받은 Enum 타입 객체의 순서를 저장한다. (기본값)|`@Enumerated(EnumType.ORDINAL)`|
  |`EnumType.STRING`|입력받은 Enum 타입 객체의 이름(값)을 저장한다.|`@Enumerated(EnumType.STRING)`|

- **주의**
  - Enum의 새로운 타입이 추가 될 때 순서가 바뀔 수 있으므로 가급적 `EnumType.ORDINAL` 사용은 권장하지 않는다.
  
## @Temporal
- 자바가 제공하는 날짜 타입과 DB의 날짜관련 타입을 매핑하여 저장할 수 있다.

  |옵션|설명|사용예|
  |---|---|---|
  |`TemporalType.DATE`|yyyy-mm-dd 형식의 date 타입을 저장한다.(필드 타입이 `LocalDate`인 경우와 동일)|`@Temporal(TemporalType.DATE)`|
  |`TemporalType.TIME`|hh:mm:ss 형식의 time 타입을 저장한다.|`@Temporal(TemporalType.TIME)`|
  |`TemporalType.TIMESTAMP`|yyyy-mm-dd hh:mm:ss 형식의 datetime(timestamp) 타입을 저장한다. (기본값, 필드 타입이 `LocalDateTime`인 경우와 동일)|`@Temporal(TemporalType.TIMESTAMP)`|

## @Lob
- BLOB, CLOB과 같이 최대 길이를 넘어서는 Large Object를 매핑하여 저장할 수 있다.

- 매핑타입이 문자일경우 CLOB으로 매핑
  - `String`, `char[]`, `java.sql.CLOB`

- 그외 나머지는 BLOB으로 매핑
  - `byte[]`, `java.sql.BLOB` 

## @Transient
- 메모리 상에서만 사용하고 싶은 필드를 지정할 수 있다.
- 실제 데이터베이스에 저장,수정,조회 되지 않는다.

## @CreateDate, @LastModifiedDate
- `@CreateDate`
  - `Entity`가 생성되고나서 저장될 때의 시간을 저장한다.
- `@LastModifiedDate`
  - 기존(조회한) `Entity`의 값이 변경될 때의 시간을 저장한다.

- JPA Auditing을 활용한 자동화
  - `@MappedSuperclass`
    - 해당 애노테이션이 적용된 `Entity`를 상속받은 경우, 상속받은 `Entity`의 필드를 컬럼으로써 인식한다.
  - `@EntityListeners(AuditingEntityListener.class)`
    - `Entity`에서 이벤트가 발생할 때마다 특정 로직을 실행시킬 수 있는 `@EntityListeners`의 인자로 JPA에서 Auditing 기능을 수행하는 리스너 객체인 `AuditingEntityListener`를 넘기면 해당 `Entity`에 선언된 `@CreateDate`, `@LastModifiedDate`를 추적하여 값 변경 시 해당 값을 자동으로 업데이트 해준다.

## 예제 구현
해당 내용들을 참고하여 다음과 같은 요구사항을 구현해보자.

1. 회원은 일반 회원과 관리자로 구분해야 한다.
2. 회원 가입일과 수정일이 있어야 한다.
3. 회원을 설명할 수 있는 필드가 있어야 한다. 이 필드는 길이 제한이 없다.

```java
public enum RoleType {
  GUEST, USER, ADMIN
}
```

```java
import javax.persistence.*; 
import java.time.LocalDate; 
import java.time.LocalDateTime; 
import java.util.Date; 

@Entity 
public class Member { 
  @Id 
  private Long id; 

  @Column(name = "name") 
  private String username;

  private Integer age;

  // 회원 역할 구분
  @Enumerated(EnumType.STRING) 
  private RoleType roleType;
  
  // 회원 가입일
  @Temporal(TemporalType.TIMESTAMP) 
  private Date createdDate;
  
  // 회원 수정일
  @Temporal(TemporalType.TIMESTAMP) 
  private Date lastModifiedDate;

  // 설명란
  @Lob 
  private String description; 

  //Getter, Setter… 
} 
```

## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
