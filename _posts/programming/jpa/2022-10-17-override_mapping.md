---
title: "고급 연관관계 매핑 - 상속관계 매핑 (작성중)"
categories: 
    - jpa
date: 2022-10-17
last_modified_at: 2022-10-17
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "상속관계 매핑을 알아보자"
---

## 상속관계 매핑 특징
- 관계형 데이터베이스는 상속관계X
- 슈퍼타입 서브타입 관계라는 모델링 기법이 객체 상속과 유사
- 상속관계 매핑 : 객체의 상속과 구조와 DB의 슈퍼타입 서브타입 관계를 매핑
- ![image](https://user-images.githubusercontent.com/36228833/196205771-58c57321-af08-40e3-89dd-45923f35fd19.png)
- JPA는 DB의 슈퍼타입 서브타입 어떤 식으로 구현하든 전부 구현 가능하도록 지원함.

## 주요 어노테이션
- `@Inheritence(strategy=InheritanceType.XXX)`
  - JOINED : 조인 전략
  - SINGLE_TABLE : 단일 테이블 전략 (default)
  - TABLE_PER_CLASS : 구현 클래스마다 테이블 전략

- `@DiscriminatorColumn(name="DTYPE")`

- `@DiscriminatorValue("XXX")`

## 구현 방법
- 슈퍼타입 서브타입 논리 모델을 실제 물리 모델로 구현하는 방법(전략들)
  - 각각 테이블로 변환 -> **조인 전략**
    - ![image](https://user-images.githubusercontent.com/36228833/196205923-61784370-8757-4ff0-9adb-00a68cfb7646.png)
    - ITEM 이라는 테이블을 만들고 ALBUM, MOVIE, BOOK 테이블을 만들어서 나눈뒤 필요할 때 JOIN으로 가져온다.
    - 예를 들어서 ALBUM 데이터를 추가할 때 ITEM에 대한 고유 ID는 ITEM 테이블에 두고 앨범의 아티스트 같은 고유 정보는 ALBUM 테이블에 둔다.
    - ITEM_ID를 PK면서 FK로 두어 JOIN 전략을 구성할 때 활용한다.
    - 보통 DTYPE 같이 구분 컬럼을 두어서 관리한다.
      ```java
      @Entity
      @Inheritance(strategy = InheritanceType.JOINED)
      public class Item {
        @Id @GenerateValuew
        private Long id;
        private String name;
        private int price;
      }

      @Entity
      public class Album extends Item {
        private String artist;
      }

      @entity
      public class Movie extends Item {
        private String director;
        private String actor;
      }

      @Entity
      public class Book extends Item {
        private String author;
        private String isbn;
      }
      ```
  - 통합 테이블로 변환 -> **단일 테이블 전략**
    - ![image](https://user-images.githubusercontent.com/36228833/196205991-02206acb-0aff-4fdb-98fe-d1f298d1abfa.png)
    - 논리 모델을 하나의 테이블로 합치는 전략
    - 하나의 PK를 두고 (ITEM_ID) 각각의 컬럼으로 구분하여 관리한다.
  - 서브타입 테이블로 변환 -> **구현 클래스마다 테이블 전략**
    - ![image](https://user-images.githubusercontent.com/36228833/196206103-af17b9fc-c9bd-4367-a41a-0ec8fb028115.png)
    - 조인 전략에 사용한 것 처럼 공통이 되는 ITEM 테이블을 두지 않고 각각 필요한 컬럼들을 전부 가지고 있는다.

## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
