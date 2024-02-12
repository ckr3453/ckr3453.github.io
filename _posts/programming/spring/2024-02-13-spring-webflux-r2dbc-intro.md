---
title: "Spring Webflux) R2DBC"
categories: 
    - spring
date: 2024-02-13
last_modified_at: 2024-02-13
toc: true
toc_sticky: true
# excerpt: "R2DBC에 대해 알아보자"
---

## R2DBC

<center><img alt="image" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/9a3f77da-964d-4d01-be61-6c30ea13583a"></center><br/>

R2DBC란 Reactive Relational Database Connectivity의 약자로 RDBMS(관계형 데이터베이스)에서 reactiveprogramming이 가능하게끔 지원하는 API이다.

Reactive streams 스펙을 제공하여 Proejct Reactor 기반으로 구현이 가능하다.

지원하는 데이터베이스 API는 다음과 같다.

### 지원하는 데이터베이스 API

- 공식 지원
  - R2DBC-h2
  - R2DBC-MSSQL
  - R2DBC-pool(Reactor pool로 커넥션 풀을 제공함)
- 벤더에서 지원
  - Oracle-R2DBC
  - R2DBC-MariaDB
  - R2DBC-PostgreSQL
  - R2DBC-MySQL

## R2DBC-SPI

<center><img alt="image" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/f269b911-8d5f-4ebf-8db5-0049ef0407f0"></center><br/>

<center><img alt="image" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/93d0fab5-3194-474d-b2e6-26e7c1320656"></center><br/>

R2DBC-SPI(R2DBC Service Provider Interface)를 통해 각 데이터베이스 드라이버는 SPI에 정의된 인터페이스를 구현 해야한다.

제공하는 표준 스펙은 다음과 같다.

### Connection

```java
public interface Connection extends Closeable {

  // db connection 닫기
  @Override
  Publisher<Void> close();

  // sql를 매개변수로 넘기고 statement를 생성
  Statement createStatement(String sql);

  // db의 version과 product name을 제공
  ConnectionMetadata getMetadata();

  // transaction을 시작한다. (TransactionDefinition로 고립 수준, 읽기전용 여부, 이름, lockWaitTime 등을 설정)
  Publisher<Void> beginTransaction();
  Publisher<Void> beginTransaction(TransactionDefinition definition);

  Publisher<Void> commitTransaction();

  // transaction 중간에 savepoint를 만들고 rollback이 가능하다.
  Publisher<Void> createSavepoint(String name);

  boolean isAutoCommit();

  IsolationLevel getTransactionIsolationLevel();

  Publisher<Void> rollbackTransaction();
  Publisher<Void> rollbackTransactionToSavepoint(String name);

  // autoCommit과 IsolationLevel(고립수준)을 설정가능하다.
  Publisher<Void> setAutoCommit(boolean autoCommit);
  Publisher<Void> setTransactionIsolationLevel(IsolationLevel isolationLevel);
}
```

- Connection, ConnectionFactory 등 DB connection과 관련된 스펙
- Connection 및 Transaction 과 관련한 메소드를 제공

### ConnectionFactory, ConnectionFactoryMetadata

```java
public interface ConnectionFactory {

  Publisher<? extends Connection> create();

  ConnectionFactoryMetadata getMetadata();
}

public interface ConnectionFactoryMetadata {

  String getName();
}
```
  
- Connection 객체를 생성하도록 돕는 factory 메소드를 제공

### Exception

- RuntimeException을 상속
- R2dbcException, R2dbcTimeoutException, R2dbcBadGrammarException 등의 Exception과 관련된 스펙

### Result
```java
public interface Result {

    Publisher<Long> getRowsUpdated();

    <T> Publisher<T> map(BiFunction<Row, RowMetadata, ? extends T> mappingFunction);

    default <T> Publisher<T> map(Function<? super Readable, ? extends T> mappingFunction) {
        Assert.requireNonNull(mappingFunction, "mappingFunction must not be null");
        return map((row, metadata) -> mappingFunction.apply(row));
    }

    Result filter(Predicate<Segment> filter);

    <T> Publisher<T> flatMap(Function<Segment, ? extends Publisher<? extends T>> mappingFunction);

    interface Segment {

    }

    interface RowSegment extends Segment {

        Row row();

    }

    interface OutSegment extends Segment {

        OutParameters outParameters();
    }

    interface UpdateCount extends Segment {

        long value();

    }

    interface Message extends Segment {

        R2dbcException exception();

        int errorCode();

        @Nullable
        String sqlState();

        String message();

    }

}
```

- Result, Row, RowMeatadata 등 result(응답) 관련 스펙

### Statement
```java
public interface Statement {

  //add: 이전까지 진행한 binding을 저장하고 새로운 binding을 생성
  Statement add();

  // bind: sql에 parameter를 bind한다. index, name 단위로 parameter를 bind
  Statement bind(int index, Object value);
  Statement bind(String name, Object value);
  Statement bindNull(int index, Class<?> type);
  Statement bindNull(String name, Class<?> type);

  //생성된 binding 수만큼 쿼리를 실행하고 Publisher로 반환
  Publisher<? extends Result> execute();

}
```

- Statement 등 요청을 보낼때 만들게되는 Statement 관련 스펙
- 커넥션 으로부터 db에 sql을 보내기 위한 메소드를 제공

## 📣 Reference
[패스트캠퍼스 - Spring Webflux 완전정복 : 코루틴부터 리액티브 MSA 프로젝트까지](https://fastcampus.co.kr/dev_online_webflux)<br/>
[Spring Data R2DBC](https://spring.io/projects/spring-data-r2dbc)<br/>


