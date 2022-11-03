---
title: "값 타입 (작성중)"
categories: 
    - jpa
date: 2022-11-02
last_modified_at: 2022-11-02
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "JPA에서의 값 타입을 알아보자."
---

## JPA의 데이터 타입 분류

JPA는 데이터 타입을 최상위 레벨로 봤을 때 엔티티 타입과 값 타입으로 분류한다.

- **엔티티 타입**
  - `@Entity`로 정의하는 클래스 객체
  - 데이터가 변해도 식별자로 지속해서 추적이 가능하다.
    - 예) 회원 엔티티의 키나 나이 값을 변경해도 식별자로 인식이 가능하다.
    - `@Id`가 100번인 경우 엔티티 내부에 다른 속성이 바뀌더라도 `@Id` 100번로 인식 및 추적이 가능하다.

- **값 타입**
  - int, Integer, String처럼 단순히 값으로 사용하는 자바 기본 타입이나 객체를 의미한다.
  - 식별자가 없고 값만 있으므로 변경시 추적이 불가능하다.
    - 예) 정수 100을 200으로 변경하면 완전히 다른값으로 대체된다.
    - 바뀐 200이 과거에 어떤 값이었는지 추적할 수 없다.

## 값 타입

값 타입은 크게 기본값 타입, 임베디드 타입, 컬렉션 값 타입으로 나눌 수 있다.

### 기본값 타입의 특징
- 자바 primitive 타입 (int, double 등)
- wrapper 클래스 (Integer, Long)
- String
- 자바가 제공하며 기본적으로 값을 세팅하여 사용할 수 있는 타입들
- 엔티티에 의존하는 생명주기를 가지고 있다.
  - 예) 회원 엔티티를 삭제하면 이름, 나이, 이메일 등의 필드도 함께 삭제된다.
- 기본값 타입은 공유하면 안된다.
  - 예) 회원의 이름을 변경 시 다른 회원의 이름도 함께 변경되면 안된다.
  - 의도치 않은 side effect가 일어날 수 있다.<br/><br/>
    ```java
    int a = 10; // a에 10 할당
    int b = a;  // b에 10 할당

    a = 20; // a에 20으로 재할당
      
    // primitive 타입은 저장공간을 따로 가지고 있기 때문에 값이 공유되지 않는다.
    System.out.println(a); // 20
    System.out.println(b); // 10
    ```
  - Integer 같은 wrapper 클래스나 String 같은 특수한 클래스는 참조가 가능하기 때문에 공유가 가능한 객체지만 값을 변경할 수는 없다<br/><br/>
    ```java
    Integer a = new Integer(10); // a에 10을 할당
    Integer b = a; // b에 a를 참조

    // 값 변경을 위한 메서드를 제공해 주지 않음. (a.setValue() 같은 메서드는 없음)

    // b의 경우 a의 인스턴스를 공유하지만 wrapper 클래스나 String의 경우 side effect가 일어나지 않게 값 변경을 지원하지 않는다.
    System.out.println(a); // 10
    System.out.println(b); // 10
    ```

### 임베디드 타입(embedded type, 복합 값 타입)의 특징
- 새로운 값 타입을 직접 정의할 수 있음
- JPA는 임베디드 타입(embedded type)이라 함.
- 주로 기본값 타입을 모아서 만들어서 복합 값 타입이라고도 함.
- int, String과 같은 값 타입
  - Entity가 아니다.
  - 당연히 Entity가 아니다보니 추적이 안된다.

#### 임베디드 타입 활용 기본 예시

<center><img src="https://user-images.githubusercontent.com/36228833/199756818-d51db840-bd28-4947-913f-81995d416983.png"></center><br/>

회원 엔티티는 이름, 근무 시작일, 근무 종료일, 주소 도시, 주소 번지, 주소 우편번호를 가진다.

회원 엔티티가 가지는 속성중에 공통되는 부분을 추상화 하여 표현한다면 다음과 같이 표현 가능하다.

<center><img src="https://user-images.githubusercontent.com/36228833/199757076-d02c312c-1c2e-4ad2-a642-80f51e4e9a88.png"></center><br/>

회원 엔티티는 이름, 근무기간, 집 주소를 가진다.

<center><img src="https://user-images.githubusercontent.com/36228833/199757189-1a73737c-f442-41f4-a5e4-f639efaffe5b.png"></center><br/>

- 근무 시작일, 근무 종료일 -> 근무 기간(workPeriod)
- 주소 도시, 주소 번지, 주소 우편번호 -> 집 주소(address)

이처럼 **유사하거나 공통되는 속성들을 모아서 하나의 타입으로 사용하는 것을 임베디드 타입**이라고 한다.

임베디드 타입을 사용하면 다음과 같은 장점을 얻을 수 있다.

- 재사용이 가능하다.
  - 위에서 사용한 근무 기간, 집 주소의 경우 회원 엔티티가 아닌 다른 엔티티에서도 활용이 가능하다.
- 높은 응집도를 가진다.
- `Period.isWork()`처럼 해당 값 타입만 사용하는 의미있는 메서드를 만들 수 있다.
- 임베디드 타입을 포함한 모든 값 타입은 **값 타입을 소유한 엔티티에 생명주기를 의존**한다.

임베디드 타입의 사용법은 다음과 같다.

- `@Embeddable`
  - 값 타입을 정의하는 곳에 표시
- `@Embedded`
  - 값 타입을 사용하는 곳에 표시

#### 임베디드 타입과 테이블 매핑

<center><img src="https://user-images.githubusercontent.com/36228833/199757327-405caeec-2617-4365-9a25-696cf142458f.png"></center><br/>

데이터베이스 입장에서는 임베디드 타입을 쓰든 안쓰든 어차피 값을 포함하고 있으니 바뀔 것이 없다.

대신 다음과 같이 어노테이션을 선언하여 매핑하는 과정이 필요하다.

```java
@Embeddable // 임베디드 값 타입을 정의
@Getter
@AllArgsConstructor
public class Period {
   private LocalDateTime startDate;
   private LocalDateTime endDate;
}

@Embeddable // 임베디드 값 타입을 정의
@Getter
@AllArgsConstructor
public class Address {
   private String city;
   private String street;

   @Column(name="ZIP_CODE") // 이런것도 가능하다.
   private String zipcode;
}

@Entity
public class Member {
   @Id
   @GeneratedValue
   private Long id;

   private String name;
   
   @Embedded // 임베디드 값 타입을 사용
   private Period workPeriod;

   @Embedded // 임베디드 값 타입을 사용
   private Address homeAddress;
}

....

Member member = new Member();
member.setName("test");
member.setHomeAddress(new Address("city", "street", "zipcode"));
member.setWorkPeriod(new Period(LocalDateTime.now(), LocalDateTime.now()));

entityManager.persist(member);
```

- 임베디드 타입은 엔티티의 값일 뿐이다.
  - 크게 그 이상의 의미를 가지면 안된다.
  - 임베디드 타입을 사용하기 전과 후에 **매핑하는 테이블은 같다.**
- 객체와 테이블을 아주 세밀하게(find-grained) 매핑하는 것이 가능하다.
  - 여러 속성을 장황하게 쓰지 않아도 되고 유사한 성격을 가진 속성을 묶어서 클래스로 사용함으로써 해당 속성과 관련된 메서드를 통한 활용도가 높다.
- 잘 설계한 ORM 애플리케이션은 **매핑한 테이블의 수보다 클래스의 수가 더 많다.**
- 용어 및 코드가 간결해지고 공통화 되면서 개발이 수월해진다.

#### 임베디드 타입과 연관관계

![image](https://user-images.githubusercontent.com/36228833/199757437-2b96b9c7-5e9e-4228-8aa8-3c61c8a27905.png)

- Member 엔티티는 임베디드 타입으로 Address와 PhoneNumber를 가질 수 있다.
  - Address는 속성으로 임베디드 타입인 Zipcode를 가질 수 있다.
  - PhoneNumber는 속성으로 엔티티 타입인 PhoneEntity를 가질 수 있다.
    - PhoneNumber 입장에서는 PhoneEntity에 대한 참조만 들고있으면 되기 때문에 가능하다.

만약에 다음과 같이 한 엔티티 안에서 같은 값 타입을 사용하려면?

```java
@Entity
public class Member {
  ...

  @Embedded
  private Address homeAddress; // 컬럼이 중복되어서 오류 발생

  @Embedded
  private Address workAddress; // 컬럼이 중복되어서 오류 발생
}
```

임베디드 타입이 중복되어서 오류가 발생한다. 그럴때는 `@AttributeOverride`를 사용하여 속성명을 재정의 하면 해결된다.

```java
@Entity
public class Member {
  ...

  @Embedded
  private Address homeAddress;

  @Embedded
  @AttributeOverrides({
      @AttributeOverride(name="city", column=@Column("WORK_CITY")),       //Address의 city를 WORK_CITY로 재정의
      @AttributeOverride(name="street", column=@Column("WORK_STREET")),   //Address의 street을 WORK_STREET로 재정의
      @AttributeOverride(name="zipcode", column=@Column("WORK_ZIPCODE")), //Address의 zipcode를 WORK_ZIPCODE로 재정의
  })
  private Address workAddress;
}
```


### 컬렉션 값 타입(collection value type)
  - 자바가 제공하는 컬렉션(List, Set 등)에 기본값 타입 혹은 임베디드 타입을 넣어서 사용할 수 있다.
  - 예) `List<Position> positions`, `Set<Integer> numbers` 등

## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
