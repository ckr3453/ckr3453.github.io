---
title: "Spring Webflux) CompletableFuture"
categories: 
    - spring
date: 2023-09-13
last_modified_at: 2023-09-13
toc: true
toc_sticky: true
excerpt: "CompletableFuture, Future, CompletionStage"
---

## CompletableFuture

`CompletableFuture`는 `Future` 와 `CompletionStage`를 구현한 클래스이다.

`Future`에서는 결과값을 반환받기 위해 block 상태를 유지해야 했지만 `CompletableFuture`는 결과를 받기까지 기다리지 않고 callback을 통해 처리할 수 있게된다. 또한 thread pool 사용을 위해 명시적으로 executor를 사용하지 않아도 비동기적으로 작업할 수 있다.

```java
public class CompletableFuture<T> implements Future<T>, CompletionStage<T> {
  public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier) { … }
  public static CompletableFuture<Void> runAsync(Runnable runnable) { … }
  
  public boolean complete(T value) { … }
  public boolean isCompletedExceptionally() { … }
  
  public static CompletableFuture<Void> allOf(CompletableFuture<?>... cfs) { … }
  public static CompletableFuture<Object> anyOf(CompletableFuture<?>... cfs) { … }
}
```


## supplyAsync vs runAsync

- **supplyAsync**
  - `Supplier`를 제공하여 `CompletableFuture`를 생성할 수 있다.
  - `Supplier`의 반환값이 `CompletableFuture`의 결과로 내려가게 되어 메서드 체이닝이 가능하다.


```java
var future = CompletableFuture.supplyAsync(() -> {
  try {
		Thread.sleep(100);
	} catch (InterruptedException e) {
		throw new RuntimeException(e);
	}
	return 1;
});

// non-blocking 하게 동작하므로 바로 상태를 확인하면 false를 반환함
assert !future.isDone();

Thread.sleep(1000);

assert future.isDone();
// CompletableFuture의 결과값을 반환한다.
assert future.get() == 1;
```

- **runAsync**
  - Runnable을 제공하여 CompletableFuture를 생성할 수 있다.
  - 값을 반환하지 않는다.
    - 다음 task에 null을 전달한다.


```java
var future = CompletableFuture.runAsync(() -> {
	try {
		Thread.sleep(100);
	} catch (InterruptedException e) {
		throw new RuntimeException(e);
	}
});

// non-blocking 하게 동작하므로 바로 상태를 확인하면 false를 반환함
assert !future.isDone();

Thread.sleep(1000);

assert future.isDone();
// 결과값을 반환하지 않는다 (null)
assert future.get() == null;
```

future가 종료되면 `supplyAsync`는 콜백함수의 반환값 1을 가져오는 반면 `runAsync`는 null을 반환한다.

## complete

- CompletableFuture가 완료되지 않았다면 매개변수의 값으로 대신하여 완료한다.
  - 만약 이미 완료 되었다면 기존 값을 유지한다.
- complete에 의해 상태가 바뀌었다면 true, 아니라면 false를 반환한다.

```java
CompletableFuture<Integer> future = new CompletableFuture<>();
assert !future.isDone();

// 1이라는 값을 넣어 명시적으로 완료함
var triggered = future.complete(1);
assert future.isDone();
// complete로 인하여 상태가 바뀌었으므로 true
assert triggered;
assert future.get() == 1;

// 2이라는 값을 넣어 명시적으로 완료함
triggered = future.complete(2);
assert future.isDone();
// 이미 위에서 complete(1)을 통해 완료 되었으므로 상태는 바뀌지않음. false
assert !triggered;
// 기존 값유지
assert future.get() == 1;
```

## Future와 CompletableFuture의 상태 비교

- **Future의 상태**
  - isDone(), isCanceled()로만 상태를 확인 할 수 있어서 exception 여부를 확인할 수 없다. (완료, 취소 여부만 확인가능)

<center><img width="352" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/090ca4f9-587b-490f-abbd-a8a43ec49b6b"></center><br/>


- **CompletableFuture의 상태**
  - `isCompletedExceptionally`를 통해 exception 여부를 확인할 수 있다. (완료, 예외가 발생하여 완료, 취소)

<center><img width="343" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/d3bffa21-807f-4fef-b2fd-658df99e4006"></center><br/>

## isCompletedExceptionally

- exception에 의해서 complete 되었는지 확인할 수 있다.

```java
var futureWithException = CompletableFuture.supplyAsync(() -> {
	return 1 / 0;
});

Thread.sleep(100);
// 어쨌든 완료가 되었기 때문에 true
assert futureWithException.isDone();
// 예외가 발생하여 완료되었기 때문에 true
assert futureWithException.isCompletedExceptionally();
```

## allOf

- `CompletableFuture`는 기본적으로 전부 다른 thread에서 동작한다.
- 만약 서로 다른 `CompletableFuture`들을 여러개 만들고 동시에 실행시킨 뒤 결과를 한 thread에서 취합해야 한다면?
- `allOf`는 여러 `CompletableFuture`를 모아서 하나의 `CompletableFuture`로 변환할 수 있다.
- 모든 `CompletableFuture`가 완료되면 상태가 done 으로 변경된다.
  - 즉 allOf 이후의 `CompletableFuture`들은 전부 완료가 보장됨
- return 타입이 void 이므로 각각의 `CompletableFuture`에 get()으로 접근해야한다.

- 예시

```java
var startTime = System.currentTimeMillis();
// Helper.waitAndReturn(int ms, int value) -> ms 만큼 sleep 했다가 value를 return하는 예제용 함수
var firstFuture = Helper.waitAndReturn(100, 1);
var secondFuture = Helper.waitAndReturn(500, 2);
var thirdFuture = Helper.waitAndReturn(1000, 3);

CompletableFuture.allOf(firstFuture, secondFuture, thirdFuture)
	.thenAcceptAsync(v -> {
    // allOf로 인해 firstFuture, secondFuture, thirdFuture는 전부 완료된 상태로 내려옴
		log.info("after allOf");
		try {
          log.info("first: {}", firstFuture.get());
          log.info("second: {}", secondFuture.get());
          log.info("third: {}", thirdFuture.get());
		} catch (Exception e) {
			throw new RuntimeException(e);
		}
}).join();

var endTime = System.currentTimeMillis();
```

```java
11:18 [ForkJoinPool.commonPool-worker-5] - waitAndReturn: 500ms
11:18 [ForkJoinPool.commonPool-worker-23] - waitAndReturn: 1000ms
11:18 [ForkJoinPool.commonPool-worker-19] - waitAndReturn: 100ms
11:19 [ForkJoinPool.commonPool-worker-5] - after allOf
11:19 [ForkJoinPool.commonPool-worker-5] - first: 1
11:19 [ForkJoinPool.commonPool-worker-5] - second: 2
11:19 [ForkJoinPool.commonPool-worker-5] - third: 3
11:19 [main] - elapsed: 1014ms
```

## anyOf

- `allOf`와 마찬가지로 여러 `CompletableFuture`를 모아서 하나의 `CompletableFuture`로 변환할 수 있다.
- 단, 주어진 future중 하나라도 완료되면 상태가 done으로 변경된다.
- 제일 먼저 done 상태가 되는 future의 값을 반환한다.

```java
var startTime = System.currentTimeMillis();
var firstFuture = Helper.waitAndReturn(100, 1);
var secondFuture = Helper.waitAndReturn(500, 2);
var thirdFuture = Helper.waitAndReturn(1000, 3);
CompletableFuture.anyOf(firstFuture, secondFuture, thirdFuture)
	.thenAcceptAsync(v -> {
    // anyOf로 인해 가장먼저 작업이 완료된(100ms) firstFuture의 값을 반환한다.
		log.info("after anyOf");
		log.info("first value: {}", v);
	}).join();
    
    
var endTime = System.currentTimeMillis();
log.info("elapsed: {}ms", endTime - startTime);
```

```java
14:18 [ForkJoinPool.commonPool-worker-23] - waitAndReturn: 1000ms
14:18 [ForkJoinPool.commonPool-worker-19] - waitAndReturn: 500ms
14:18 [ForkJoinPool.commonPool-worker-5] - waitAndReturn: 100ms
14:18 [ForkJoinPool.commonPool-worker-9] - after anyOf
14:18 [ForkJoinPool.commonPool-worker-9] - first value: 1
14:18 [main] - elapsed: 114ms
```

## CompletableFuture의 한계

- 지연로딩 기능을 제공하지 않는다.
  - `CompletableFuture`를 반환하는 함수를 호출시 즉시 작업이 실행된다.
  - thread sleep 후 반환하면 되지않나? -> thread sleep 자체도 다른 thread를 통해 진행되기 때문에 지연로딩이 되었다고 보기 어렵다.
- 지속적으로 생성되는 데이터를 처리하기 어렵다.
  - `CompletableFuture`에서 데이터를 반환하고 나면 다시 다른값을 전달하기 어렵다.
  - `CompletableFuture`는 무조건 값을 한번에 모아서 내려주기 때문에 비동기적으로 중간중간 값을 내리는 경우 대응이 어렵다.


## 📣 Reference
[패스트캠퍼스 - Spring Webflux 완전정복 : 코루틴부터 리액티브 MSA 프로젝트까지](https://fastcampus.co.kr/dev_online_webflux)<br/>

