---
title: "Spring Webflux) Project Reactor" (작성중)
categories: 
    - spring
date: 2023-09-23
last_modified_at: 2023-10-24
toc: true
toc_sticky: true
excerpt: "Reactor"
---

## Reactor

<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/9f291d31-b723-4310-b031-384958863950"></center><br/>

- Pivotal 사에서 개발하였다.
- Spring reactor에서 사용한다
- Mono와 Flux publisher를 제공한다.

## Flux

<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/38331f13-d0bb-43ab-8497-66a7816e1035"></center><br/>

- 0..n개의 item을 전달한다.
- 에러가 발생하면 error signal을 전달하고 종료된다.
- 모든 item을 전달했다면 complete signal을 전달하고 종료된다.
- backPressure를 지원한다.
  - request를 통해 요청수를 조절할 수 있다.

```java
@RequiredArgsConstructor
public class SimpleSubscriber<T> implements Subscriber<T> { 
    private final Integer count;

    @Override
    public void onSubscribe(Subscription s) {
        log.info("subscribe");
        s.request(count);
        log.info("request: {}", count);
    }

    @SneakyThrows
    @Override
    public void onNext(T t) {
        log.info("item: {}", t);
        Thread.sleep(100);
    }

    @Override
    public void onError(Throwable t) {
        log.error("error: {}", t.getMessage());
    }

    @Override
    public void onComplete() {
        log.info("complete");
    }
}
```

```java
@Slf4j
public class FluxSimpleExample { 
    public static void main(String[] args) {
        log.info("start main");
        getItems().subscribe(new SimpleSubscriber<>(Integer.MAX_VALUE));
        log.info("end main");
    }

    private static Flux<Integer> getItems() {
        return Flux.fromIterable(List.of(1, 2, 3, 4, 5));
    }
}
```

```java
09:32:48 [main] - start main
09:32:48 [main] - subscribe
09:32:48 [main] - request: 2147483647 
09:32:48 [main] - item: 1
09:32:48 [main] - item: 2
09:32:48 [main] - item: 3
09:32:49 [main] - item: 4
09:32:49 [main] - item: 5
09:32:49 [main] - complete
09:32:49 [main] - end main
```

### subscribeOn

- 다른 thread에서 sub를 실행할 수 있다.
- CompletableFuture 문제점중 하나는 CompletableFuture 반환하는 함수를 실행할때 지연로딩이 불가능했었다.

```java
@Slf4j
public class FluxSimpleSubscribeOnExample {
    @SneakyThrows
    public static void main(String[] args) { 
        log.info("start main");
        getItems()
            .map(i -> {
                log.info("map {}", i);
                return i; 
            })
            .subscribeOn(Schedulers.single()) // 사용할 특정 thread 를 등록
            .subscribe(new SimpleSubscriber<>(Integer.MAX_VALUE)); 
            
            log.info("end main");
            Thread.sleep(1000);
    }
    
    private static Flux<Integer> getItems() {
        return Flux.fromIterable(List.of(1, 2, 3, 4, 5)); 
    }
}
```

```java
09:57:21 [main] - start main
09:57:21 [main] - subscribe
09:57:21 [main] - request: 2147483647 
09:57:21 [main] - end main
09:57:21 [single-1] - map 1
09:57:21 [single-1] - item: 1
09:57:21 [single-1] - map 2
09:57:21 [single-1] - item: 2
09:57:22 [single-1] - map 3
09:57:22 [single-1] - item: 3
09:57:22 [single-1] - map 4
09:57:22 [single-1] - item: 4
09:57:22 [single-1] - map 5
09:57:22 [single-1] - item: 5
09:57:22 [single-1] - complete
```



## 📣 Reference
[패스트캠퍼스 - Spring Webflux 완전정복 : 코루틴부터 리액티브 MSA 프로젝트까지](https://fastcampus.co.kr/dev_online_webflux)<br/>

