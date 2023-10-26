---
title: "Spring Webflux) Project Reactor"
categories: 
    - spring
date: 2023-09-23
last_modified_at: 2023-10-25
toc: true
toc_sticky: true
excerpt: "Reactor"
---

## Reactor

<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/9f291d31-b723-4310-b031-384958863950"></center><br/>

- Pivotal 사에서 개발하였다.
- Spring reactor에서 사용한다
- Mono와 Flux 라는 publisher를 제공한다.

## Flux

<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/38331f13-d0bb-43ab-8497-66a7816e1035"></center><br/>

- 0..n개의 item을 전달한다.
  - 마치 List<T>와 유사 -> 무한하거나 유한한 여러개의 값
  - ex) repository.findAll()
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

### subscribe

- publisher를 통해 subscribe를 하지 않으면 아무일도 일어나지 않는다.

```java
@Slf4j
public class FluxSubscribeExample {
    public static void main(String[] args) {
        log.info("start main");
        // Flux 객체를 생성하여 아이템을 뽑았으나 결과적으로 subscribe를 하지않아 출력되지 않는다.
        getItems();
        log.info("end main");
    }

    private static Flux<Integer> getItems() {
        return Flux.create(fluxSink -> {
            log.info("start getItems");
            for(int i=0; i<5; i++) {
                fluxSink.next(i);
            }
            fluxSink.complete();
            log.info("end getItems");
        })
    }
}
```

```java
09:22:23 [main] - start main
09:22:23 [main] - end main
```

### backPressure

```java
@Slf4j
public class Example {
    public static void main(String[] args) {
        getItems().subscribe(new ContinuousRequestSubscriber<>());
    }

    private static Flux<Integer> getItems() {
        return Flux.fromIterable(List.of(1, 2, 3, 4, 5));
    }
}
```

```java
// Subscriber를 구현
@Slf4j
public class ContinuousRequestSubscriber<T> implements Subscriber<T> {
    private final Integer count = 1;
    private Subscription subscription = null;

    // onSubscribe를 통해 받은 subp의 request를 통해 backPressure를 조절한다.
    @Override
    public void onSubscribe(Subscription s) {
        this.subscription = s;
        log.info("subscribe");
        // subscribe시 최초로 데이터를 1개 요청한다.
        s.request(count);
        log.info("request: {}", count);
    }

    @SneakyThrows
    @Override
    public void onNext(T t) {
        log.info("item: {}", t);
        Thread.sleep(1000);
        // 이후 다음 데이터들을 1개씩 계속해서 요청한다.
        subscription.request(1);
        log.info("request: {}", count);
    }
}
```

```java
09:27:29 [main] - subscribe
09:27:29 [main] - request: 1
09:27:29 [main] - item: 1
09:27:30 [main] - request: 1
09:27:30 [main] - item: 2
09:27:31 [main] - request: 1
09:27:31 [main] - item: 3
09:27:32 [main] - request: 1
09:27:32 [main] - item: 4
09:27:33 [main] - request: 1
09:27:33 [main] - item: 5
09:27:34 [main] - request: 1
09:27:34 [main] - complete
```

### error

```java
@Slf4j
public class FluxErrorExample {
    public static void main(String[] args) {
        log.info("start main");
        getItems().subscribe(new SimpleSubscriber<>(Integer.MAX_VALUE));
        log.info("end main");
    }

    private static Flux<Integer> getItems() { 
        return Flux.create(fluxSink -> {
            fluxSink.next(0);
            fluxSink.next(1);
            // sub에게 데이터를 2개 넘긴 이후 에러 발생
            // 만약 에러를 만나게 되면 더이상 진행하지않고 바로 종료됨
            var error = new RuntimeException("error in flux"); 
            fluxSink.error(error);
        }); 
    }
}
```

```java
10:08:09 [main] - start main
10:08:09 [main] - subscribe
10:08:09 [main] - request: 2147483647
10:08:09 [main] - item: 0
10:08:09 [main] - item: 1
10:08:10 [main] - error: error in flux
10:08:10 [main] - end main
```

### complete

```java
@Slf4j
public class FluxCompleteExample {
    public static void main(String[] args) {
        log.info("start main");
        getItems().subscribe(new SimpleSubscriber<>(Integer.MAX_VALUE));
        log.info("end main");
    }
    
    private static Flux<Integer> getItems() { 
        return Flux.create(fluxSink -> {
            // sub에게 아무 값도 넘기지 않은 상태에서 바로 complete 할 수 있다.
            fluxSink.complete(); 
        });
    }
}
```

```java
10:08:54 [main] - start main
10:08:54 [main] - subscribe
10:08:54 [main] - request: 2147483647
10:08:54 [main] - complete
10:08:54 [main] - end main
```

## Mono

<center><img width="691" alt="스크린샷 2023-10-25 오후 11 51 04" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/4f855c9f-5188-4527-8dfe-362964c97d46"></center><br/>


- 0..1개의 item을 전달한다.
  - 마치 Optional<T>과 유사 -> 없거나 혹은 존재하는 하나의 값
  - ex) repository.findOne(), repository.count() 등
- 에러가 발생하면 error signal을 전달하고 종료한다.
- 모든 item을 전달했다면 complete signal을 전달하고 종료한다.
- Mono<Void>로 특정 사건이 완료되는 시점을 가리킬수도 있다.
  - 예를들면, 특정 사건에 대한 종료를 알아야한다.
  - Mono를 pub으로 사용하는 구성요소는 이벤트가 끝난 뒤 Mono<Void>로 내부 데이터 없이 텅 빈상태로 success를 함으로써 시점 반환 역할을 할 수 있다.

### Flux를 사용해서 하나의 값만 넘겨주면 되는데 굳이 Mono를 사용하는 이유는?

- Mono는 1개의 item만 전달하기 때문에 next가 한번 실행되면 이후 complete 실행이 보장된다.
  - 혹은 값을 전달하지 않고 complete를 하면 값이 없다는 것을 의미한다.
- 하나의 값이 있거나 없다.

```java
@Slf4j
public class MonoSimpleExample {
    @SneakyThrows
    public static void main(String[] args) { 
        log.info("start main");
        getItems().subscribe(new SimpleSubscriber<>(Integer.MAX_VALUE)); 
        log.info("end main");
        Thread.sleep(1000); 
    }

    private static Mono<Integer> getItems() { 
        return Mono.create(monoSink -> {
            // 값을 하나 넘긴 이후 별도로 complete를 호출하지 않아도 complete가 전달된다.
            // 1이라는 값을 출력 이후 바로 end main
            monoSink.success(1); 
        });
    }
}
```

## Flux를 Mono로 변환

- `Mono.from(publisher)` 으로 Flux를 Mono로 바꿀 수 있다.
- 첫번째 값만 전달하게 된다.

```java
@Slf4j
public class FluxToMonoExample {
    public static void main(String[] args) {
        log.info("start main");
        // from 으로부터 받은 pub의 next를 받으면 바로 success를 실행함
        // 여기서는 1을 반환한다.
        Mono.from(getItems()).subscribe(new SimpleSubscriber<>(Integer.MAX_VALUE));
        log.info("end main"); 
    }

    private static Flux<Integer> getItems() {
        return Flux.fromIterable(List.of(1, 2, 3, 4, 5)); 
    }
}
```

```java
10:46:05 [main] - start main
10:46:06 [main] - subscribe
10:46:06 [main] - request: 2147483647 
10:46:06 [main] - item: 1
10:46:06 [main] - complete
10:46:06 [main] - end main
```

- 만약 첫번째 값만 받고싶은게 아니라 iter의 item들을 전부 받고싶다면?
- `Flux.collectList()`를 활용하자
  - Flux의 값들을 collect 하고 complete 이벤트가 발생하는 시점에 모은 값들을 전달한다.
    - onNext로 오는 아이템들을 전부 받아서 내부 배열에다가 저장을 한다.
    - onComplete가 실행되는 순간 내부 배열 값을 onNext로 전달하는 구조로 되어있다.

```java
@Slf4j
public class FluxToListMonoExample {
    public static void main(String[] args) {
        log.info("start main"); 
        getItems().collectList().subscribe(new SimpleSubscriber<>(Integer.MAX_VALUE));
        log.info("end main");
    }

    private static Flux<Integer> getItems() {
        return Flux.fromIterable(List.of(1, 2, 3, 4, 5)); 
    }
}
```

```java
10:41:41 [main] - start main
10:41:42 [main] - subscribe
10:41:42 [main] - request: 2147483647 
10:41:42 [main] - item: [1, 2, 3, 4, 5] 
10:41:42 [main] - complete
10:41:42 [main] - end main
```

## Mono를 Flux로 변환

- `flux()`를 사용하여 Mono를 next 한번 호출하고 onComplete를 호출하는 Flux로 변환한다.

```java
@Slf4j
public class MonoToFluxExample {
    public static void main(String[] args) {
        log.info("start main");
        // Mono.flux()를 통해 Mono를 하나의 아이템을 전달하고 complete 하는 flux로 변환
        getItems().flux().subscribe(new SimpleSubscriber<>(Integer.MAX_VALUE)); 
        log.info("end main");
    }

    private static Mono<List<Integer>> getItems() {
        return Mono.just(List.of(1, 2, 3, 4, 5)); 
    }
}
```

```
10:52:25 [main] - start main
10:52:25 [main] - subscribe
10:52:25 [main] - request: 2147483647 
10:52:25 [main] - item: [1, 2, 3, 4, 5] 
10:52:25 [main] - complete
10:52:25 [main] - end main
```

- 위에 예제는 list 객체 하나만 전달했지만 만약 list item들을 각각 따로따로 나눠서 여러 flux로 전달하고 싶다면 어떻게해야할까?
- `flatMapMany()`
  - Mono의 값으로 여러개의 값을 전달하는 Flux를 만들고 이를 연결해주는 메서드

```java
@Slf4j
public class ListMonoToFluxExample {
    public static void main(String[] args) {
        log.info("start main");
        // Mono의 결과를 Flux.fromIterable을 통해 각각의 item들을 개별 Flux로 반환
        getItems().flatMapMany(value -> Flux.fromIterable(value)).subscribe(new SimpleSubscriber<>(Integer.MAX_VALUE)); 
        log.info("end main");
    }

    private static Mono<List<Integer!>> getItems() {
        return Mono.just(List.of(1, 2, 3, 4, 5)); 
    }
}
```

```
10:54:52 [main] - start main
10:54:52 [main] - subscribe
10:54:52 [main] - request: 2147483647 
10:54:52 [main] - item: 1
10:54:52 [main] - item: 2
10:54:52 [main] - item: 3
10:54:52 [main] - item: 4
10:54:53 [main] - item: 5
10:54:53 [main] - complete
10:54:53 [main] - end main
```

## 📣 Reference
[패스트캠퍼스 - Spring Webflux 완전정복 : 코루틴부터 리액티브 MSA 프로젝트까지](https://fastcampus.co.kr/dev_online_webflux)<br/>

