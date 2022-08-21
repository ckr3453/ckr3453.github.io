---
title: "스프링 컨테이너의 싱글톤 패턴 활용"
categories: 
    - Spring
date: 2022-08-21
last_modified_at: 2022-08-21
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
---

지난 포스팅에 스프링 컨테이너에 대하여 간략하게 설명했다. 이번 시간에는 스프링 컨테이너가 스프링 빈을 어떻게 관리하는지에 대하여 구체적으로 알아보자.

## 상황 발생
![image](https://user-images.githubusercontent.com/36228833/185788559-adf560af-25da-4333-8ba5-1e7b151f3739.png)
- 웹 어플리케이션은 보통 여러 고객이 동시에 요청을 한다.
- 클라이언트 요청에 따라 DI 컨테이너에 등록된 스프링 빈을 매번 생성하여 반환을 해주어야 한다.
- 객체를 매번 생성하여 반환하는 이런 방식은 시스템 자원의 소모가 너무 극심하기 때문에 좋지 않다.
  - ex) 초당 100개의 요청이 발생 시 초당 100개의 객체가 생성되고 소멸 되어야한다. (메모리 낭비)
- 이와 같은 문제는 싱글톤 패턴을 적용하여 해결할 수 있다. (객체가 딱 1개만 생성되고 공유하도록 설계)

## 싱글톤 패턴 (Singleton pattern)
- 클래스의 인스턴스가 딱 1개만 생성되는 것을 보장하는 디자인 패턴이다.
- 스프링 빈 객체에 싱글톤 패턴을 적용함으로써 이미 만들어진 빈 객체를 공유해서 효율적으로 사용할 수 있다.

### 구현
```java
public class SingletonService {

    // static 영역에 객체를 1개만 생성한다.
    private static final SingletonService instance = new SingletonService(); // 자기자신을 static에 올림

    // 객체 인스턴스가 필요하면 해당 메서드를 통해서만 조회하도록 허용한다.
    public static SingletonService getInstance(){
        return instance;
    }

    // 생성자를 private로 선언하여 외부에서 new 키워드를 사용한 객체 생성을 제한한다. (자기 자신만 생성자 호출 가능)
    private SingletonService(){
    }

    public void logic(){
        System.out.println("싱글톤 객체 로직 호출");
    }

}
```

하지만 싱글톤 패턴은 다음과 같은 문제점을 가지고 있다.

### 문제점
- 싱글톤 패턴을 구현하는 코드 자체가 많이 들어간다.
- 의존관계상 클라이언트가 구체 클래스에 의존한다. DIP를 위반한다.
- 클라이언트가 구체 클래스에 의존해서 OCP 원칙을 위반할 가능성이 높다.
- 테스트하기 어렵다.
- 내부 속성을 변경하거나 초기화 하기 어렵다.
- private 생성자로 자식 클래스를 만들기 어렵다.
- 결론적으로 유연성이 떨어진다.
- 안티패턴으로 불리기도 한다

## 스프링 컨테이너의 싱글톤 패턴 적용
스프링 컨테이너는 이러한 싱글톤 패턴의 문제점을 해결하면서, 객체 인스턴스를 싱글톤(1개만 생성)으로 관리한다.

- 스프링 컨테이너는 싱글턴 패턴을 적용하지 않아도, 객체 인스턴스를 싱글톤으로 관리한다.
  - 이전에 설명한 컨테이너 생성 과정을 자세히 보자. 컨테이너는 객체를 하나만 생성해서 관리한다.
- 스프링 컨테이너는 싱글톤 컨테이너 역할을 한다. 이렇게 싱글톤 객체를 생성하고 관리하는 기능을 **싱글톤 레지스트리**라 한다.
- 스프링 컨테이너의 이런 기능 덕분에 싱글턴 패턴의 모든 단점을 해결하면서 객체를 싱글톤으로 유지할 수 있다.
  - 싱글톤 패턴을 위한 지저분한 코드가 들어가지 않아도 된다.
  - DIP, OCP, 테스트, private 생성자로 부터 자유롭게 싱글톤을 사용할 수 있다.

![image](https://user-images.githubusercontent.com/36228833/185789127-1489b0be-fa73-4853-bdb2-0e6326b42e0d.png)
- 스프링 컨테이너 덕분에 고객의 요청이 올 때 마다 객체를 생성하는 것이 아니라, 이미 만들어진 객체를 공유해서 효율적으로 재사용할 수 있다.

> default는 싱글톤 방식이지만 스프링이 싱글톤 방식만 지원하는 것은 아니다. 요청할 때 마다 새로운 객체를 생성해서 반환하는 기능도 제공한다.

### 테스트
```java
@Test
@DisplayName("스프링 컨테이너의 싱글톤 패턴 적용")
void springContainer() {
  ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
  
  MemberService memberService = ac.getBean("memberService", MemberService.class);
  MemberService memberService2 = ac.getBean("memberService", MemberService.class);

  //참조값이 같은 것을 확인
  System.out.println("memberService = " + memberService);
  System.out.println("memberService2 = " + memberService2);

  //memberService1 == memberService2
  assertThat(memberService1).isSameAs(memberService2);
}
```

**결과**<br/>
![image](https://user-images.githubusercontent.com/36228833/185789088-df3289a9-45de-4f99-9d33-033cacc7df20.png)

- 스프링 컨테이너 덕분에 고객의 요청이 올 때 마다 객체를 생성하는 것이 아니라, 이미 만들어진 객체를 공유해서 효율적으로 재사용할 수 있다.

## 주의할점
- 싱글톤 패턴이든, 스프링 같은 싱글톤 컨테이너를 사용하든, 객체 인스턴스를 하나만 생성해서 공유하는 싱글톤 방식은 여러 클라이언트가 하나의 같은 객체 인스턴스를 공유하기 때문에 싱글톤 객체는 상태를 유지(stateful)하게 설계하면 안된다.
- 무상태(stateless)로 설계해야 한다!
  - 특정 클라이언트에 의존적인 필드가 있으면 안된다.
  - 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안된다!
  - 가급적 읽기만 가능해야 한다.
  - 필드 대신에 자바에서 공유되지 않는, 지역변수, 파라미터, ThreadLocal 등을 사용해야 한다.
- 스프링 빈의 필드에 공유 값을 설정하면 정말 큰 장애가 발생할 수 있다.

아래 예시를 통해 확인해보자.

**StatefulService.java**
```java
public class StatefulService {
  private int price; //상태를 유지하는 필드

  public void order(String name, int price) {
    System.out.println("name = " + name + " price = " + price);
    this.price = price; //여기가 문제!
  }

  public int getPrice() {
    return price;
  }
}
```

**테스트**
```java
public class StatefulServiceTest {
  @Test
  void statefulServiceSingleton() {
    ApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);
    StatefulService statefulService1 = ac.getBean("statefulService", StatefulService.class);
    StatefulService statefulService2 = ac.getBean("statefulService", StatefulService.class);
    //ThreadA: A사용자 10000원 주문
    statefulService1.order("userA", 10000);
    //ThreadB: B사용자 20000원 주문
    statefulService2.order("userB", 20000);
    //ThreadA: 사용자A 주문 금액 조회
    int price = statefulService1.getPrice();
    //ThreadA: 사용자A는 10000원을 기대했지만, 기대와 다르게 20000원 출력
    System.out.println("price = " + price);
    Assertions.assertThat(statefulService1.getPrice()).isEqualTo(20000);
  }

  static class TestConfig {
    @Bean
    public StatefulService statefulService() {
      return new StatefulService();
    }
  }
}
```

- ThreadA가 사용자A 코드를 호출하고 ThreadB가 사용자B 코드를 호출한다 가정하자.
- `StatefulService` 의 `price` 필드는 공유되는 필드인데, 특정 클라이언트가 값을 변경한다.
- 사용자A의 주문금액은 10000원이 되어야 하는데, 20000원이라는 결과가 나왔다.
- 이로인해 정말 해결하기 어려운 큰 문제들이 터진다.
- 진짜 공유필드는 조심해야 한다! 스프링 빈은 항상 무상태(stateless)로 설계하자.

## @Configuration의 바이트코드 조작
스프링 컨테이너는 싱글톤 레지스트리다. 따라서 스프링 빈이 싱글톤이 되도록 보장해주어야 한다. 그래서 스프링은 클래스의 바이트코드를 조작하는 라이브러리를 사용한다.

다음 예를 확인해보자.

**AppConfig.java**
```java
@Configuration
public class AppConfig {
  @Bean
  public MemberService memberService() {
    return new MemberServiceImpl(memberRepository());
  }
  @Bean
  public OrderService orderService() {
    return new OrderServiceImpl(memberRepository(), discountPolicy());
  }
  @Bean
  public MemberRepository memberRepository() {
    return new MemoryMemberRepository();
  }
  @Bean
  public DiscountPolicy discountPolicy() {
    return new RateDiscountPolicy();
  }
}
```

**테스트**
```java
@Test
void configurationDeep() {
  ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
  AppConfig bean = ac.getBean(AppConfig.class);
 
  System.out.println("bean = " + bean.getClass());
}
```

- 다음과 같이 스프링 빈의 클래스 정보를 출력하면 다음과 같이 출력된다.

![image](https://user-images.githubusercontent.com/36228833/185789770-713d0618-e5f3-4b51-bc6a-1b5664ada43e.png)

순수한 클래스라면 다음과 같이 출력되어야 한다.
- `bean = class com.inflearn.demo.AppConfig`

그런데 예상과는 다르게 클래스 명에 **EnhancerBySpringCGLIB**가 붙으면서 상당히 복잡해진 것을 볼 수 있다. 이것은 내가 만든 클래스가 아니라 스프링이 `CGLIB`라는 바이트코드 조작 라이브러리를 사용해서 `AppConfig` 클래스를 상속받은 임의의 다른 클래스를 만들고, 그 다른 클래스를 스프링 빈으로 등록한 것이다!

![image](https://user-images.githubusercontent.com/36228833/185789833-dc8ba605-8996-4423-950b-23d668641ddc.png)

- 그 임의의 다른 클래스가 바로 싱글톤이 보장되도록 해준다.
- `@Bean`이 붙은 메서드마다 이미 스프링 빈이 존재하면 존재하는 빈을 반환하고, 스프링 빈이 없으면 생성해서 스프링 빈으로 등록하고 반환하는 코드가 동적으로 만들어진다.
- 덕분에 싱글톤이 보장되는 것이다.

## @Configuration 없이 @Bean만 적용
`@Configuration` 을 붙이면 바이트코드를 조작하는 `CGLIB` 기술을 사용해서 싱글톤을 보장하지만, 만약 `@Bean`만 적용하면 어떻게 될까?

`AppConfig.java`에 @Configuration를 주석 처리하고 위의 테스트를 다시 실행해보자.

![image](https://user-images.githubusercontent.com/36228833/185790102-b94d6cde-dd3f-4070-8f6a-81e7d6efa3ae.png)

AppConfig가 CGLIB 기술 없이 순수한 AppConfig로 스프링 빈에 등록된 것을 확인할 수 있다. (싱글톤을 보장하지 않음)

## 📣 Reference
본 포스팅은 김영한님의 강의를 듣고 스스로 정리 및 추가한 내용입니다.

[스프링 핵심 원리 - 기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8)<br/>
