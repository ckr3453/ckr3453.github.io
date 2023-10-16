---
title: "Spring Webflux) Future"
categories: 
    - spring
date: 2023-10-10
last_modified_at: 2023-10-10
toc: true
toc_sticky: true
excerpt: "CompletableFuture, Future, CompletionStage"
---

## CompletableFuture

<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/728e1555-2afb-4df5-890f-2ebe2b72ed21"></center>

- 2014년, Java 8에서 처음으로 도입
- 비동기 프로그래밍을 지원하는 클래스
- `Lambda`, `Method reference` 등 Java 8의 새로운 기능들을 지원
- `Future` 인터페이스와 `CompletionStage` 인터페이스를 구현한 클래스
  - `Future` 인터페이스의 주요 역할
    - 비동기적인 작업을 수행
    - 해당 작업이 완료되면 결과를 반환하는 인터페이스
  - `CompletionStage` 인터페이스의 주요 역할
    - 비동기적인 작업을 수행
    - 해당작업이 완료되면 결과를 처리하거나 다른 `CompletionStage`를 연결하는 인터페이스, 즉 콜백을 여러개 등록할 수 있다.

## ExecutorService

`Executor` 인터페이스를 상속받았으며 쓰레드 풀을 이용하여 비동기적으로 작업을 실행하고 관리할수 있게끔 지원하는 인터페이스이다.

```java
// Executors 클래스에서 제공하는 Factory method를 사용하여 구현 가능

// 1개로 고정된 사이즈의 단일 ThreadPool 생성
ExecutorService executorService = Executors.newSingleThreadExecutor();

// 10개로 고정된 사이즈의 ThreadPool 생성
ExecutorService executorService = Executors.newFixedThreadPool(10);

// 유동적으로 증가하고 줄어드는 ThreadPool 생성
// 사용 가능한 쓰레드가 없다면 새로 생성해서 작업을 처리하고, 쓰레드가 있다면 재사용한다.
// 쓰레드가 일정시간 사용되지 않으면 회수한다.
ExecutorService executorService = Executors.newCachedThreadPool();

// 10개로 고정된 사이즈의 스케줄링 ThreadPool 생성
// 주기적이거나 지연이 발생하는 작업들을 실행한다.
ExecutorService executorService = Executors.newScheduledThreadPool(10);

// work steal 알고리즘을 사용하는 ForkJoinPool 생성
ExecutorService executorService = Executors.newWorkStealingPool();

```

## Future 주요 메서드

```java
public interface Future<V> {

    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
}
```

## Future 예시

### 예시에 사용할 helper 클래스 생성

```java
public class FutureHelper {
  // 새로운 쓰레드를 생성하여 1을 반환하는 메서드
  public static Future<Integer> getFuture() {
    var executor = Executors.newSingleThreadExecutor();
    try {
      return executor.submit(() -> {
        return 1;
      })
    } finally {
      executor.shutdown();
    }
  }

  // 새로운 쓰레드를 생성하고 1초 대기후 1을 반환하는 메서드
  public static Future<Integer> getFutureCompleteAfter1s() {
    var executor = Executors.newSingleThreadExecutor();

    try {
      return executor.submit(() -> {
        Thread.sleep(1000);
        return 1;
      });
    } finally {
      executor.shutdown();
    }
  }
}
```

### Future.get()

- 결과를 구할때까지 thread가 계속 block 상태로 유지된다.
- 즉, future에서 무한 루프나 오랜시간이 걸린다면 thread가 blocking 상태를 유지한다.

```java
// 비록 결과를 바로 반환 하더라도 상태를 바로 호출했기 때문에 아직 끝나지 않은 상태
Future future = FutureHelper.getFuture();
assert !future.isDone();
assert !future.isCancelled();

// future에 결과값이 담길때 가져오므로 끝난 상태
var result = future.get();
assert result.equals(1);
assert future.isDone();
assert !future.isCancelled();
```

- 단, `get()`에 매개변수인 timeout 값을 넣어서 호출하게되면 timeout 시간 동안만 thread가 block이 되고 timeout 동안 응답이 반환되지 않을 시 `TimeoutException` 발생

```java 
Future future = FutureHelper.getFutureCompleteAfter1s();
var result = future.get(1500, TimeUnit.MILLISECONDS);
assert result.equals(1);

Future futureToTimeout = FutureHelper.getFutureCompleteAfter1s();
Exception exception = null;

try {
  // 1초 sleep 상태이므로 0.5초 내로 결과를 반환받지 못함
  futureToTimeout.get(500, TimeUnit.MILLISECONDS);
} catch (TimeoutException e) {
  exception = e;
}
assert exception != null;
```

### Future.cancel()

- future의 작업 실행을 취소한다
- 취소할 수 없는 상황이라면 `false`를 반환한다.
- 매개 변수인 `mayInterruptIfRunning`이 `false`인 경우 시작하지 않은 작업에 대해서만 취소한다.

```java
Future future = FutureHelper.getFuture();
// 작업이 시작하기전에 cancel
var successToCancel = future.cancel(true);
assert future.isCancelled();
assert future.isDone();
// cancel에 의해서 취소됨
assert successToCancel;

// 이미 취소가 된 작업을 또 취소
successToCancel = future.cancel(false);
assert future.isCancelled();
assert future.isDone();
// cancel를 통해 취소된게 아니기 때문에 false 반환
assert !successToCancel;
```

## Future 인터페이스의 한계

- cancel을 제외하고 외부에서 future를 컨트롤 할 수 있는 방법이 없다.
- 반환된 결과를 `get()`으로 접근하기 때문에 비동기 처리가 어렵다. 
  - `get()`은 결과가 나올때까지 block 상태를 유지한다.
- 완료되거나 에러가 발생했는지 구분하기 어렵다.
  - 상태를 알수있는 메서드는 `isDone()`, `isCancelled()` 밖에 없으므로 구분할수 없다.

## 📣 Reference
[패스트캠퍼스 - Spring Webflux 완전정복 : 코루틴부터 리액티브 MSA 프로젝트까지](https://fastcampus.co.kr/dev_online_webflux)<br/>

