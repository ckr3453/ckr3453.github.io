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
excerpt: "객체지향 쿼리 언어 - JPQL에 대한 소개"
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


## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
