---
title: "Spring Webflux) Java NIO"
categories: 
    - spring
date: 2023-09-25
last_modified_at: 2023-10-24
toc: true
toc_sticky: true
excerpt: "Java New Input/Output"
---

## Java NIO

<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/69e69345-87f4-47a8-8dcb-aa145f1543f6"></center>

- Java New Input/Ouput 이라는 의미
- Java 1.4 에서 처음으로 도입되었다.
- Java IO와 마찬가지로 파일과 네트워크에 데이터를 읽고 쓸 수 있는 API를 제공한다.
- Java IO는 byte 기반이었으나 NIO는 buffer 기반이다.
- non-blocking을 지원한다.
  - non-blocking을 지원하지 않는 일부 API가 존재한다.
- selector, channel의 도입으로 높은 성능을 보장한다.

## Channel과 Buffer

<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/e00e41a6-af8c-48fc-aa7b-e1a27e7569ef"></center>

- 데이터를 읽을 때
  - 적절한 크기의 Buffer를 생성하고 Channel의 `read()` 메서드를 사용하여 데이터를 Buffer에 저장한다.
- 데이터를 쓸 때
  - 먼저 Buffer에 데이터를 저장하고 Channel의 `write()` 메서드를 사용하여 목적지로 전달한다.
- Buffer는 `clear()` 메서드로 초기화 하여 다시 사용 가능하다.

## Buffer

### Buffer의 종류

<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/19dd2ac5-a832-4ba4-b578-ab24ae7c020c"></center>

- ByteBuffer (byte 단위로 데이터 read/write)
- CharBuffer (char)
- ShortBuffer (short)
- IntBuffer (int)
- LongBuffer (long)
- FloatBuffer (float)
- DoubleBuffer (double)

### Buffer 위치 속성

- capacity
  - Buffer가 저장할수 있는 데이터의 최대 크기를 뜻한다.
  - Buffer 생성시 결정되며 변경이 불가하다.
  - ex. capacity가 1kb에 해당하는 buffer를 저장하면 1kb 밖에 저장못한다.
- position
  - Buffer에서 현재 위치를 가리킨다.
  - Buffer에서 데이터를 읽거나 쓸 때, 해당 위치부터 시작한다.
  - Buffer에 1byte가 추가될 때마다 1씩 증가한다.
- limit
  - Buffer에서 데이터를 읽거나 쓸 수 있는 마지막 위치이다.
  - limit 이후로는 데이터를 읽거나 쓰기가 불가하다.
  - 최초 생성시 capacity와 동일한 값을 가진다.
- mark
  - 현재 position 위치를 `mark()`로 지정할 수 있다.
  - `reset()` 호출 시 position을 mark로 이동한다.

- 각 속성별 가질수 있는 크기 비교
  - 0 <= mark <= position <= limit() <= capacity

### Buffer 위치 초기상태 예시

<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/a7f5dbfc-3b6a-4758-b34a-55c739dbd491"></center>

- capacity는 초기 주어진 값으로 세팅됨
- limit은 capacity와 동일
- position은 0
  - 데이터를 읽거나 쓸때마다 증가하며 limit, capacity를 넘을 수 없다.

### flip

<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/1920fa66-dc16-4cc8-9157-3e89a2f44c7a"></center>

- flip은 Buffer의 limit 위치를 현재 position 위치로 이동시키고 position을 0으로 리셋시킨다.
- Buffer를 쓰기 모드에서 읽기모드로 전환하는 경우에 사용한다.
  - ex. 파일로부터 buffer를 읽어 들여서 한 문자당 1byte씩 100개의 문자를 읽어들인다.
  - buffer에 작성된 문자를 읽기위해 flip을 사용하여 position을 0으로 초기화하고 마지막으로 작성된 문자열 위치를 limit으로 둔다.

### rewind

<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/7c4b247c-40c2-4f6b-abd4-600e3e1a8f14"></center>

- rewind는 Buffer의 position 위치를 0으로 리셋하되, **limit은 유지한다.**
- buffer에 있는 내용을 처음부터 다시 읽는 경우에 사용한다.

### clear

<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/b75d2c6d-9d16-4795-b5ca-f044c397f24e"></center>

- clear는 buffer의 limit 위치를 capacity 위치로 이동시키고, position을 0으로 리셋한다.
- buffer안의 내용을 초기화 할때 사용한다.

### Buffer 예시

```java
try (var fileChannel = FileChannel.open(file.toPath())) {
    var byteBuffer = ByteBuffer.allocateDirect(1024); 
    // buffer 별 position, limit, capacity 로그 출력하는 메서드
    logPosition("allocate", byteBuffer);
    
    // file로부터 값을 읽어서 byteBuffer에 write 
    fileChannel.read(byteBuffer); 
    logPosition("write", byteBuffer);
    
    // flip()을 호출하여 읽기모드로 전환     
    byteBuffer.flip(); 
    logPosition("flip1", byteBuffer);
    
    // 읽기모드로 전환하여 처음부터 limit(마지막까지 write한 위치까지)까지 읽음 
    var result = StandardCharsets.UTF_8.decode(byteBuffer); 
    log.info("result: {}", result);
    logPosition("read1", byteBuffer);

    byteBuffer.rewind(); 
    logPosition("rewind", byteBuffer);

    var result2 = StandardCharsets.UTF_8.decode(byteBuffer); 
    log.info("result2: {}", result2);
    logPosition("read2", byteBuffer);

    var result3 = StandardCharsets.UTF_8.decode(byteBuffer); 
    log.info("result3: {}", result3);
    logPosition("read3", byteBuffer);

    byteBuffer.clear(); 
    logPosition("clear", byteBuffer);
}
```

```java
21:25 [main] - allocate) position: 0, limit: 1024, capacity: 1024 
21:25 [main] - write) position: 12, limit: 1024, capacity: 1024 
21:25 [main] - flip1) position: 0, limit: 12, capacity: 1024 
21:25 [main] - result: Hello world
21:25 [main] - read1) position: 12, limit: 12, capacity: 1024 
21:25 [main] - rewind) position: 0, limit: 12, capacity: 1024 
21:25 [main] - result2: Hello world
21:25 [main] - read2) position: 12, limit: 12, capacity: 1024 
21:25 [main] - result3:
21:25 [main] - read3) position: 12, limit: 12, capacity: 1024 
21:25 [main] - clear) position: 0, limit: 1024, capacity: 1024
```

## 📣 Reference
[패스트캠퍼스 - Spring Webflux 완전정복 : 코루틴부터 리액티브 MSA 프로젝트까지](https://fastcampus.co.kr/dev_online_webflux)<br/>

