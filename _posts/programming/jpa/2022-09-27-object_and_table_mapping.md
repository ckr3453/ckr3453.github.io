---
title: "객체와 테이블 매핑"
categories: 
    - jpa
date: 2022-09-27
last_modified_at: 2022-09-28
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "JPA에서 객체와 데이터베이스가 어떻게 매핑이 되는지 알아보자."
---
## @Entity
- `@Entity`가 붙은 클래스는 JPA가 관리하며, `Entity`라고 한다.
- JPA를 사용해서 테이블과 매핑할 클래스는 `@Entity` 애노테이션이 필수다.
- **사용시 유의할 점**
  - 기본 생성자가 필수로 있어야 함 (파라미터가 없는 public 혹은 protected 생성자)
    - JPA 스펙 상에 명시되어 있음 : JPA에서 다양한 기술 구현을 위한 프록시 패턴을 사용하기 위해서
  - `enum`, `interface` 및 `final` 클래스, `inner` 클래스는 사용할 수 없다.
  - 데이터베이스에 저장할 대상이 되는 필드에 `final` 키워드를 사용할 수 없다.
- **속성**

  |속성|기능|기본값|사용예|
  |---|---|---|---|
  |`name`|매핑할 `Entity` 이름|클래스 이름을 사용|`@Entity(name = "Member")`|

## @Table
- `@Table`은 `Entity`와 실제로 매핑할 테이블 이름을 지정
- **속성**

  |속성|기능|기본값|사용예|
  |---|---|---|---|
  |`name`|매핑할 테이블 이름|엔티티 이름을 사용|`@Table(name="EntityName")`|
  |`catalog`|데이터베이스 catalog 매핑||`@Table(catalog="YOUR_CATALOG")`|
  |`schema`|데이터베이스 schema 매핑||`@Table(schema="YOUR_SCHEMA")`|
  |`uniqueConstraints`|DDL 생성 시에 유니크 제약조건을 생성||`@Table(uniqueConstraints={@UniqueConstraint(name = "NAME_AGE_UNIQUE", columnNames = {"NAME", "AGE"})}`|

## 데이터베이스 스키마 자동 생성 기능
- JPA는 `Entity` 대상이 되는 클래스를 참조하여 애플리케이션 실행 시점에 DDL을 자동 생성한다.
  - 설정된 데이터베이스의 방언을 활용하여 그에 맞는 적절한 DDL을 생성한다.

- **테이블 중심이 아닌 객체 중심으로 개발을 할 수 있어** 테스트 환경에서 매우 유용하다.
  - 개발자는 테이블을 신경쓰지 않고 매핑의 대상이 되는 `Entity`만 고려하여 개발 가능하다.

- 이렇게 생성된 DDL은 **개발 로컬 환경에서만 사용**하는 것을 권장한다.
  - 생성된 DDL은 운영서버에서 적절히 다듬거나 사용하지 않는 것을 추천한다. (안정성 보장X)

- **속성**
  - `hibernate.hdm2ddl.auto`
    - JPA 사용을 위한 설정 정보 옵션중 하나로 애플리케이션 실행 시점에 DDL을 핸들링 할 수 있다.

      |옵션|설명|
      |---|---|
      |`create`|기존 테이블 삭제 후 다시 생성 (DROP + CREATE)|
      |`create-drop`|`create`와 같으나 종료 시점에 테이블 DROP (테스트에서 사용하면 도움 됨)|
      |`update`|변경 분만 반영 **(운영 DB에서는 사용하면 안됨)**|
      |`validate`|엔티티와 테이블이 정상 매핑되었는지만 확인|
      |`none`|아무 속성도 사용하지 않음|

    - **사용시 유의할 점**
      - **운영 장비에는 절대 `create`, `create-drop`, `update`를 사용하면 안된다.**
      - 개발 초기 단계는 `create` 또는 `update` 를 사용
      - 테스트 서버는 `update` 또는 `validate` 를 사용
        - 여러명이 함께 사용하는 테스트 서버의 경우, `create` 등을 사용하여 데이터가 날아갈 경우 문제가 발생할 수 있다.
        - 왠만해서는 테스트 서버에서도 `validate` 또는 `none`을 사용하는 것이 좋다.
          - 대용량의 데이터가 있는 상태에서 `update`를 적용하여 `alter ~`가 적용됐을 경우, 자칫하면 **시스템이 중단될 수도 있다.**
          - 가급적이면 직접 스크립트를 짜서 적용해보고 문제가 없을 시 DBA에게 확인을 받은 후 적용하는 것을 권장한다.
      - 스테이징과 운영 서버는 `validate` 또는 `none` 를 권장한다.
      - **정리**
        - 혼자 사용하는 로컬 환경에서는 사용해도 괜찮으나, **다른 사람과 같이 사용하는 환경에서는 가급적 사용하지 않는 것을 권장**한다.

## DDL 생성 기능
- `@Column`
  - DDL 생성 시 컬럼 별 제약조건을 추가할 수 있다.
  - `@Column(nullable=false, unique=true, length=10)`
    - 해당 컬럼은 NOT NULL이며, unique_key이고 길이는 10 이하 이다.
- `@Table(uniqueConstraints={@UniqueConstraint(name = "NAME_AGE_UNIQUE", columnNames = {"NAME", "AGE"})}`
  - 위와 같이 `@Table`의 `uniqueConstraints` 속성을 사용하여 unique 제약조건을 걸 수 있다.
  - `NAME`과 `AGE` 컬럼을 unique_key 제약 조건으로 설정한다.

- 본 기능은 DDL을 자동 생성할 때만 사용되고 애플리케이션의 실행 로직에는 영향을 주지 않는다.

## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
