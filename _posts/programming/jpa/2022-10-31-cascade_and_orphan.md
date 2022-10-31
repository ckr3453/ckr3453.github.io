---
title: "연관관계 관리 - 영속성 전이(cascade)와 고아객체(orphan)"
categories: 
    - jpa
date: 2022-10-31
last_modified_at: 2022-10-31
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "연관관계에서의 영속성 전이와 고아객체에 대해 알아보자."
---

## 영속성 전이(Cascade)란

특정 엔티티를 영속 상태로 만들 때 **연관된 엔티티도 함께 영속 상태로 만들고 싶을 때** 사용한다.

데이터베이스의 참조 무결성을 위해 FK에 CASCADE를 사용하는것과 개념이 비슷하다.

영속성 전이를 사용하지 않고 **부모 엔티티가 저장을 할 때 자식 엔티티도 함께 저장**하려면 다음과 같이 사용할 수 있다.

![image](https://user-images.githubusercontent.com/36228833/199054660-4a3e7e04-de1a-4fab-87d3-7ac46d6a2e28.png)

### 영속성 전이 옵션 사용 전
```java
@Entity
@Getter
@Setter
public class Parent {
  @Id
  @GeneratedValue
  private Long id;

  private String name;

  @OneToMany(mappedBy= "parent_id")
  private List<Child> childList = new ArrayList<>();

  public void addChild(Child child){
    childList.add(child);
    child.setParent(this); // 직접 저장
  }
}

@Entity
@Getter
@Setter
public class Child {
  @Id
  @GeneratedValue
  private Long id;

  private String name;

  @ManyToOne
  @JoinColumn(name= "parent_id") //외래키가 child에 있으므로 연관관계 주인
  private Parent parent;
}

...

Child child = new Child();
Child child2 = new Child();

Parent parent = new Parent();
parent.addChild(child);
parent.addChild(child2);

em.persist(parent);
em.persist(child);   // 영속성 컨텍스트에 강제로 저장해야함
em.persist(child2);  // 영속성 컨텍스트에 강제로 저장해야함

```

`parent.addChild(child)`를 통해 엔티티를 저장하지만 영속화를 위해 영속성 컨텍스트 저장하는 과정을 따로 작성해야 한다. 

영속성 전이 옵션을 사용하면 이러한 불필요한 과정 없이 연관 엔티티도 자동으로 영속 상태로 만들 수 있다.

### 영속성 전이 옵션 사용 후

```java
@Entity
@Getter
@Setter
public class Parent {
  @Id
  @GeneratedValue
  private Long id;

  private String name;

  @OneToMany(mappedBy= "parent_id", cascade= CascadeType.PERSIST) //영속성 전이 옵션 사용
  private List<Child> childList = new ArrayList<>();

  public void addChild(Child child){
    childList.add(child);
    child.setParent(this);
  }
}

@Entity
@Getter
@Setter
public class Child {
  @Id
  @GeneratedValue
  private Long id;

  private String name;

  @ManyToOne
  @JoinColumn(name= "parent_id") //외래키가 child에 있으므로 연관관계 주인
  private Parent parent;
}

...

Child child = new Child();
Child child2 = new Child();

Parent parent = new Parent();
parent.addChild(child);
parent.addChild(child2);

em.persist(parent);
// em.persist(child);   // 생략
// em.persist(child2);  // 생략

```

![image](https://user-images.githubusercontent.com/36228833/199055137-e5e2cbf1-fe98-454d-84be-8aee899d5843.png)

Parent 엔티티에서 childList 연관관계에 `cascade=CascadeType.PERSIST` 를 사용함으로써 **Parent를 persist 할 때 대상 childList에 저장된 객체또한 persist를 적용할 것**이라는 의미다.

한마디로 **부모 엔티티가 영속 상태가 될 때 연쇄작용으로 인해 연관된 엔티티 또한 영속화**를 진행한다.

그러나 영속성 전이는 연관관계를 매핑하는 것과 아무 관련이 없다.
(엔티티를 영속화 할 때 연관된 엔티티도 함께 영속화 하는 편리함을 제공할 뿐 그 이상 그 이하도 아니다.)


|옵션|설명|
|---|---|
|CascadeType.ALL|상위 엔티티에서 하위 엔티티로 모든 경우의 cascade를 전파 (엔티티간의 모든 라이프사이클을 맞춰야 할 때 주로 사용)|
|CascadeType.PERSIST|상위 엔티티를 영속화 할 때 연관 엔티티도 영속화(저장할 때 사용)|
|CascadeType.REMOVE|상위 엔티티를 제거할 때 연관된 엔티티도 모두 제거|
|CascadeType.MERGE|상위 엔티티 상태를 병합할 때 연관된 엔티티도 모두 병합|
|CascadeType.REFRESH|상위 엔티티를 새로고침할 때 연관된 엔티티도 모두 새로고침|
|CascadeType.DETACH|상위 엔티티가 detach()를 수행하면 연관된 엔티티도 detach()가 되어 변경사항 반영이 안됨|


### 주의사항
- 영속성 전이는 하나의 부모 엔티티가 자식 엔티티들을 관리할 때 유용하다. (1:N)
  - ex) 하나의 게시물에 있는 여러 첨부파일들을 관리 할 때, 하나의 게시물에 달린 여러 댓글들을 관리할 때 등등<br/><br/>

- 그러나 해당 엔티티가 다른 엔티티들에 의해 연관되어 있으면 사용하면 안된다.
  - 즉, **자식 엔티티가 단일 엔티티에 대하여 완전히 종속적일때 사용**해야 한다.<br/><br/>

- 영속성 전이는 연관관계를 나타내는 어노테이션에 옵션으로 적용할 수 있다.
  - `@ManyToOne(cascade=CascadeType.ALL)`
    - 자식이 삭제되면 부모도 삭제된다.
    - (부모가 삭제될 때는 아무 동작하지 않음 = 부모를 잃은 자식 엔티티 발생 -> 고아객체)
  - `@OneToMany(cascade=CascadeType.ALL)`
    - 부모가 삭제되면 자식도 삭제된다.
    - 부모를 추가할때 자식도 추가한다.
  - `@ManyToMany`의 관계에서는 엔티티간의 라이프 사이클이 꼬일 확률이 높으므로 사용을 지양해야한다.
  - `@OneToOne`의 관계에서는 상호 관계를 생각하여 누가 부모-자식의 개념인지 잘 생각하고 사용해야 한다.

## 고아 객체(orphan)

고아 객체란 **부모 엔티티와 연관관계가 끊어진 자식 엔티티**를 의미한다. 

JPA에서는 이를 위해 고아 객체를 자동으로 삭제해주는 기능을 제공한다. (`orphanRemoval=true`)

```java
@Entity
@Getter
@Setter
public class Parent {
  @Id
  @GeneratedValue
  private Long id;

  private String name;

  @OneToMany(mappedBy= "parent_id", orphanRemoval=true) // 고아객체 자동제거
  private List<Child> childList = new ArrayList<>();

  public void addChild(Child child){
    childList.add(child);
    child.setParent(this);
  }
}

@Entity
@Getter
@Setter
public class Child {
  @Id
  @GeneratedValue
  private Long id;

  private String name;

  @ManyToOne
  @JoinColumn(name= "parent_id")
  private Parent parent;
}

...

Child child = new Child();
Child child2 = new Child();

Parent parent = new Parent();
parent.addChild(child);
parent.addChild(child2);

em.persist(parent);

em.flush();
em.clear(); // 영속성 컨텍스트 초기화

Parent parent = em.find(Parent.class, parent.getId());
parent.getChildList().remove(0); // 자식 엔티티를 컬렉션에서 제거 (연관관계가 끊어져서 고아 객체로 인지, Child delete 쿼리 수행)
```

### 주의사항
- orphanRemoval은 참조가 제거된 엔티티를 다른곳에서 참조하지 않는 고아 객체로 판단하고 삭제한다<br/><br/>

- 참조하는 곳이 하나일 때만 사용해야 한다.
  - 즉, 특정 엔티티가 개인 소유할 때 사용
  - 다른 엔티티가 연관되어 있으면 안됨<br/><br/>

- 개념적으로 부모를 제거하면 자식은 고아가 된다.
  - 따라서 고아 객체 제거 기능을 활성화 하면 부모를 제거할 때 자식도 함께 제거된다. (마치 CascadeType.REMOVE 처럼)
  - 그러면 왜 CascadeType.REMOVE를 안쓰고 orphanRemoval 옵션을 사용하는 것일까?

### CascadeType.REMOVE VS orphanRemoval=true
`CascadeType.REMOVE`와 `orphanRemoval=true`는 개념적으로는 같아보이지만 다르게 동작한다.

이는 부모 엔티티를 통해서 연관 엔티티를 제거할 때 차이점이 명확히 드러난다.

- **부모 엔티티를 삭제하는 경우**<br/><br/>
  ```java
  ...
  Parent parent = em.find(Parent.class, parent.getId());
  parentRepository.save(parent);
  parentRepository.delete(parent);
  ```
  - `CascadeType.REMOVE` && `orphanRemoval=true`
    - 두가지 옵션 모두 **부모 엔티티가 삭제될 때 연관 엔티티 child가 삭제된다.**<br/><br/>

- **부모 엔티티에서 자식 엔티티를 제거하는 경우**<br/><br/>
  ```java
  ...
  Parent parent = em.find(Parent.class, parent.getId());
  parentRepository.save(parent);
  parent.getChildList().remove(0);
  ```
  - `CascadeType.REMOVE`
    - 연관 엔티티 child를 delete 하는 쿼리를 수행하지 않는다.
    - `parent.getChildList().remove(0)`를 함으로써 parent와 child 엔티티간의 연관관계가 끊어졌지만 **이를 제거됐다고 보지 않기 때문이다.**
  - `orphanRemoval=true`
    - 관계가 끊어진 연관 엔티티 child를 고아 객체로 인식하고 delete하는 쿼리를 수행한다.<br/><br/>

- 정리
  - `CascadeType.REMOVE` 옵션만으로는 부모 엔티티를 통해 자식 엔티티의 생명주기를 제대로 관리할 수 없다. (고아객체 인지x)
  - `CascadeType.ALL` + `orphanRemoval=true` 옵션을 같이 사용한다면 부모 엔티티를 통해 자식의 생명주기를 관리할 수 있게된다.


## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
