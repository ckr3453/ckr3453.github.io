---
title: "필드와 컬럼 매핑 (작성중)"
categories: 
    - jpa
date: 2022-09-28
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

## @Id
- 해당 애노테이션이 적용된 필드(컬럼)는 PK 제약조건이 적용된다.

## @Column
- 해당 애노테이션의 속성을 통하여 대상 컬럼에 제약조건 등을 설정할 수 있다.
  |옵션|설명|사용예|
  |---|---|---|
  |name|DB 컬럼명을 정의한다.(설정하지 않을 시 필드명이 DB 컬럼명이 된다.)|`@Column(name="COLUMN_NAME")`|
  |insertable|(작성중)|`@Column(insertable=true)`|
  |updatable|(작성중)|`@Column(updatable=true)`|
  |nullable|(작성중)|`@Column(nullable=true)`|
  |unique|(작성중)|`@Column(unique=true)`|
  |columnDefinition|(작성중)|`@Column(columnDefinition="varchar(10) default 'NONE'")`|
  |length|(작성중)|`@Column(length=10)`|
  |precision|(작성중)|`@Column(precision=5)`|
  |scale|(작성중)|`@Column(scale=2)`|


해당 내용들을 다음과 같은 요구사항이 있을 때 이런식으로 작성할 수 있다.

1. 회원은 일반 회원과 관리자로 구분해야 한다.
2. 회원 가입일과 수정일이 있어야 한다.
3. 회원을 설명할 수 있는 필드가 있어야 한다. 이 필드는 길이 제한이 없다.

## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
