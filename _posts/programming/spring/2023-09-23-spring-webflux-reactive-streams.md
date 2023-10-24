---
title: "Spring Webflux) Reactive Streams"
categories: 
    - spring
date: 2023-09-23
last_modified_at: 2023-10-24
toc: true
toc_sticky: true
excerpt: "Reactive Streams"
---

## Reactive streams

<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/4ac92c11-996a-447d-9063-e1f9d62fae6e"></center>

Reactive하게 개발하는 쉬운 방법중 하나는 Reactive streams를 활용하는 방법이다. 

Reactive streams는 `Publisher(이하 pub)`, `Subscriber(이하 sub)`, `Subscription(이하 subp)`으로 구성되어있다.

- 데이터 혹은 이벤트를 제공하는 `Publisher`
- 데이터 혹은 이벤트를 제공받는 `Subscriber`
- 데이터 흐름을 조절하는 `Subscription`
  - `Subscriber`가 `Publisher`에게 요청하는 일종의 리모콘인 셈이다.

`Publisher`이 `Subscriber`에게 `onSubscribe`를 통해 `Subscription`를 전달하면 `Subscriber`은 `Subscription`을 통해 `Publisher`에게 데이터 요청 수, 요청취소여부 등을 전달할 수 있다. 


## Publisher

```java
@FunctionalInterface
public static interface Publisher<T> {
  public void subscribe(Subscriber<? super T> subscriber);
}
```

- subscribe 함수를 제공해서 pub 에 다수의 sub를 등록할 수 있게끔 지원한다.
  - sub가 추가되면 subp를 제공한다.

## Subscriber

```java
public static interface Subscriber<T> {
  public void onSubscribe(Subscription subscription);
  public void onNext(T item);
  public void onError(Throwable throwable);
  public void onComplete();
}
```

- subscribe 하는 시점에 pub으로부터 subp를 받을수 있는 인자를 제공한다.
  - sub은 subp를 활용하여 pub에게 n개의 요청 혹은 취소 요청을 할수 있다.
- onNext, onError, onComplete를 통해서 값이나 이벤트를 받을 수 있다.
- onNext는 여러번, onError나 onComplete는 딱 한번만 호출된다.

# Subscription

```java
public static interface Subscription {
  public void request(long n);
  public void cancel();
}
```

- back-pressure를 조절할 수 있는 request 함수를 제공한다.
- pub이 onNext를 통해서 sub에게 값을 전달하는 것을 취소할 수 있는 cancel 함수를 제공한다.

## Cold / Hot Publisher

- Hot Publisher
  - sub가 없더라도 데이터를 생성하고 stream에 push 하는 pub를 의미한다. (능동적)
  - 트위터 게시글 읽기, 공유 리소스 변화, 외부의 변화에 의해서 실시간으로 바뀌는 데이터 등에 사용한다.
  - 여러 sub에게 동일한 데이터를 전달한다.

- Cold Publisher
  - subscribe가 시작되는 순간부터 데이터를 생성하고 전송한다. (수동적)
  - 파일 읽기 웹 API 요청 등에 사용한다.
  - sub에 따라 독립적인 데이터 스트림을 제공한다.

### Cold Publisher 예제 구현해보기

- SimpleColdPublisher

```java
// Flow.Publisher 를 구현
public class SimpleColdPublisher implements Flow.Publisher<Integer> {
    @Override
    public void subscribe(Flow.Subscriber<? super Integer> subscriber) {
        // sub가 pub를 subscribe를 하게되면 1~10 까지 숫자를 iter로 생성하여 subp 생성 및 전달
        var iterator = Collections.synchronizedList(IntStream.range(1, 10).boxed().collect(Collectors.toList())).iterator();
        var subscription = new SimpleColdSubscription(iterator, subscriber);
        subscriber.onSubscribe(subscription);
    }

    // Flow.Subscription 를 구현
    @RequiredArgsConstructor
    public class SimpleColdSubscription implements Flow.Subscription {
        private final Iterator<Integer> iterator;
        private final Flow.Subscriber<? super Integer> subscriber;
        private final ExecutorService executorService = Executors.newSingleThreadExecutor();

        @Override
        public void request(long n) {
            // subp로써 request를 받으면 별도 thread로 구성
            // 별도 thread로 구성하는 이유는 sub의 onNext와 subp의 request가 동일한 thread 내에서 동기적으로 동작하면 안되기 때문
            executorService.submit(() -> {
                for (int i = 0; i < n; i++) {
                    if (iterator.hasNext()) {
                        var number = iterator.next();
                        iterator.remove();
                        subscriber.onNext(number);
                    } else {
                        subscriber.onComplete();
                        executorService.shutdown();
                        break;
                    }
                }
            });
        }

        @Override
        public void cancel() {
            subscriber.onComplete();
        }
    }
}
```

- cold publisher 예제 실행

```java
public class SimpleColdPublisherMain {
    @SneakyThrows
    public static void main(String[] args) {
        // 예제용 cold pub 생성
        var publisher = new SimpleColdPublisher();

        // SimpleNamedSubscriber 를 통해 각 이벤트 마다 로깅 수행과 이름을 가지는 sub 를 생성
        var subscriber = new SimpleNamedSubscriber<Integer>("subscriber1");
        publisher.subscribe(subscriber);

        // 5초 sleep 이후 진행
        Thread.sleep(5000);

        // SimpleNamedSubscriber 를 통해 각 이벤트 마다 로깅 수행과 이름을 가지는 sub 를 생성
        var subscriber2 = new SimpleNamedSubscriber<Integer>("subscriber2");
        publisher.subscribe(subscriber2);
    }
}
```

- 결과

```java
21:25:33 [main] - onSubscribe
21:25:33 [pool-2-thread-1] - name: subscriber1, onNext: 1
21:25:33 [pool-2-thread-1] - name: subscriber1, onNext: 2
21:25:33 [pool-2-thread-1] - name: subscriber1, onNext: 3
21:25:33 [pool-2-thread-1] - name: subscriber1, onNext: 4
21:25:33 [pool-2-thread-1] - name: subscriber1, onNext: 5
21:25:33 [pool-2-thread-1] - name: subscriber1, onNext: 6
21:25:33 [pool-2-thread-1] - name: subscriber1, onNext: 7
21:25:33 [pool-2-thread-1] - name: subscriber1, onNext: 8
21:25:33 [pool-2-thread-1] - name: subscriber1, onNext: 9
21:25:33 [pool-2-thread-1] - onComplete
21:25:38 [main] - onSubscribe
21:25:38 [pool-3-thread-1] - name: subscriber2, onNext: 1
21:25:38 [pool-3-thread-1] - name: subscriber2, onNext: 2
21:25:38 [pool-3-thread-1] - name: subscriber2, onNext: 3
21:25:38 [pool-3-thread-1] - name: subscriber2, onNext: 4
21:25:38 [pool-3-thread-1] - name: subscriber2, onNext: 5
21:25:38 [pool-3-thread-1] - name: subscriber2, onNext: 6
21:25:38 [pool-3-thread-1] - name: subscriber2, onNext: 7
21:25:38 [pool-3-thread-1] - name: subscriber2, onNext: 8
21:25:38 [pool-3-thread-1] - name: subscriber2, onNext: 9
21:25:38 [pool-3-thread-1] - onComplete
```

- sub가 subcribe를 한 순간부터 1~10 까지의 데이터를 만들고 전송하는 것을 확인할 수 있다.

### Hot Publisher 예제 구현해보기

- SimpleHotPublisher

```java
// Flow.Publisher 를 구현
@Slf4j
public class SimpleHotPublisher implements Flow.Publisher<Integer> {
    private final ExecutorService publisherExecutor = Executors.newSingleThreadExecutor();
    private final Future<Void> task;
    private List<Integer> numbers = new ArrayList<>();
    private List<SimpleHotSubscription> subscriptions = new ArrayList<>();

    // SimpleHotPublisher 객체가 생성될때 최초로 1을 넣어주고 thread가 중단될 때 까지 계속해서 값을 추가한다.
    // pub는 값을 계속해서 생성하지만 sub는 subscribe 하는 시점부터 값을 받아갈수 있다.
    public SimpleHotPublisher() {
        numbers.add(1);
        task = publisherExecutor.submit(() -> {
            for(int i = 2;!Thread.interrupted();i++) {
                numbers.add(i);
                // 저장된 subp 리스트들을 대상으로 sub에게 전송할 값이 있다면 전달하는 메서드 실행
                subscriptions.forEach(SimpleHotSubscription::wakeup);
                Thread.sleep(100);
            }

            return null;
        });
    }

    public void shutdown() {
        this.task.cancel(true);
        publisherExecutor.shutdown();
    }

    // subp를 생성하여 sub에게 전달하고 내부 리스트에 저장함
    @Override
    public void subscribe(Flow.Subscriber<? super Integer> subscriber) {
        var subscription = new SimpleHotSubscription(subscriber);
        subscriber.onSubscribe(subscription);
        // 각 subp을 통해 전달하기 위해 내부 리스트로 저장
        subscriptions.add(subscription);
    }

    // Flow.Subscription 를 구현
    private class SimpleHotSubscription implements Flow.Subscription {
        // pub가 생성한 데이터의 마지막 인덱스
        private int offset;

        // sub가 subp를 통해 pub에게 요청한 데이터 인덱스
        // 아직 pub가 데이터를 만들지 못했는데 만약 sub로부터 그 이상의 데이터 요청이 들어올때를 대비하여 요청한 갯수를 저장
        private int requiredOffset;

        private final Flow.Subscriber<? super Integer> subscriber;
        private final ExecutorService subscriptionExecutorService = Executors.newSingleThreadExecutor();

        // 
        public SimpleHotSubscription(Flow.Subscriber<? super Integer> subscriber) {
            // pub numbers의 마지막 인덱스를 각각 저장하여 sub의 데이터 요청에 대응
            int lastElementIndex = numbers.size() - 1;
            this.offset = lastElementIndex;
            this.requiredOffset = lastElementIndex;
            this.subscriber = subscriber;
        }

        // 요청한 갯수(인덱스)만큼 requiredOffset를 확장
        @Override
        public void request(long n) {
            requiredOffset += n;
            onNextWhilePossible();
        }

        // 값이 생길때마다 sub에게 값을 전달하는 역할을 하는 메서드
        public void wakeup() {
            onNextWhilePossible();
        }

        @Override
        public void cancel() {
            this.subscriber.onComplete();
            if (subscriptions.contains(this)) {
                subscriptions.remove(this);
            }
            subscriptionExecutorService.shutdown();
        }

        // 값이 생길때마다 sub에게 값을 전달하는 역할을 하는 메서드
        // 만약 sub가 요청한 데이터가 pub가 전송한 데이터에 비해 많다면 요청한 데이터를 전부 전송할때 까지 sub의 onNext를 통해 전달 
        private void onNextWhilePossible() {
            subscriptionExecutorService.submit(() -> {
                while (offset < requiredOffset && offset < numbers.size()) {
                    var item = numbers.get(offset);
                    subscriber.onNext(item);
                    offset++;
                }
            });
        }
    }
}
```

- hot publisher 예제 실행

```java
@Slf4j
public class SimpleHotPublisherMain {
    @SneakyThrows
    public static void main(String[] args) {
        // 예제용 hot pub 객체 생성
        var publisher = new SimpleHotPublisher();

        // subscriber1이 pub을 구독
        var subscriber = new SimpleNamedSubscriber<>("subscriber1");
        publisher.subscribe(subscriber);

        // 5초이후 subscriber1 구독 중단
        Thread.sleep(5000);
        subscriber.cancel();

        // subscriber2과 subscribe3이 pub을 구독
        var subscriber2 = new SimpleNamedSubscriber<>("subscriber2");
        var subscriber3 = new SimpleNamedSubscriber<>("subscriber3");
        publisher.subscribe(subscriber2);
        publisher.subscribe(subscriber3);

        // 5초 이후 subscriber2, subscriber3 구독 중단
        Thread.sleep(5000);
        subscriber2.cancel();
        subscriber3.cancel();


        Thread.sleep(1000);

        // subscriber4가 pub을 구독
        var subscriber4 = new SimpleNamedSubscriber<>("subscriber4");
        publisher.subscribe(subscriber4);

        // 5초 이후 subscriber4 구독 중단
        Thread.sleep(5000);
        subscriber4.cancel();

        // 최종적으로 pub 객체의 task와 스레드를 종료
        publisher.shutdown();
    }
}
```

```java
21:26:33 [main] - onSubscribe
21:26:33 [pool-2-thread-1] - name: subscriber1, onNext: 2
21:26:33 [pool-2-thread-1] - name: subscriber1, onNext: 3
21:26:33 [pool-2-thread-1] - name: subscriber1, onNext: 4
21:26:33 [pool-2-thread-1] - name: subscriber1, onNext: 5
21:26:33 [pool-2-thread-1] - name: subscriber1, onNext: 6
21:26:33 [pool-2-thread-1] - name: subscriber1, onNext: 7
...
..
21:26:35 [pool-2-thread-1] - name: subscriber1, onNext: 49
21:26:36 [pool-2-thread-1] - name: subscriber1, onNext: 50
21:26:36 [main] - cancel
21:26:36 [main] - onComplete
21:26:37 [main] - onSubscribe
21:26:38 [pool-4-thread-1] - name: subscriber2, onNext: 50
21:26:38 [pool-5-thread-1] - name: subscriber3, onNext: 51
21:26:38 [pool-4-thread-1] - name: subscriber2, onNext: 52
21:26:38 [pool-5-thread-1] - name: subscriber3, onNext: 52
...
..
21:26:45 [pool-4-thread-1] - name: subscriber2, onNext: 98
21:26:45 [pool-5-thread-1] - name: subscriber3, onNext: 98
21:26:45 [main] - cancel
21:26:46 [main] - onComplete
21:26:46 [main] - cancel
21:26:46 [main] - onComplete
21:26:47 [main] - onSubscribe
21:26:48 [pool-3-thread-1] - name: subscriber4, onNext: 108
21:26:48 [pool-3-thread-1] - name: subscriber4, onNext: 109
21:26:48 [pool-3-thread-1] - name: subscriber4, onNext: 110
...
..
21:26:54 [pool-3-thread-1] - name: subscriber4, onNext: 157
21:26:54 [main] - cancel
21:26:54 [main] - onComplete
```

- 예제를통해 hot pub은 sub가 subcribe 하는 순간마다 데이터 다르다는것을 알수 있다.


## 📣 Reference
[패스트캠퍼스 - Spring Webflux 완전정복 : 코루틴부터 리액티브 MSA 프로젝트까지](https://fastcampus.co.kr/dev_online_webflux)<br/>

