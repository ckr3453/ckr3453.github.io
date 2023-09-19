---
title: "Spring Webflux) 개요"
categories: 
    - spring
date: 2023-09-11
last_modified_at: 2023-09-11
toc: true
toc_sticky: true
excerpt: "Spring webflux에 대해 간단히 알아본다."
---

## Spring Webflux?

<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/97ff2702-a0ed-4cac-a0d1-05130a1c1eb0"></center><br/>

Spring 5.0부터 지원하는 Reactive web framework 다.

Reactive Streams API를 지원하여 Reactive 스타일의 어플리케이션 개발을 할수 있다.

> Reactive Programming? 데이터 스트림과 변화의 전파를 중심으로 한 비동기 프로그래밍 패러다임이다. 즉, 데이터의 변화를 이벤트로 하여 수/송신자 사이에 데이터 스트림을 주고받는다.

## Spring MVC 와 차이점

<center><img width="600" height="600" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/67688927-2339-4a1d-8edd-bc282e995841"></center><br/>

- Spring MVC
  - 동기적으로 동작하는 Blocking 방식
  - 코드 작성과 디버깅에 용이함
  - Thread pool을 활용하여 사용자의 요청에 대응 (Thread per request model)
    - 다수의 사용자 요청이 들어올 경우 그만큼 많은 리소스를 사용해야함
    - Context switching 비용 발생
    - Thread pool size를 초과할경우, Thread Pool Hell 현상 발생가능
  - JDBC, JPA 지원

- Spring Webflux
  - 비동기적으로 동작하는 Non-blocking 방식
  - event loop를 활용하여 thread를 block 하지 않음 (Event Loop model)
    - callback 형식으로 thread에 작업을 요청하고 완료시 응답을 받음
    - Context switching의 overhead가 줄어들음
    - (thread block 하지 않기 때문에) 단일(혹은 최소한의) thread로도 최대 성능 보장
      - thread 가 쉬지않고 계속해서 일함
  - RxJava, Reactor 지원

## Spring Webflux 장/단점

- 장점
  - 효율적으로 리소스를 사용함
  - 요청이 순간적으로 늘어나도 유연하게 커버 가능함
  - 동시성을 극한으로 이용하여 응답속도를 단축
  - Reactor, Coroutine으로 코드 가독성을 유지가능
    - 코드 자체는 동기, 절차지향적인 스타일로 작성
    - 그러나 내부적으로는 비동기적으로 동작

- 단점
  - 비동기 로직 처리에 대한 고민이 필요함
    - 결과가 나중에 들어옴
    - 해당 결과를 다른 thread로 일임하여 핸들링 해야하는 경우가 발생
    - 때문에 디버깅, 에러 핸들링 할때 어려움 발생
  - 아직 완벽하지 않은 생태계

## 그렇다면 무조건 Spring Webflux가 더 좋은가?

<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/b4f2cda2-ade4-4535-b2b4-c67815fec124">
</center><br/>


구축하는 서비스가 다량의 동시접속자를 대응해야 한다면 Spring Webflux를 사용하는 것이 좋다. 

또한 요청을 처리하는 파이프라인의 구성요소들이 전부 Non-blocking 하게 동작해야 최상의 성능을 이끌어낼 수 있다.

특정한 구간에서 Blocking이 발생하는 경우, thread block 이 발생하며 이는 곧 성능 저하로 직결된다.<br/>
(보통 Webflux 환경에서는 Spring MVC 환경보다 상대적으로 더 적은 스레드(core * 2)를 운용하므로 치명적)

그렇지 않은경우 Spring MVC를 사용하는 것이 코드작성, 디버깅 등 생산성 면에서 훨씬 좋다. 

## 📣 Reference
[패스트캠퍼스 - Spring Webflux 완전정복 : 코루틴부터 리액티브 MSA 프로젝트까지](https://fastcampus.co.kr/dev_online_webflux)<br/>
[Spring Framework docs - Spring Webflux](https://docs.spring.io/spring-framework/reference/web/webflux.html)<br/>
[자바 기반의 스프링 Web MVC 와 WebFlux 성능 분석](https://manuscriptlink-society-file.s3-ap-northeast-1.amazonaws.com/kips/conference/kips2020spring/KIPS_C2020A0050.pdf)<br/>
[Spring Webflux와 WebClient](https://velog.io/@rnqhstlr2297/Spring-Webflux%EC%99%80-WebClient#reactor%EB%9E%80-)<br/>
[Spring5 리액티브 스트림 정리 및 api 전달 방식 정리](https://wedul.site/624)<br/>
[Spring WebFlux는 어떻게 적은 리소스로 많은 트래픽을 감당할까?](https://alwayspr.tistory.com/44#Spring%20MVC-1)<br/>
[Webflux 진짜로 대용량 처리에서 MVC 보다 빠를까??](https://okky.kr/questions/1412539)<br/>
[[Spring Webflux] 웹플럭스에 대한 간단 정리](https://developer-jiing.tistory.com/53)<br/>
[내가 만든 WebFlux가 느렸던 이유](https://forward.nhn.com/2020/session/26)<br/>

