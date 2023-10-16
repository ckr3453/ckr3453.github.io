---
title: "Spring Webflux) CompletionStage"
categories: 
    - spring
date: 2023-09-10
last_modified_at: 2023-10-10
toc: true
toc_sticky: true
excerpt: "CompletableFuture, Future, CompletionStage"
---

## CompletionStage

`Future` 인터페이스의 문제점들(연산 결합 불가능, 예외처리 불가능)을 보완할 수 있는 장점들을 가지고 있는 인터페이스이다.

```java
public interface CompletionStage<T> {
 public <U> CompletionStage<U> thenApply(Function<? super T,? extends U> fn);
 public <U> CompletionStage<U> thenApplyAsync(Function<? super T,? extends U> fn);
 
 public CompletionStage<Void> thenAccept(Consumer<? super T> action);
 public CompletionStage<Void> thenAcceptAsync(Consumer<? super T> action);
 
 public CompletionStage<Void> thenRun(Runnable action);
 public CompletionStage<Void> thenRunAsync(Runnable action);
 
 public <U> CompletionStage<U> thenCompose(Function<? super T, ? extends CompletionStage<U > fn);
 public <U> CompletionStage<U> thenComposeAsync(Function<? super T, ? extends CompletionStage<U > fn);
 
 public CompletionStage<T> exceptionally(Function<Throwable, ? extends T> fn);
}
```

`CompletionStage`는 함수형 인터페이스를 지원하기 떄문에 `Future` 와 다르게 각 메서드들의 결과로 파이프 라인 구성을 할 수 있다. 

또한 `exceptionally`를 통해 예외 처리도 가능하다.

## CompletionStage 연산자 조합

연산자들을 활용하여 다음과 같이 비동기 task들을 실행하고 값을 변형하는 등 메서드 체이닝을 이용한 조합이 가능하다.

```java
Helper.completionStage()
            .thenApplyAsync(value -> {
        log.info("thenApplyAsync: {}", value);
        return value + 1;
    }).thenAccept(value -> {
        log.info("thenAccept: {}", value);
    }).thenRunAsync(() -> {
        log.info("thenRun");
    }).exceptionally(e -> {
        log.info("exceptionally: {}", e.getMessage());
        return null;
    });
Thread.sleep(100);
```

비동기 non-blocking 형태의 개발을 할수 있게끔 의도했기 때문(?)인지 직접적으로 값을 조회할 수는 없다.

## ForkJoinPool

<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/f346c841-448c-498a-b2ff-976c01f29433"></center><br/>

`CompletableFuture`는 내부적으로 비동기 함수들을 실행하기 위해 `ForkJoinPool`을 사용한다.

`ForkJoinPool`은 하나의 작업을 thread들이 비동기적으로 처리할 수 있을 정도의 작은 단위로 나눠서 처리하는 분할 정복 방식(divide and conquer)의 thread pool 이다.

- ForkJoinPool의 기본 사이즈는 '할당된 cpu 코어 - 1' 이다.
- ForkJoinPool은 daemon thread 이므로 main thread 가 종료되면 즉각적으로 종료된다. 
- task를 fork를 통해서 subtask로 나누고 -> thread pool에서 steal work 알고리즘을 통해 thread 들이 task들을 균등하게 처리한뒤 -> join을 통해서 결과를 생성한다.

## 예시에 사용할 helper 클래스 생성
```java
@Slf4j
public class Helper {
    // 1을 반환하는 완료된 CompletableFuture 반환
    @SneakyThrows
    public static CompletionStage<Integer> finishedStage() {
        var future = CompletableFuture.supplyAsync(() -> {
            log.info("return in future");
            return 1;
        });
        Thread.sleep(100);
        return future;
    }

    // 1초를 sleep 한 후 1을 반환하는 CompletableFuture
    public static CompletionStage<Integer> runningStage() {
        return CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(1000);
                log.info("I'm running!");
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            return 1;
        });
    }
}
```

## thenAccept[Async]

- `Consumer`를 파라미터로 받는다.
- 이전 task로부터 값을 받지만 값을 넘기지는 않는다.
  - 즉, 다음 task에게 `null`이 전달된다.
- 값을 받아서 특정 action만 수행한뒤 끝내는 경우에 유용하다.

thenAccept[Async]는 작업이 완료된 `CompletionStage`에서의 동작과 작업중인 `CompletionStage`에서의 **동작이 다르다.**

### 작업이 완료된 CompletionStage에서 동작 방식

- **thenAccept**

```java
log.info("start main");
// 작업이 완료된 CompletionStage
CompletionStage<Integer> stage = Helper.finishedStage(); 
stage.thenAccept(i -> {
    log.info("{} in thenAccept", i);  // i는 1 반환
}).thenAccept(i -> {
    log.info("{} in thenAccept2", i); // i는 null 을 반환
});
log.info("after thenAccept");

Thread.sleep(100);
```
```java
[main] - start main
[ForkJoinPool.commonPool-worker-19] - return in future
[main] - 1 in thenAccept
[main] - null in thenAccept2
[main] - after thenAccept
```

- **thenAcceptAsync**

```java
log.info("start main");
// 작업이 완료된 CompletionStage
CompletionStage<Integer> stage = Helper.finishedStage();
stage.thenAcceptAsync(i > {
 log.info("{} in thenAcceptAsync", i); // i는 1 반환
}).thenAcceptAsync(i > {
 log.info("{} in thenAcceptAsync2", i); // i는 null 을 반환
});
log.info("after thenAccept");
Thread.sleep(100)
```
```java
[main] - start main
[ForkJoinPool.commonPool-worker-19] - return in future
[main] - after thenAccept
[ForkJoinPool.commonPool-worker-19] - 1 in thenAcceptAsync
[ForkJoinPool.commonPool-worker-5] - null in thenAcceptAsync2
```

<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/1a08d104-31fe-471a-8c4c-ca2ad7423987"></center><br/>

- (done 상태일 때) `thenAccept` vs `thenAcceptAsync`
  - `thenAccept`는 caller 의 thread에서 실행한다. (blocking)
  - `thenAcceptAsync`는 각각 별도의 thread에서 실행된다. (non-blocking)

### 작업중인 CompletionStage에서 동작 방식

- **thenAccept**

```java
log.info("start main");
// 작업이 진행중인 CompletionStage
CompletionStage<Integer> stage = Helper.runningStage();
// 작업이 완료된 시점에 메서드 실행
stage.thenAccept(i > {
 	log.info("{} in thenAccept", i);
}).thenAccept(i > {
 	log.info("{} in thenAccept2", i);
});
Thread.sleep(2000);
```
```java
[main] INFO - start main
[ForkJoinPool.commonPool-worker-19] INFO - I'm running!
[ForkJoinPool.commonPool-worker-19] INFO - 1 in thenAccept
[ForkJoinPool.commonPool-worker-19] INFO - null in thenAccept2
```

- **thenAcceptAsync**

```java
log.info("start main");
// 작업이 진행중인 CompletionStage
CompletionStage<Integer> stage = Helper.runningStage();
// 작업이 완료된 시점에 메서드 실행
stage.thenAcceptAsync(i > {
	log.info("{} in thenAcceptAsync", i);
}).thenAcceptAsync(i > {
 	log.info("{} in thenAcceptAsync", i);
});
Thread.sleep(2000);
```
```java
[main] INFO - start main
[ForkJoinPool.commonPool-worker-19] INFO - I'm running!
[ForkJoinPool.commonPool-worker-6] INFO - 1 in thenAcceptAsync
[ForkJoinPool.commonPool-worker-6] INFO - null in thenAcceptAsync2
```

<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/f290df70-267b-4a66-9c35-9477e6191d80"></center><br/>

- (done 상태가 아닐때) `thenAccept` vs `thenAcceptAsync`
  - `thenAccept`는 자신을 호출한 callee thread에서 실행한다. (blocking)
  - `thenAcceptAsync`는 각각 별도의 thread에서 실행된다. (non-blocking)

## then*[Async]의 실행 쓰레드

`thenAccept` 뿐만 아니라 then으로 시작하는 연산자들의 실행 쓰레드는 다음과 같다.

<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/4530f8d3-7688-461b-915a-2f5cd9fe810b"></center><br/>

- (Async가 붙지 않은 경우) stage 상태(done 유무)에 따라 caller 혹은 callee 에서 action을 수행한다.
  - 상황에 따라 caller와 callee 를 block 할수 있으므로 **가능한 사용을 자제해야한다.**
- (Async가 붙은 경우) thread pool에 있는 각각 별도의 쓰레드에서 action을 수행한다.

## then*Async의 thread pool 변경하기

- 모든 then*Async 연산자는 executor를 추가 인자로 받는다.
- 이를 통해서 다른 thread pool로 각각의 task를 실행할 수 있다.

```java
// 단일 스레드풀 선언
var single = Executors.newSingleThreadExecutor();

// 10개 고정 스레드풀 선언
var fixed = Executors.newFixedThreadPool(10);

log.info("start main");
CompletionStage<Integer> stage = Helper.completionStage();
stage.thenAcceptAsync(i -> {
  // 10개의 고정 스레드풀 사용
	log.info("{} in thenAcceptAsync", i);
}, fixed).thenAcceptAsync(i -> {
  // 단일 스레드풀 사용
	log.info("{} in thenAcceptAsync2", i);
}, single);
log.info("after thenAccept");
Thread.sleep(200);

single.shutdown();
fixed.shutdown();
```
```java
[main] - start main
[ForkJoinPool.commonPool-worker-19] - return in future
[main] - after thenAccept
[pool-3-thread-1] - 1 in thenAcceptAsync
[pool-2-thread-1] - null in thenAcceptAsync2
```

## thenApply[Async]

- `Function`을 매개변수로 받는다.
- 이전 task로부터 `T` 타입의 값을 받아서 가공하고 `U` 타입의 값을 반환한다.
  - 즉, 다음 task에게 반환했던 값이 전달된다.
- 값을 변형해서 전달해야 하는 경우에 유용하다.

```java
// 1을 반환하는 CompletionStage
CompletionStage<Integer> stage = Helper.completionStage();

stage.thenApplyAsync(value -> {
  var next = value + 1;
  // 1을 더해서 2를 넘김
  return next;
}).thenApplyAsync(value -> {
  var next = "result: " + value;
  // 기존 값 2에 문자열을 더해서 넘김
  return next;
}).thenApplyAsync(value -> {
  var next = value.equals("result: 2");
  // 넘어온 값을 비교하여 boolean 값을 마지막으로 넘김
  return next;
}).thenAcceptAsync(value -> log.info("{}", value)); // thenApply로 더했던 모든 값들을 출력한다.

Thread.sleep(100);
```
```java
[ForkJoinPool.commonPool-worker-19] - return in future
[ForkJoinPool.commonPool-worker-19] - in thenApplyAsync: 2
[ForkJoinPool.commonPool-worker-19] - in thenApplyAsync2: result: 2
[ForkJoinPool.commonPool-worker-19] - in thenApplyAsync3: true
[ForkJoinPool.commonPool-worker-19] - true
```

## thenCompose[Async]

- `Function`을 매개변수로 받는다.
- 이전 task로부터 `T` 타입의 값을 받아서 가공하고 `U` 타입의 `CompletionStage`를 반환한다.
  - 반환한 `CompletionStage`가 done 상태가 되면 그때 값을 다음 task에 전달한다.
  - 즉, `Function`을 조합하여 실행할때 사용 
- 다른 future를 반환해야 하는 경우에 유용하다.

```java
// 100ms 쉬었다가 받은 정수에 1을 더해서 CompletionStage를 반환한다.
public static CompletionStage<Integer> addOne(int value) {
	return CompletableFuture.supplyAsync(() -> {
      try {
	      Thread.sleep(100);
      } catch (InterruptedException e) {
    	  throw new RuntimeException(e);
      }
      return value + 1;
	});
}
```
```java
CompletionStage<Integer> stage = Helper.completionStage();
stage.thenComposeAsync(value -> {
  var next = Helper.addOne(value);
  // 작업이 완료되지 않았기 때문에 [Not completed] 를 출력
  log.info("in thenComposeAsync: {}", next);
  // 작업이 완료된 후 다음 task에 전달
	return next;
}).thenComposeAsync(value -> {
  // 100ms 쉬고 "result :" 를 붙여주는 함수
  var next = Helper.addResultPrefix(value);
  // 작업이 완료되지 않았기 때문에 [Not completed] 를 출력
  log.info("in thenComposeAsync2: {}", next);
  // 작업이 완료된 후 다음 task에 전달
  return next;
}).thenAcceptAsync(value -> {
  // 최종적으로 가공된 값을 받아서 출력
	log.info("{} in thenAcceptAsync", value);
});

Thread.sleep(1000);
```
```java
[main] - start main
[ForkJoinPool.commonPool-worker-19] - I'm running!
[ForkJoinPool.commonPool-worker-19] - in thenComposeAsync: java.util.concurrent.CompletableFuture@37b05857[Not completed]
[ForkJoinPool.commonPool-worker-19] - in thenComposeAsync2: java.util.concurrent.CompletableFuture@6398d2c5[Not completed]
[ForkJoinPool.commonPool-worker-5] - result: 2 in thenAcceptAsync
```

## thenRun[Async]

- `Runnable`를 매개변수로 받는다.
- 이전 task로부터 값을 받지않고 값을 반환하지 않는다.
  - 즉, 다음 task에게 `null`이 전달된다.
- future가 완료되었다는 이벤트를 기록할 때 유용하다.

```java
log.info("start main");
CompletionStage<Integer> stage = Helper.completionStage();
stage.thenRunAsync(() -> {
	log.info("in thenRunAsync");
}).thenRunAsync(() -> {
	log.info("in thenRunAsync2");
}).thenAcceptAsync(value -> {
  // 반환 받는 값이 null이기 때문에 null 출력
	log.info("{} in thenAcceptAsync", value);
});

Thread.sleep(100);
```
```java
[main] - start main
[ForkJoinPool.commonPool-worker-19] - return in future
[ForkJoinPool.commonPool-worker-19] - in thenRunAsync
[ForkJoinPool.commonPool-worker-19] - in thenRunAsync2
[ForkJoinPool.commonPool-worker-19] - null in thenAcceptAsync
```

## exceptionally

- `Function(Throwable)` 을 매개변수로 받는다.
- 이전 task에서 발생한 `exception`을 받아서 처리하고 값을 반환한다.
  - 다음 task에게 반환된 값을 전달한다.
- future 파이프에서 발생한 에러를 처리할때 유용하다.

```java
Helper.completionStage()
  .thenApplyAsync(i -> {
		log.info("in thenApplyAsync");
    // ArithmeticException을 강제로 발생 시킴
		return i / 0;
	}).exceptionally(e -> {
    // exception 을 받아서 메세지를 출력
    log.info("{} in exceptionally", e.getMessage());
    // 값은 0을 반환
    return 0;
	}).thenAcceptAsync(value -> {
		log.info("{} in thenAcceptAsync", value);
	});
    
Thread.sleep(1000);
```
```java
[ForkJoinPool.commonPool-worker-19] - return in future
[ForkJoinPool.commonPool-worker-5] - in thenApplyAsync
[ForkJoinPool.commonPool-worker-5] - java.lang.ArithmeticException: / by zero in exceptionally
[ForkJoinPool.commonPool-worker-5] - 0 in thenAcceptAsync
```

## 📣 Reference
[패스트캠퍼스 - Spring Webflux 완전정복 : 코루틴부터 리액티브 MSA 프로젝트까지](https://fastcampus.co.kr/dev_online_webflux)<br/>

