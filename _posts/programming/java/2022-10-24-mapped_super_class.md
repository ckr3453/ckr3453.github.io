---
title: "JPA) 고급 연관관계 매핑 - @MappedSuperclass"
categories: 
    - java
date: 2022-10-24
last_modified_at: 2022-10-24
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "@MappedSuperclass에 대해 알아보자."
---

## 엔티티 별 중복되는 멤버 상속 받기
<center><img src="https://user-images.githubusercontent.com/36228833/197555995-9998982b-8a5c-43e4-84e6-d557f585804a.png"></center>

- 객체의 입장에서 중복되는 필수 멤버들이 계속 나옴 (ex. id, name, created_date 등)
- 마치 클래스를 상속 받아서 사용하듯이 공통된 속성을 상속받아서 쓸수는 없을까?
- 즉 공통 매핑 정보를 상속받아 사용할 수 있다면 유용하게 활용할 수 있다.
- 예를 들어, 누군가 등록 혹은 수정한 시간에 대한 정보가 테이블마다 필요하다.<br/>

  ```java
  @Entity
  public class Member {
    ...

    private String createdBy; // 생성한 사람

    private LocalDateTime createdDate; // 생성 날짜

    private String lastModifiedBy; // 마지막으로 수정한 사람

    private LocalDateTime lastModifiedDate; // 마지막으로 수정한 날짜

    ...
  }

  @Entity
  public class Team {
    ...

    private String createdBy; // 생성한 사람

    private LocalDateTime createdDate; // 생성 날짜
    
    private String lastModifiedBy; // 마지막으로 수정한 사람

    private LocalDateTime lastModifiedDate; // 마지막으로 수정한 날짜

    ...
  }
  ```

위 처럼 각 엔티티 마다 해당하는 멤버를 전부 넣어줘야 한다.
- 이렇게 중복되는 멤버들을 매번 써줘야할까?

엔티티 작성 시 중복되는 멤버를 상속받을 클래스에 적어주고 `@MappedSuperclass`를 작성하여 상속받으면 된다.<br/>

  ```java
  @MappedSuperclass
  public abstract class BaseEntity {
    
    @Column(name="INSERT_MEMBER")
    private String createdBy; // 생성한 사람

    private LocalDateTime createdDate; // 생성 날짜
    
    @Column(name="UPDATE_MEMBER")
    private String lastModifiedBy; // 마지막으로 수정한 사람

    private LocalDateTime lastModifiedDate; // 마지막으로 수정한 날짜

  }

  @Entity
  public class Member extends BaseEntity {
    ...

  }

  @Entity
  public class Team extends BaseEntity {
    ...

  }
  ```

이렇게 작성하면 Member와 Team 엔티티에서 BaseEntity 의 멤버를 상속받아 사용할 수 있다.

## 정리
- `@MappedSuperclass`는 상속관계 매핑이 아니다.

- `@MappedSuperclass`가 선언된 클래스는 엔티티가 아니다.
  - 즉 테이블과 매핑되지 않는다.
  - 엔티티가 아니므로 당연히 `EntityManager.find(BaseEntity)` 같이 조회, 검색이 불가능하다.

- `@MappedSuperclass`가 선언된 클래스(부모 클래스)를 상속 받는 자식 클래스에 매핑 정보만 제공한다.

- 직접 생성해서 사용할 일이 없으므로 혹시나 누군가가 사용할 수 있는 상황을 배제하기 위해 **추상 클래스로 작성할 것을 권장**한다.

- 주로 등록일, 수정일 등과 같이 **전체 엔티티에서 공통으로 적용하는 정보를 모을 때 사용**한다.

- 엔티티는 같은 `@Entity`나 `@MappedSuperclass`로 지정한 클래스만 상속이 가능하다.

## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
