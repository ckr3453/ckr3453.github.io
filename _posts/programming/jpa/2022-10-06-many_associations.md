---
title: "연관관계 매핑 - 다양한 연관관계 (작성중"
categories: 
    - jpa
date: 2022-10-06
last_modified_at: 2022-10-07
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
  - 설계 의도
    - `Member`가 속한 `Team`을 추가하거나 수정하고 싶을 때
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
  - 설계 의도
    - `Member` 입장에서 `Member`가 속한 `Team`도 추가하거나 수정할 수 있고
    - `Team` 입장에서 `Team`에 속한 `Member`들을 조회해야 할 때
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

일대다의 관계에서 일(1)을 연관관계 주인으로 설정할 수 있다.

- 일대다 단방향
  - ![image](https://user-images.githubusercontent.com/36228833/194346363-a2626a73-639e-4ead-9603-637e069bf26d.png)
  - 설계 의도
    - `Team`은 `Team`에 속한 `Member`들을 추가하거나 수정하고 싶을 때

  - 1의 입장인 `Team`에서 연관관계의 주인을 관리
    ```java
    @Entity
    public class Team {
      ...
      @OneToMany
      @JoinColumn(name = "TEAM_ID")
      private List<Member> members = new ArrayList<>();
    }

    ...
    // 예제
    Member member = new Member();
    member.setUsername("member1");

    em.persist(member);

    Team team = new Team();
    team.setName("teamA");
    team.getMembers().add(member);  // 연관관계 주인 활용

    em.persist(team);
    ```
  - 일반적으로 수행되는 `insert` 쿼리에 추가로 `update` 쿼리가 실행 된다.
    - team 엔티티를 저장했지만 연관 관계는 반대편(Member) 테이블을 가리키고 있으므로 반대편 테이블의 `update` 쿼리가 한번 더 실행됨
    - 의도하지 않은 쿼리가 수행되므로 **복잡도가 올라가며 및 성능상 좋지 않다.**

  - 테이블 일대다 관계는 항상 다(N) 쪽에 외래키가 있음.
  - 객체와 테이블의 차이 때문에 **반대편 테이블의 외래키를 관리**하는 특이한 구조가 된다.
  - `@JoinColumn`을 사용하지 않으면 조인 테이블 방식을 사용하므로 `@JoinColumn`을 꼭 써야한다.
    - 조인 테이블 방식 : 연관관계를 위해 두 테이블 사이에 중간 테이블을 생성하여 매핑한다 (좋지 않음)
  - 일대다 단방향 매핑보다는 **다대일 양방향 매핑을 사용**하자!

- 일대다 양방향
  - ![image](https://user-images.githubusercontent.com/36228833/194356306-82ef8c6e-6947-4bbc-affb-b16e0d4eeecb.png)
  - 설계 의도
    - `Team`은 `Team`에 속한 `Member`들을 추가하거나 수정하고 싶고
    - `Member`는 `Member`가 속한 팀을 조회하고 싶을 때
  - 공식 스펙상으로 지원하지 않으나 야매 방식(insertable, updatable 속성을 사용하여 읽기전용 필드로 만듬)으로 처리할 수 있음
    ```java
    @Entity
    public class Member {
      ...
      @ManyToOne
      @JoinColumn(name="TEAM_ID", insertable=false, updatable=false)
      private Team team;
    }
    ```
  - `@JoinColumn`의 속성으로 해당 필드를 수정하지 못하게 만듦으로 써 read-only로 만듬 (양방향 매핑 구현)
  - **복잡하게 하지말고 깔끔하게 다대일 양방향을 사용하자.**

## 일대일 (1:1)

- **주 테이블**이나 **대상 테이블** 중에 외래키 선택 가능하다.
  - 주 테이블에 외래키
  - 대상 테이블에 외래키
- 외래키에 데이터베이스 유니크 제약조건을 추가해야 함.

- 주 테이블에 외래키 - 단방향
  - ![image](https://user-images.githubusercontent.com/36228833/194869769-48974508-b8be-41d9-9907-0f2d3da22768.png)
  - 설계 의도
    - 각 `Member`(주 테이블)는 락커를 하나 소유할 수 있다.
    - 다대일(`@ManyToOne`) 단방향 매핑과 유사함
    ```java
    @Entity
    public class Locker {
      @Id @GeneratedValue
      private Lond id;

      private String name;
    }

    @Entity 
    public class Member {

      @Id @GeneratedValue
      private Long id;

      @OneToOne
      @JoinColumn(name = "LOCKER_ID")
      private Locker locker;

      private String username;
    }
    ```
  - 추후에 하나의 `Locker`를 여러명의 `Member`가 사용할 수 있도록 (다대일) 관계를 확장할 수 있다. (DB 입장)
  - `Member`에 `Locker`가 있는게 성능상으로 유리하다
    - ex) `Member`가 주 테이블이기 때문에 `Member` 조회는 필수적이다. `Member`가 가진 `Locker` 정보를 확인하기 위해 굳이 `Locker`를 대상으로 한번 더 조회하지 않아도 된다.

- 주 테이블에 외래키 - 양방향
  - ![image](https://user-images.githubusercontent.com/36228833/194869818-a3dbda19-0018-49f0-9d9a-3d848a33aaac.png)
  - 설계 의도
    - 각 `Member`는 락커를 하나 소유할 수 있고
    - `Locker` 는 자신을 소유한 `Member`를 알수있음.
  ```java
    @Entity
    public class Locker {
      @Id @GeneratedValue
      private Lond id;

      private String name;

      @OneToOne(mappedBy= "locker")
      private Member member;

    }

    @Entity 
    public class Member {

      @Id @GeneratedValue
      private Long id;

      @OneToOne
      @JoinColumn(name = "LOCKER_ID")
      private Locker locker;

      private String username;
    }
    ```
  - 다대일 양방향 매핑처럼 **외래키가 있는곳이 연관관계의 주인**
    - 예제에선 `Member`가 주인
  - 반대편은 `mappedBy`를 적용

- **주 테이블 외래키 정리**
  - 주 객체가 대상 객체의 참조를 가지는 것 처럼 주 테이블에 외래키를 두고 대상 테이블을 찾는다.
  - **객체지향 개발자**가 선호한다.
  - JPA 매핑이 편리하다.
  - 장점 : 주 테이블만 조회해도 (외래키로) 대상 테이블에 데이터가 있는지 확인 가능하다. (성능상 유리)
  - 단점 : 값이 없으면 외래키에 null을 허용한다.

- 대상 테이블에 외래키 - 단방향 (불가능)
  - ![image](https://user-images.githubusercontent.com/36228833/194869880-b9db37f7-087f-4725-a244-4f11b3263c67.png)
  - `Member` 엔티티의 `locker`가 연관관계의 주인으로 설정하고 싶으나 외래키는 `LOCKER`에 있는 상황
  - 일대다 단방향 같은 상황
  - JPA에서 지원하지 않는다. 

- 대상 테이블에 외래키 - 양방향
  - ![image](https://user-images.githubusercontent.com/36228833/194869936-a8adcd67-1ba7-4145-b7e7-f6b3c2ef8f36.png)
  - `Locker`에 있는 `member`를 연관관계 주인으로 설정하고 (`LOCKER` 테이블에 외래키가 있으므로) 양방향 연관관계를 설정
    - 반대편인 `Member`의 `locker`를 읽기 전용으로 설정 
  - 사실 주 테이블에 외래키 양방향과 매핑 방법은 동일하다.

- **대상 테이블 외래키 정리**
  - 대상 테이블에 외래키가 존재한다. (null 허용 문제 해결)
  - **전통적인 데이터베이스 개발자**들이 많이 선호한다.
  - 장점 : 주 테이블과 대상 테이블을 **일대일에서 일대다 관계로 변경할 때 테이블 구조가 유지**된다. (확장에 유리)
  - 단점 : 프록시 기능의 한계로 **지연 로딩으로 설정해도 항상 즉시 로딩된다.**
    - JPA 입장에서는 `Member` 객체를 로딩할 때 외래키 대상인 `locker`에 값이 있는지 없는지 알아야함.
    - 그러나 외래키는 반대편인 `LOCKER`가 관리하기 때문에 `MEMBER` 테이블 말고도 `LOCKER` 테이블까지 전부 조회 해봐야함.
    - 결국 **주 테이블 과 대상 테이블 둘다 확인하는 과정을 필히 거치므로** 지연 로딩이 아무 의미 없음. (그래서 항상 즉시 로딩이 됨)
      - 지연 로딩 : 실제 객체를 사용하는 시점에 조회
      - 즉시 로딩 : Join을 활용하여 연관된 객체까지 한번에 조회

## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
