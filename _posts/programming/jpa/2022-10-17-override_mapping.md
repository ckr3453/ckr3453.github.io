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
  - **조인 전략** (각각의 테이블로 변환)
    - ![image](https://user-images.githubusercontent.com/36228833/196205923-61784370-8757-4ff0-9adb-00a68cfb7646.png)
    - ITEM 이라는 테이블을 만들고 ALBUM, MOVIE, BOOK 테이블을 만들어서 나눈뒤 필요할 때 JOIN으로 가져온다.
    - 예를 들어서 ALBUM 데이터를 추가할 때 ITEM에 대한 고유 ID는 ITEM 테이블에 두고 앨범의 아티스트 같은 고유 정보는 ALBUM 테이블에 둔다.
    - ITEM_ID를 PK면서 FK로 두어 JOIN 전략을 구성할 때 활용한다.
    - 보통 DTYPE 같이 구분 컬럼을 두어서 관리한다.
      ```java
      @Entity
      @Inheritance(strategy = InheritanceType.JOINED)
      @DiscriminatorColumn(name = "ITEM_TYPE")
      public class Item {
        @Id @GenerateValuew
        private Long id;
        private String name;
        private int price;
      }

      @Entity
      @DiscriminatorValue
      public class Album extends Item {
        private String artist;
      }

      @Entity
      @DiscriminatorValue("M")
      public class Movie extends Item {
        private String director;
        private String actor;
      }

      @Entity
      @DiscriminatorValue
      public class Book extends Item {
        private String author;
        private String isbn;
      }

      ...
      
      Movie movie = new Movie(); 
      movie.setDirector("aaaa");
      movie.setActor("bbbb");
      movie.setName("테스트영화");
      movie.setPrice(10000);

      // ITEM 테이블과 MOVIE 테이블 2개에 insert가 되고
      // MOVIE 테이블의 ID는 PK면서 ITEM 테이블의 FK 역할을 하게된다.
      em.persist(movie);

      em.flush();
      em.clear();

      // MOVIE 테이블과 ITEM 테이블을 ID로 inner join하여 가져옴
      em.find(Movie.class, movie.getId());
      ```
    - (부모 테이블 입장인) Item 엔티티에 `@DiscriminatorColumn`를 추가하면 ITEM 테이블에 어떤 종류의 ITEM 인지 명시해주는 `DTYPE` 컬럼이 추가된다.
      - default는 `DTYPE` 이나 name 속성으로 컬럼 명을 변경할 수 있다. (`@DiscriminatorColumn(name="ITEM_TYPE")`)
      - 데이터베이스 입장에서는 row 만으로는 구분할 수 없기 때문에(어떤 종류의 ITEM 인지) `DTYPE` 이 있는 것이 좋다.
    - (자식 테이블 입장인) Movie 엔티티에 `@DiscriminatorValue("M")`를 추가하면 `DTYPE` 컬럼에 들어갈 값을 Movie가 아닌 M으로 지정할 수 있다.
      - default는 해당 엔티티 이름이다 (ex. Movie)
    - 장점
      - 테이블이 기본적으로 정규화가 되어있다.
      - 외래키 참조 무결성 제약조건을 활용 가능하다.
        - 필요에 따라 외래키 참조를 통해 헤매지 않고 바로 데이터 조회를 할 수 있다.
        - 예를 들어 주문 내역을 확인할 때 A 영화의 가격이 궁금하다면? (MOVIE join ITEM)
      - 저장공간이 효율화 된다.
        - 테이블들이 정규화가 되어있기 때문에 효율적이다.
    - 단점
      - 조회 시 조인을 많이 사용하기 때문에 성능이 저하된다.
      - 조회 쿼리가 복잡해진다.
      - 데이터 저장시 INSERT SQL을 2번 호출하게 된다. 
        - ITEM 테이블 한번, MOVIE 테이블 한번

  - **단일 테이블 전략**(통합 테이블로 변환)
    - ![image](https://user-images.githubusercontent.com/36228833/196205991-02206acb-0aff-4fdb-98fe-d1f298d1abfa.png)
    - 논리 모델을 하나의 테이블로 합치는 전략
    - 하나의 PK를 두고 (ITEM_ID) 각각의 컬럼으로 구분하여 관리한다.
      ```java
      @Entity
      @Inheritance(strategy = InheritanceType.SINGLE_TABLE)
      public class Item {
        @Id @GenerateValuew
        private Long id;
        private String name;
        private int price;
      }

      @Entity
      @DiscriminatorValue
      public class Album extends Item {
        private String artist;
      }

      @Entity
      @DiscriminatorValue
      public class Movie extends Item {
        private String director;
        private String actor;
      }

      @Entity
      @DiscriminatorValue
      public class Book extends Item {
        private String author;
        private String isbn;
      }
      ```
    - 단일 테이블 전략 사용 시 부모 역할을 하는 ITEM 테이블에 ALBUM, MOVIE, BOOK 테이블의 컬럼이 일렬로 전부 추가 된다.
      - ALBUM, MOVIE, BOOK 테이블은 따로 생성되지 않는다.
      - 테이블 하나에 모든 컬럼이 포함되있기 때문에 성능상 매우 유리하다. (쿼리 복잡도가 낮으므로)
    - 단일 테이블 전략은 `@DiscriminatorColumn`이 없어도 `DTYPE` 컬럼이 자동 생성된다.
      - 조인 전략은 테이블이 나뉘어져 있기 때문에 `DTYPE` 컬럼이 없어도 다른 방법을 통해 알아낼 수 있다.
        - FK를 기준으로 조인쿼리를 날린 다던지 등
      - 그러나 단일 테이블 전략은 **하나의 테이블에 전부 들어가 있기 때문에 구분할 수 있는 `DTYPE` 컬럼이 필수로 있어야 한다.**

  - 서브타입 테이블로 변환 -> **구현 클래스마다 테이블 전략**
    - ![image](https://user-images.githubusercontent.com/36228833/196206103-af17b9fc-c9bd-4367-a41a-0ec8fb028115.png)
    - 조인 전략에 사용한 것 처럼 공통이 되는 ITEM 테이블을 두지 않고 자식 테이블이 각각 필요한 컬럼들을 전부 가진다.
      ```java
      @Entity
      @Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
      public abstract class Item {
        @Id @GenerateValuew
        private Long id;
        private String name;
        private int price;
      }

      @Entity
      @DiscriminatorValue
      public class Album extends Item {
        private String artist;
      }

      @Entity
      @DiscriminatorValue
      public class Movie extends Item {
        private String director;
        private String actor;
      }

      @Entity
      @DiscriminatorValue
      public class Book extends Item {
        private String author;
        private String isbn;
      }

      Movie movie = new Movie(); 
      movie.setDirector("aaaa");
      movie.setActor("bbbb");
      movie.setName("테스트영화");
      movie.setPrice(10000);

      em.persist(movie);

      em.flush();
      em.clear();

      // ALBUM, MOVIE, BOOK 테이블을 union all로 조회
      em.find(Item.class, movie.getId());
      ```
    - 직접 사용하는게 아니기 때문에 부모 테이블의 역할인 ITEM 엔티티를 추상 클래스로 작성해야 한다.
    - ALBUM, MOVIE, BOOK 테이블에 각각 ID 컬럼이 생성된다.
    - 단순하게 값을 넣고 뺄때는 좋지만 **부모 타입의 엔티티를 조회할 때 문제**가 생긴다.
      - ALBUM, MOVIE, BOOK 테이블에 전부 ITEM 테이블에게서 받은 ID 컬럼이 존재하므로 JPA가 Union all로 조회하기 때문이다.
## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
