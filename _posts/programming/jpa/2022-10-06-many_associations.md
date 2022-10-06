---
title: "연관관계 매핑 - 다양한 연관관계 (작성중"
categories: 
    - jpa
date: 2022-09-30
last_modified_at: 2022-10-04
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "다양한 연관관계를 알아보자"
---

## 연관관계 매핑시 고려사항
- 다중성
  - 1:N(`@OneToMany`)
  - N:1(`@ManyToOne`)
  - 1:1(`@OneToOne`)
  - N:M(`@ManyToMany`)
    - 권장하지 않음.

- 단방향, 양방향
  - 테이블
    - 외래키 하나로 양쪽 조인이 가능하다.
    - 방향이라는 개념 X
  - 객체
    - 참조용 필드가 있는 쪽으로만 참조 가능
    - 한쪽만 참조하면 단방향
    - 양쪽이 서로 참조하면 양방향

- 연관관계의 주인
  - 테이블은 외래키 하나로 두 테이블이 연관관계를 맺음.
  - 객체 양방향 관계는 A -> B, B -> A 처럼 참조가 2군데임.
  - 객체 양방향 관계는 참조가 2군데있음. 둘중 테이블의 외래키를 관리할 곳을 지정해야함.
  - 연관관계의 주인 : 외래키를 관리하는 참조
  - 주인의 반대편 : 외래키에 영향을 주지않고 단순 조회만 (read-only)

## 다대일 (N:1)

다대일의 관계에서 다(N)를 연관관계의 주인으로 설정한다.

- 다대일 단방향
  - ![image](https://user-images.githubusercontent.com/36228833/194346109-b38a9955-1d7a-45f4-8339-9c79bff2f259.png)
  - DB 입장에선 Member와 Team의 관계가 N:1이므로 외래키가 Member쪽에 속한다.
    - N쪽에 항상 외래키가 존재해야함.
  - 객체 입장에선 **외래키가 존재하는 엔티티에 참조용 필드를 만들어서 연관관계를 설정**하면 된다.
    ```java
    @Entity
    public class Member {
      ...
      @ManyToOne
      @JoinColumn(name = "TEAM_ID")
      private Team team;
    }
    ```

- 다대일 양방향
  - ![image](https://user-images.githubusercontent.com/36228833/194346248-7ab7820b-90d4-47c3-a5b8-0a4f6c3d2472.png)
  - 연관관계 주인으로 설정된 **반대편에 참조용 객체를 생성**하여 이어주면 된다.
    ```java
    @Entity
    public class Team {
      ...
      @OneToMany(mappedBy = "team")
      private List<Member> members = new ArrayList<>();
    }
    ```
  - 반대쪽에 참조용 객체를 추가한다고 해도 테이블에 영향을 주지않는다.
    - 어차피 read-only 이므로
  - **외래키가 있는 쪽**이 연관관계의 주인이다.
  - 양쪽을 서로 참조하도록 개발한다.

가장 많이 사용하는 연관관계이며 **다대일**의 반대는 **일대다** 이다.

## 일대다 (1:N)

일대다의 관계에서 일(1)을 연관관계 주인으로 설정할 수 있다. (권장하지 않음)

- 일대다 단방향
  - ![image](https://user-images.githubusercontent.com/36228833/194346363-a2626a73-639e-4ead-9603-637e069bf26d.png)
  - 1의 입장인 `Team`에서 연관관계의 주인으로써 관리하려고 할때





## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
