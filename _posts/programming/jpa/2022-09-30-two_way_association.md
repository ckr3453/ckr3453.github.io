---
title: "연관관계 매핑 - 양방향 연관관계와 연관관계의 주인"
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
excerpt: "양방향 연관관계와 연관관계의 주인"
---

단방향 연관관계에 이어 양방향 연관관계를 알아보자.

## 시나리오
단방향 연관관계에서 사용한 시나리오를 다시 보자.

- 회원과 팀이 있다.
- 회원은 하나의 팀에만 소속될 수 있다.
- 회원과 팀은 다대일(N:1) 관계다
  - 하나의 팀에 여러명의 회원이 소속될 수 있다.

## 단방향 연관관계의 한계

<center><img src="https://user-images.githubusercontent.com/36228833/193238678-a29802d1-75a2-4559-a8db-22a67ea42e0d.png"></center>

```java
//조회
Member findMember = em.find(Member.class, member.getId());

//참조를 사용해서 연관관계 조회
Team findTeam = findMember.getTeam();

//그렇다면 팀에 속해있는 멤버들을 호출하는 것은? (불가능)
// List<Member> members = team.getMembers(); 
```

이와 같은 관계일 때 `Team` 객체 입장에서는 `Member` 객체를 참조하고 있지않기 때문에 **팀에 속한 멤버들을 호출하는 것이 불가능**하다. 

그렇다면 팀의 입장에서 해당 팀에 속한 멤버들을 호출하려면 어떻게 해야할까?

`Team` 객체도 `Member` 객체를 참조하여 **서로가 서로를 참조하는 양방향 연관관계로 설계**하면 된다!

## 단방향에서 양방향으로 연관관계 설정하기
기존 연관관계를 양방향 연관관계로 수정하면 다음과 같이 표현할 수 있다.

<center><img src="https://user-images.githubusercontent.com/36228833/193282200-75ac8ab4-2268-4840-95c9-d5ad08f75305.png"></center>

- 단방향에서 양방향으로 바꿨음에도 **기존 테이블 연관관계는 변하지 않는다** Why?
  - 두 테이블 간의 **외래키가 존재한다면 양방향이 성립**되기 때문이다.
    - `Member` 테이블 기준
      ```sql 
      SELECT M.* FROM MEMBER AS M INNER JOIN TEAM AS T ON M.TEAM_ID = T.TEAM_ID 
      ```
    - `Team` 테이블 기준
      ```sql
      SELECT T.* FROM TEAM AS T INNER JOIN MEMBER AS M ON T.TEAM_ID = M.TEAM_ID
      ```
    - 어느 테이블을 기준으로 하든, 외래키로 조회가 가능하다!
    - 사실상 테이블 간의 관계에 있어서 방향이라는 개념 자체는 없다.

- 문제는 객체다.
  - 기존 단방향 연관관계의 경우 `Team` 객체 입장에서는 `Member` 객체를 참조하고 있지 않았기 때문에 불러올 수 없다.
  - 그래서 그림과 같이 `Team` 객체가 `Member` 객체를 참조할 수 있도록 하여 양방향 연관관계로 만들어 줘야한다.

기존 `Team` 객체를 다음과 같이 수정하여 양방향 연관관계를 설정한다.

```java
@Entity
public class Team {
  @Id @GeneratedValue
  private Long id;

  private String name;

  // 양방향 연관관계를 위해 추가됨
  // 연관관계의 주인을 나타내기 위해 mappedBy를 사용하여 명시
  @OneToMany(mappedBy="team")
  private List<Member> members = new ArrayList<>();

  //getter and setter 
}
```

이로써 팀을 통해 팀에 속한 멤버들을 호출할 수 있게끔 객체 그래프를 탐색할 수 있게된다.

```java
// 5번 멤버가 속한 팀의 모든 멤버들을 조회하기
Member findMember = em.find(Member.class, 5L);
List<Member> members = findMember.getTeam().getMembers();

// 3번 팀에 속한 모든 멤버들을 조회하기
Team findTeam = em.find(Team.class, 3L);
List<Member> members = findTeam.getMembers();
```

## 연관관계의 주인 (mappedBy 활용)
양방향 연관관계의 경우, 연관관계의 주인을 설정하기 위해 `@OneToMany`는 `mappedBy`라는 속성을 지원한다.

연관관계의 주인을 설정하기 위해 객체와 테이블이 관계를 맺는 차이에 대해 이해해야 한다.

위의 시나리오를 예를 들면 객체와 테이블의 연관관계는 다음과 같다.

- 객체의 양방향 연관관계 (총 2개)
  - 객체로 양방향 관계로 설정하려면 **객체간의 단방향 연관관계를 2개** 만들어야 한다.
  - 회원 -> 팀 연관관계 1개 (단방향)
    - `Member.getTeam()`
  - 팀 -> 회원 연관관계 1개 (단방향)
    - `Team.getMembers()`

- 테이블의 양방향 연관관계 (총 1개)
  - 테이블은 **외래키 하나로 두 테이블의 연관관계를 설정**할 수 있다. (외래키를 기준으로 서로 join 할 수 있으므로)
  - 회원 <-> 팀의 연관관계 1개 (방향이 없으나 외래키 하나로 서로 join이 가능하므로 양방향)
    - 외래키인 `TEAM_ID`를 활용하여 join 수행 가능

### 둘중 하나로 외래키를 관리해야 한다.
<center><img src="https://user-images.githubusercontent.com/36228833/193282446-4bea20b3-8dbc-49eb-a40b-6ea454fbc957.png"></center>

- 객체 연관관계가 양방향이 되면서 고민에 빠진다.
  - 팀의 멤버를 바꾸고 싶거나 / 멤버가 새로운 팀으로 들어가고 싶을 때
    - Member 객체의 team 값을 수정했을 때 외래키(TEAM_ID) 값을 수정해야할지
    - Team 객체의 members 값을 수정했을 때 외래키(TEAM_ID) 값을 수정해야할지
  - 외래키 값의 업데이트를 `Member` 객체에서 관리할 지 `Team` 객체에서 관리할 지 정해야 한다!
  - 이같은 상황을 해결하기 위해 **연관관계의 주인** 개념이 등장했다.

### 연관관계의 주인을 설정할 때 규칙
연관관계의 주인 개념은 객체간의 양방향 매핑이 성립될 때만 적용된다. 적용할 때의 규칙은 다음과 같다.
- 객체의 두 관계중 하나를 연관관계의 주인으로 지정한다.
- **연관관계의 주인만이 외래키를 관리(등록, 수정)한다.** (INSERT, UPDATE)
- **연관관계의 주인이 아닌쪽은 읽기만 가능하다.** (SELECT)
- **연관관계의 주인이 아닌쪽이 mappedBy 속성으로 주인을 지정**하며 주인은 mappedBy 속성을 사용하지 않는다. - mappedBy : ?에 의해서 매핑이 되었다는 의미

### 그렇다면 누구를 주인으로 설정해야할까?

- **외래키가 있는곳을 주인으로** 설정하라.
- 여기서는 `MEMBER` 테이블에 외래키(TEAM_ID)가 있으므로 `Member.team`이 연관관계의 주인이다!

<center><img src="https://user-images.githubusercontent.com/36228833/193282539-516f363d-ff5c-4f28-847f-10f539bcb9d1.png"></center>

- 외래키가 있는 `Member.team`이 연관관계의 주인이 되어야하는 이유
  - 물론 복잡한 설정을 통해 `Team.members`가 연관관계의 주인이 될 수도 있다.
  - 그러나 그렇게 되면 `Team.members`의 값을 바꿨을 때 `TEAM` 테이블이 아닌 외래키를 가지고 있는 `MEMBER` 테이블의 값을 바꿔야 한다.
  - 이런식으로 **엔티티와 다른 테이블을 대상으로 쿼리가 나가게 되면 복잡도가 올라가고 혼란을 야기할 수 있음.**
  - 또한 데이터베이스 입장에서 봤을 때 외래키를 가진 테이블 쪽이 N의 관계를 가짐 (반대편은 1, 즉 N:1 관계)
    - 즉, **`@ManyToOne`이 연관관계의 주인**이 된다.

## 주의할 점

양방향 매핑 시 가장 많이 실수하는 부분을 알아보자.

### 연관관계의 주인에 값을 입력하지 않음

```java
Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setName("member1");

// 역방향(주인이 아닌방향)만 연관관계 설정
team.getMembers().add(member);

em.persist(member);
```

`team.getMembers()`로 호출하는 `members`의 연관관계의 주인은 `team`이다.

그러나 연관관계의 주인에 값을 입력하지 않고 주인이 아닌 반대방향에만 값을 입력하고 있다. (`team.getMembers().add(member);`)

이렇게 될 경우 실제 테이블에는 해당 멤버의 `team` 값이 들어가지 않게된다. 그래서 다음과 같이 수정해야 한다.

```java
Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setName("member1");

// 연관관계의 주인에 값을 입력
member.setTeam(team);

// 역방향(주인이 아닌방향)만 연관관계 설정
team.getMembers().add(member);

em.persist(member);
```

그렇다고 해서 연관관계의 주인에만 값을 입력하라는 의미가 아니다.

양방향 연관관계에 **서로 값을 입력해줘야 테이블의 값 입력은 물론, 순수한 객체 관계가 유지**될 수 있다.

아래와 같이 주인에만 값을 저장할 경우 `EntityManager`를 통해 주인객체의 역방향을 호출할 때 제대로 호출되지 않는다. (1차 캐시에 이미 올라가 있는 객체를 불러오게 됨)

```java
Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setName("member1");

// 연관관계의 주인에 값을 입력
member.setTeam(team);

// 역방향에는 값을 입력하지 않음
// team.getMembers().add(member);

em.persist(member);

Team findTeam = em.find(Team.class, team.getId());  // 실제 저장된 값이 아닌 1차 캐시에 저장된 team 객체를 불러옴 (members에 값이 없는 상태의 객체)
List<Member> members = findTeam.getMembers();       // members에 값을 입력한 적이 없으니 조회할 수 없음. (그러나 실제 DB에는 저장된 상태)
```

- **순수 객체 상태를 고려해서 항상 양쪽에 값을 설정하자!**
- 연관관계 편의 메소드(양쪽에 값을 저장하는 메소드)를 만들어서 활용하자.
  - ex) `member.setTeam(team)` 호출 시 `team.getMembers().add(member)`를 추가
    ```java
    @Entity
    public class Member {
      ...
      public void setTeam(Team team){
          this.team = team;
          team.getMembers().add(this);
      }
    }
    ```
  - 그러나 연관관계 메소드가 **양쪽에 전부 있으면 무한루프에 빠질 수 있으니 조심**하자.

### toString(), Lombok, JSON 생성 라이브러리 사용 시 주의

양방향 매핑 시 `toString()`등을 통해 객체(엔티티)를 json으로 변환하여 서로 호출할 경우 무한루프에 빠질 수 있다.

```java
@Entity
public class Member {
  ...
  @Override
  public String toString(){
      return "Member {" +
              "id=" + id +
              ", team=" + team +  // Team 객체를 호출
              '}';
  }
}

@Entity
public class Team {
  ...
  @Override
  public String toString(){
      return "Team {" +
              "id=" + id +
              ", members=" + members + // Member 객체를 호출
              '}';
  }
}

Team findTeam = em.find(Team.class, team.getId());
System.out.println("team=" + findTeam); // 무한루프 발생!
```

- 왠만해선 `Lombok` 라이브러리를 통해 `toString()` 사용을 지양하자.
- (JSON 생성 라이브러리를 통한) Controller에서 `Entity` 반환을 지양하고 DTO로 반환하자.
  - `toString()`가 있을 경우 무한루프를 야기할 수 있음.
  - API Spec에 변화가 생기는 문제 발생

## 정리
- 사실 단방향 매핑만으로도 이미 연관관계 매핑은 완료가 된 것이다.
- 양방향 매핑은 반대 방향으로 조회(객체 그래프 탐색) 기능이 추가된 것 뿐이다.
- 단방향 매핑을 잘하고 양방향은 필요할 때 추가해도 된다.
  - 객체관계를 위한 것이므로 테이블에 영향을 주지 않기 때문이다.
- 비즈니스 로직을 기준으로 연관관계의 주인을 선택하면 안된다.
  - **외래키의 위치를 기준으로** 정해야 한다!
  
## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
