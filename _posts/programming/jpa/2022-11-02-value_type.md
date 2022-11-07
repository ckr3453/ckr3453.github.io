---
title: "값 타입 (작성중)"
categories: 
    - jpa
date: 2022-11-02
last_modified_at: 2022-11-05
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

값 타입은 복잡한 객체 세상을 조금이라도 단순화 하고 만든 개념이다. 따라서 값 타입은 단순하고 안전하게 다룰 수 있어야 한다.

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
@Setter
@AllArgsConstructor
public class Period {
   private LocalDateTime startDate;
   private LocalDateTime endDate;
}

@Embeddable // 임베디드 값 타입을 정의
@Getter
@Setter
@AllArgsConstructor
public class Address {
   private String city;
   private String street;

   @Column(name="ZIP_CODE") // 이런것도 가능하다.
   private String zipcode;
}

@Entity
@Getter
@Setter
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
  @Id
  @GeneratedValue
  private Long id;
  
  private String name;

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

값 타입을 컬렉션에 담아서 사용하는 것을 의미한다. 예시를 통해 알아보자.

![image](https://user-images.githubusercontent.com/36228833/200177014-f1a0cc38-4c6c-4b3c-acc7-d6631be7eec9.png)

Member라는 엔티티는 id와 선호하는 음식들인 favoriteFoods, 주소 기록인 addressHistory로 구성된다.

단순하게 값 타입이 단일인 경우에는 클래스 필드로 작성하고 사용하면 됐지만 컬렉션인 경우 얘기가 다르다.

관계형 데이터베이스는 기본적으로 컬렉션을 내부에 담을수 없는 구조이기 때문이다.

그래서 **컬렉션을 1:N 구조(컬렉션 : 내부 아이템들)로 하여 다음과 같이 별도의 테이블로 추출**하여 뽑아야 한다.

![image](https://user-images.githubusercontent.com/36228833/200177032-2cccf89f-311b-4cd6-adf5-f84b59abbd31.png)

테이블 구성 방식은 **원래 테이블의 PK + 나머지 필드로 PK를 만든 뒤 테이블로 추출**한다. 그 후 테이블 매핑 및 식별을 위해 원래 테이블의 PK를 FK로 잡는다.

왜냐하면 기본적으로 컬렉션에 들어오는 값들은 값 타입이기 때문에 FAVORITE_FOOD_ID, ADDRESS_ID 와 같이 개별 식별자 PK를 가지게 되면 값 타입이 아닌 엔티티로 봐야하기 때문이다.

값 타입 컬렉션은 `@ElementCollection`로 선언하고 `@CollectionTable`로 관계형 데이터베이스 내에서 매핑할 정보를 입력하여 사용한다.

#### 저장 예제

```java
@Embeddable
@Getter
@AllArgsConstructor
public class Address {
   private String city;
   private String street;
   private String zipcode;
}

@Entity
@Getter
@Setter
public class Member {
  @Id
  @GeneratedValue
  private Long id;

  private String name;

  @ElementCollection  // 값 타입 컬렉션인 경우 선언
  @CollectionTable(name = "FAVORITE_FOOD", joinColumns = @JoinColumn(name="MEMBER_ID")) // 컬렉션 테이블명을 선언 및 FK 설정
  @Column(name = "FOOD_NAME") // 컬렉션 내 값이 단일이며 기본값일 경우 값에 대한 컬럼명 정의가 가능하다.
  private Set<String> favoriteFoods = new HashSet<>();

  @ElementCollection  // 값 타입 컬렉션인 경우 선언
  @CollectionTable(name = "ADDRESS", joinColumns = @JoinColumn(name="MEMBER_ID")) // 컬렉션 테이블명을 선언 및 FK 설정
  private List<Address> addressHistory = new ArrayList<>();
}

...

Member member = new Member();
member.setName("member1");

member.getFavoriteFoods().add("치킨");
member.getFavoriteFoods().add("피자");
member.getFavoriteFoods().add("족발");

member.getAddressHistory().add(new Address("city2", "street2", "2412"));
member.getAddressHistory().add(new Address("city3", "street3", "2412"));

em.persist(member);

```

위 예제를 실행하면 다음과 같이 테이블이 만들어진다.

- **MEMBER**

  |MEMBER_ID|NAME|
  |---|---|---|---|---|
  |1|member1|

- **FAVORITE_FOOD**

  |MEMBER_ID|FOOD_NAME|
  |---|---|
  |1|족발|
  |1|피자|
  |1|치킨|

- **ADDRESS**

  |MEMBER_ID|CITY|STREET|ZIPCODE|
  |---|---|---|---|
  |1|city2|street2|2412|
  |1|city3|street3|2412|


여기서 알수있는 흥미로운 사실은 **값 타입 컬렉션에 대한 persist를 따로 선언하지 않았음에도 영속화가 되어있다**는 점이다.

그 이유는 값 타입 컬렉션 또한 값 타입 이며 때문에 **MEMBER 엔티티의 하나의 필드로 인식하기 때문이다. (생명주기또한 엔티티에 의존한다.)**

덕분에 컬렉션을 수정할 때 따로 persist를 할 필요가 없으며 컬렉션 객체를 다루듯 사용하면 자동으로 update 되어 반영된다.
(마치 값 타입 컬렉션에 영속성 전이 + 고아객체 제거 기능을 활성화한 것과 같다.)

#### 조회, 수정 예제

```java
@Embeddable
@Getter
@AllArgsConstructor
public class Address {
   private String city;
   private String street;
   private String zipcode;
}

@Entity
@Getter
@Setter
@EqualsAndHashCode
public class Member {
  @Id
  @GeneratedValue
  private Long id;

  private String name;

  @ElementCollection
  @CollectionTable(name = "FAVORITE_FOOD", joinColumns = @JoinColumn(name="MEMBER_ID"))
  @Column(name = "FOOD_NAME")
  private Set<String> favoriteFoods = new HashSet<>();

  @ElementCollection
  @CollectionTable(name = "ADDRESS", joinColumns = @JoinColumn(name="MEMBER_ID"))
  private List<Address> addressHistory = new ArrayList<>();
}

...

Member member = new Member();
member.setName("member1");

member.getFavoriteFoods().add("치킨");
member.getFavoriteFoods().add("피자");
member.getFavoriteFoods().add("족발");

member.getAddressHistory().add(new Address("old1", "street", "10000"));
member.getAddressHistory().add(new Address("old2", "street3", "2412"));

em.persist(member);

em.flush();
em.clear();

// 지연 로딩으로 인해 member만 조회
Member findMember = em.find(Member.class, member.getId());  
List<Address> addressHistory = findMember.getAddressHistory();

for(Address address : addressHistory){
  // 지연 로딩으로 인해 이 시점에서 address 조회 쿼리 수행
  System.out.println("address = " + address.getCity()); 
}

// 1. 치킨을 스시로 수정하기
findMember.getFavoriteFoods().remove("치킨");
findMember.getFavoriteFoods().add("스시");

// 2. city가 old1인 아이템을 new로 수정하기
findMember.getAddressHistory().remove(new Address("old1", "street", "10000"));
findMember.getAddressHistory().add(new Address("new", "street", "10000"));
```

조회의 경우 기본적으로 값 타입 컬렉션은 지연로딩 전략을 사용한다.

그렇기 때문에 조회 시 직접 아이템에 접근하는 시점에 쿼리를 수행한다.

첫번째 수정의 경우 컬렉션에 해당하는 타입이 String 이기 때문에 update가 불가능하다. 직접 제거한 뒤 다시 넣어준다.

두번째 수정의 경우 `equals()`와 `hashCode()`를 override 하여 동일한 값을 찾아내서 제거한다. 그 후 새로운 값을 넣어준다.
(equals와 hashCode를 재정의 하지 않으면 객체 내 필드 값으로 비교하지 않기 때문에 위와 같이 remove 할 수 없다.)

컬렉션 수정은 update를 수행하지 않고 **FK(MEMBER_ID)를 기준으로 전부 delete를 한뒤 컬렉션에 남아있는 아이템들을 다시 전부 insert 하는 쿼리를 수행**한다.

그래서 2번째 수정의 경우 delete 쿼리 한번(전부 삭제), insert 쿼리 두번(기존 city인 old2, 새롭게 추가된 city인 new)을 수행한다.

#### 값 타입 컬렉션의 제약사항 정리

- 값 타입은 엔티티와 다르게 식별자 개념이 없다.
  - `@Id` 같은 식별자가 없기 때문에 find 같은 메서드 사용이 불가능하다.
  - 그래서 값을 변경하면 추적이 어렵다. <br/>
- 값 타입 컬렉션에 변경 사항이 발생하면, **주인 엔티티와 연관된 모든 데이터를 삭제하고, 값 타입 컬렉션에 있는 현재 값을 모두 다시 저장한다.**
  - 위에 수정 예제 참고
  - `@OrderColumn`을 통해 컬렉션 순서에 대한 컬럼을 추가하면 update 쿼리를 수행하게 바꿀 수 있다.
    - 그러나 컬렉션 중간에 아이템이 비는경우 null이 들어가는 등 이슈가 발생할 수 있기 때문에 권장하지 않는다.<br/>
- 값 타입 컬렉션을 매핑하는 테이블은 **모든 컬럼을 묶어서 기본키를 구성**해야한다.
  - 그래서 컬렉션 객체 속성은 null을 입력할 수 없고, 중복 저장이 불가능하다.


#### 값 타입 컬렉션 대안

실무에서는 상황에 따라 값 타입 컬렉션을 사용하는 **대신에 일대다 관계를 사용하는 것을 추천**한다.

일대다 관계를 위한 엔티티를 만들고, 여기에서 값 타입을 매핑하여 사용한다. (값 타입을 엔티티로 승격시켜서 사용)

영속성전이(Cascade) + 고아 객체 제거를 사용해서 값 타입 컬렉션 처럼 사용한다. 

```java
@Embeddable
@Getter
@AllArgsConstructor
public class Address {
   private String city;
   private String street;
   private String zipcode;
}

// 컬렉션 값 타입 대체를 위해 엔티티로 직접 구현
@Entity
@Table(name = "ADDRESS")
public class AddressEntity {
  @Id @GeneratedValue
  private Long id;

  private Address address;

  public AddressEntity(String city, String street, String zipcode){
    this.address = new Address(city, street, zipcode);
  }
}

@Entity
@Getter
@Setter
@EqualsAndHashCode
public class Member {
  @Id
  @GeneratedValue
  private Long id;

  private String name;

  // @ElementCollection
  // @CollectionTable(name = "ADDRESS", joinColumns = @JoinColumn(name="MEMBER_ID"))
  // private List<Address> addressHistory = new ArrayList<>();

  // 컬렉션 값 타입을 엔티티(1:N 구조)로 풀기
  @OneToMany(Cascade = CascadeType.ALL, orphanRemoval = true)
  @JoinColumn(name = "MEMBER_ID")
  private List<AddressEntity> addressHistory = new ArrayList<>();
}

...

Member member = new Member();
member.setName("member1");

member.getAddressHistory().add(new AddressEntity("old1", "street", "10000"));
member.getAddressHistory().add(new AddressEntity("old2", "street3", "2412"));

em.persist(member);
```

그렇다면 값 타입 컬렉션은 언제 사용하면 좋을까?

위 예제의 `favoriteFoods`처럼 **update가 일어나지 않게 아이템들이 정해져있고 단순한 내용들을 다룰 때** 사용하면 좋다. 

그러나 실제 실무에서는 값 타입 컬렉션을 사용할 상황이 많이 나오지 않는다.

### 값 타입 공유참조

- 임베디드 타입 같은 값 타입을 **여러 엔티티에서 공유하면 위험하다.**
- 부작용(side effect) 발생

![image](https://user-images.githubusercontent.com/36228833/200013090-949d7fd9-06d4-48fc-bfa5-000bcc88f1ec.png)

회원1 엔티티와 회원2 엔티티가 둘다 city를 보고있을 때 city가 OldCity에서 NewCity로 변경 시 회원1과 회원2에 영향이 생긴다.(NewCity로 바뀜)

```java
@Embeddable
@Getter
@Setter
@AllArgsConstructor
public class Address {
   private String city;
   private String street;

   @Column(name="ZIP_CODE")
   private String zipcode;
}

@Entity
@Getter
@Setter
public class Member {
  @Id
  @GeneratedValue
  private Long id;

  private String name;

  @Embedded
  private Address homeAddress;
}

...

Address address = new Address("oldCity", "street", "12424");

Member member = new Member();
member.setName("member1");
member.setHomeAddress(address);  // 동일한 임베디드 타입 사용
em.persist(member);

Member member2 = new Member();
member2.setName("member2");
member2.setHomeAddress(address); // 동일한 임베디드 타입 사용
em.persist(member2);

// 의도: 첫번쨰 member의 주소만 newCity로 바꿔야겠다!
// side effect 발생) 그러나 두번째 member 또한 newCity로 바뀌게 된다.
member.getHomeAddress().setCity("newCity");

```

이러한 side effect로 인하여 의도치않게 두번째 멤버 또한 값이 변경되게 된다.

이런식으로 값 타입의 실제 인스턴스인 값을 공유하는것은 매우 위험하다.

대신 값(인스턴스)를 복사해서 사용해야 한다!

![image](https://user-images.githubusercontent.com/36228833/200013363-2ba577b2-305a-4624-a01e-83844db6ba5c.png)

```java
...

Address address = new Address("oldCity", "street", "12424");

Member member = new Member();
member.setName("member1");
member.setHomeAddress(address);
em.persist(member);

// 동일한 임베디드 타입을 사용하지 않고 값을 복사하여 새로운 인스턴스를 생성한다.
Address copyAddress = new Address(address.getCity(), address.getStreet(), address.getZipcode());

Member member2 = new Member();
member2.setName("member2");
member2.setHomeAddress(copyAddress);  // 새로운 인스턴스
em.persist(member2);

// 의도대로 첫번쨰 member의 주소만 newCity로 바뀐다
member.getHomeAddress().setCity("newCity");
```

그러나 만약 다음과 같이 기존 값을 그대로 사용하게 된다면?

```java
...

Address address = new Address("oldCity", "street", "12424");

Member member = new Member();
member.setName("member1");
member.setHomeAddress(address);
em.persist(member);

// 동일한 임베디드 타입을 사용하지 않고 값을 복사하여 새로운 인스턴스를 생성한다.
Address copyAddress = new Address(address.getCity(), address.getStreet(), address.getZipcode());

Member member2 = new Member();
member2.setName("member2");
member2.setHomeAddress(address); // 기존 인스턴스를 사용
em.persist(member2);

// side effect 발생
member.getHomeAddress().setCity("newCity");
```

- 항상 값을 복사해서 사용하면 공유 참조로 인해 발생하는 부작용을 피할 수 있다.
  - 그러나 값을 복사한 새로운 인스턴스를 만들었어도 누군가가 기존 인스턴스 객체를 사용하는것을 막을수가 없다. <br/>

- 문제는 임베디드 타입처럼 직접 정의한 값 타입은 자바의 기본 타입(primitive type)이 아니라 객체 타입이다.
  - 기본 타입은 값을 할당 시 (=) 무조건 기존값이 복사되어 넘어가게된다. (값을 공유하지 않는다.)
    - 그렇기 때문에 절대 값이 같을 수 없다. <br/>
      ```java
      // 기본 타입 (primitive type)
      int a = 10; 
      int b = a; // 값을 복사
      b = 4;
      ```

  - 그러나 객체 타입은 다르다. 
    - 객체 타입은 참조 값을 직접 대입하는것을 막을 방법이 없다. <br/>
      ```java
      // 객체 타입
      Address a = new Address("old");
      Address b = a; // 참조를 전달
      b.setCity("New") // side effect 발생 (a도 New로 바뀜)
      ```
 
#### 불변 객체

객체 타입의 side effect를 막을 방법은 없는 걸까?

객체 타입을 수정할 수 없게 만들면 된다. 즉 불변 객체(immutable)로 설계해야한다.
- 불변객체 : 생성 시점 이후 절대 값을 변경할 수 없는 객체

생성자(Constructor)로만 값을 설정하고 수정자(Setter)를 만들지 않으면 된다!
- Integer, String은 자바가 제공하는 대표적인 불변 객체이다.

```java
@Embeddable
@Getter
//@Setter // Setter를 막아서 immutable 하게 설계한다.
@AllArgsConstructor
public class Address {
   private String city;
   private String street;

   @Column(name="ZIP_CODE")
   private String zipcode;
}

...

Address address = new Address("oldCity", "street", "12424");

Member member = new Member();
member2.setName("member");
member2.setHomeAddress(address);
em.persist(member2);

// 오류 발생 (setCity 사용 불가능)
member.getHomeAddress().setCity("newCity");
```

결국 불변이라는 작은 제약(Setter 삭제)으로 부작용이라는 큰 재앙을 막을 수 있다!

만약 값을 바꿔야할 상황이 온다면 다음과 같이 객체를 다시 생성하자.

```java
Address address = new Address("oldCity", "street", "12424");

Address copyAddress = new Address("oldCity", address.getStreet(), address.getZipcode());
```

### 값 타입의 비교

값 타입을 어떻게 비교하는지 알아보자.

값 타입은 **인스턴스가 달라도 그안에 값이 같으면 같은것**으로 봐야한다.

우리가 흔히 아는 기본 타입(primitive type)의 경우 참조가 아닌 순수 값으로 비교하기 때문에 `==` 으로 비교하면 `true`가 나온다.

```java
int a = 10;
int b = 10;

System.out.println(a == b); // true
```

그렇다면 객체 타입의 비교는 어떨까?

다음과 같이 객체끼리 비교할 때 `==` 으로 비교하면 객체간의 참조값이 서로 다르므로 당연히 `false`가 나온다.

```java
Address a = new Address("서울시");
Address b = new Address("서울시");

System.out.println(a == b); // false
```

그렇다면 객체 타입(인스턴스)의 비교는 어떻게 해야할까?

비교 방식은 다음과 같이 2가지로 나뉜다.

- 동일성(identity) 비교
  - 인스턴스의 **참조 값을 비교**한다. `==`를 사용한다.

- 동등성(equivalence) 비교
  - 인스턴스의 **값을 비교**한다. `equals()`를 사용한다.

그렇다면 **equals()**를 사용해서 객체간의 동등성 비교를 해보자.

```java
Address a = new Address("서울시");
Address b = new Address("서울시");

System.out.println(a.equals(b)); // false
```

`equals()`를 사용해도 false가 출력되는 것을 확인할 수 있다. 그 이유는 기본적으로 `equals()`는 `==`을 사용하여 비교하기 때문이다.

`==`를 사용하여 비교하게 되면 단순히 **객체의 참조 값만을 비교하기 때문에** 값에 대한 비교를 할 수 없다. 

그래서 `equals()`로 **객체 필드들의 모든 값을 비교하여 사용하기 위해** 재정의를 하여 사용할 필요가 있다.

```java
@Embeddable
@Getter
@AllArgsConstructor
public class Address {
   private String city;
   private String street;

   @Column(name="ZIP_CODE")
   private String zipcode;

   ...
   

   // equals 재정의 (만일 프록시가 들어가거나 복잡하게 구조가 짜여져 있는경우, 필드에 직접 접근이 아닌 getter로 불러와야 할 수 있다.)
   @Override
   public boolean equals(Object o){
     if(this == o) return true;
     if(o == null || getClass() != o.getClass()) return false;
     Address address = (Address) o;
     return Objects.equals(this.city, address.city) &&
             Objects.equals(this.street, address.street) &&
             Objects.equals(this.zipcode, address.zipcode);
   }
   
   @Override
   public int hashCode() {
     return Objects.hash(this.city, this.street, this.zipcode);
   }
}

...

Address a = new Address("서울시", "강남구", "123123");
Address b = new Address("서울시", "강남구", "123123");

System.out.println(a.equals(b)); // true (equals 하나로 객체 내 모든 필드를 비교)
```

`equals()`를 재정의 할 때 `hashCode()`도 같이 재정의하여 구현하면 hash 값을 사용하는 Collection(HashMap, HashSet, HashTable)들을 사용할 때 동등객체 비교를 수월하게 할수 있다.

hash 값을 사용하는 Collection에서 값을 비교할 때 **기본적으로 객체의 `hashCode()`로 인스턴스들을 비교**하게 되는데 `hashCode()`는 객체마다 랜덤으로 부여되기 때문에 논리적으로 비교하기 어렵다.

(단, 기본 `hashCode()`는 인스턴스의 경우 주소 값을 기반으로 생성되고 String의 경우 문자열의 ascii 코드값으로 생성되기 때문에 상황에 따라 같을 수 있다.)

그리고 최종적으로 **`equals()`와 `hashCode()` 두가지 경우 모두 `true`로 반환해야 같다고 보기 때문에** 둘다 재정의해서 사용하는 편이 좋다.


```java
HashMap<Integer, Address> hashMap = new HashMap<>();
hashMap.put(1, new Address("서울시", "강남구", "123123"));
hashMap.put(2, new Address("서울시", "강남구", "123123"));

System.out.println(hashMap.get(1).equals(hashMap.get(2))); // false
```

## 정리

- 엔티티 타입
  - `@Id` 같은 식별자가 존재한다.
  - 생명 주기를 스스로 관리한다.
  - 공유
- 값 타입
  - 식별자 X
  - 생명 주기를 엔티티에 의존
  - 공유하지 않는 것이 안전하다.(복사해서 사용하기)
  - 불변 객체로 만드는 것이 안전하다.

## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)<br/>
