---
title: "Spring Webflux) R2DBC - Reactive MongoDB (작성중)"
categories: 
    - spring
date: 2024-02-26
last_modified_at: 2024-02-26
toc: true
toc_sticky: true
---

## MongoDB Driver

MongoDB 사에서는 다음과 같이 2가지의 공식 Java용 Driver를 제공한다.

<center><img alt="image" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/9a3f77da-964d-4d01-be61-6c30ea13583a"></center><br/>

- 동기 방식의 Java Driver (사진)
  - 동기적으로 동작하는 어플리케이션을 위한 드라이버
  - 클라이언트가 요청을 보내면 응답이 돌아오기전까지 thread는 blocking 상태
      - 쿼리를 다 처리하고 나면 이후에 결과 리턴
  - Reactive Streams에 비해 메소드가 응답 객체를 바로 반환하기 때문에 직관적이며 쉽게 작성가능
      - 그러나 동시성 문제로 인하여 많은 요청을 처리하기 힘듬

<center><img alt="image" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/9a3f77da-964d-4d01-be61-6c30ea13583a"></center><br/>

- 비동기 방식의 Reactive Streams Driver (사진)
  - 비동기적으로 동작하는 어플리케이션을 위한 MongoDB 드라이버
    - 클라이언트가 요청을 보내면 thread는 non-blocking 상태
    - 모든 응답이 Publisher를 이용해서 전달되기 때문에 처리하기에 어려움
        - 요청을 보낸 시점과 받는 시점이 다르기 때문에 직관적으로 처리하기 어려움
    - Spring Reactive Stack 과 함께 사용되어 높은 성능과 안정성을 제공함


## Spring Data MongoDB Reactive
- 추가

## Reactive Streams MongoDB Driver 
(사진)

### MongoCollection (사진)

- 이름에서 알수있다 싶이 컬렉션에 접근하기 위한 객체이다.
- 컬렉션은 도큐먼트를 포함하는 컨테이너의 개념으로 r2dbc 개념으로 보면 table에 해당함
- MongoClient > MongoDatabase > MongoCollection
- MongoClients로부터 create를 통해서 MongoClient를 생성할 수 있음
- MongoClient에서 getDatabase를 통해서 MongoDatabase 접근이 가능함
- MongoDatabase에서 getCollection을 통해서 MongoCollection을 접근

- MongoCollection 획득 예제

```java
// ConnectionString을 이용해서 MongoDB 연결 정보를 String 형태로 제공
var connection = new ConnectionString("mongodb://localhost:1234/wooman")
// MongoClientSettings builder에 Connection 정보를 전달
var settings = MongoClientSettings.builder()
        .applyConectionString(connection)
        .build();

// MongoClientSettings로 MongoClient 생성
try (MongoClient mongoClient = MongoClients.create(settings)) {

  // MongoClient로 MongoDatabase 접근
  var database = mongoClient.getDatabase("ckr");
  log.info("database: {}", database.getName());

  // MongoDatabase로 MongoCollection 접근
  var collection = database.getCollection("employee");
  log.info("collection: {}", collection.getNamespace().getCollectionName());
}
```

```
56:04 [main] - MongoClient with metadata {"driver": {"name": "mongo-java-driver|reactive-streams", "version": "4.6.1"}, ...}
56:04 [main] - database: ckr
56:04 [main] - collection: employee
```

- count
  - `Publisher<Long> countDocuements(ClientSession clientSession, Bson filter, CountOptions options);`
  - ClientSession을 통해서 multi document transaction을 제공
    - 세션을 통해서 count document 뿐만 아니라 여러개의 연산들을 하나의 세션 즉, 트랜잭션으로 수행할 수 있음.
  - Bson 구현제로 filter 제공
  - CountOptions로 hint, limit, skip, maxTime, collation 등의 정보들을 제공

- find
  - `FindPublisher<TDocument> find(ClientSession clientSession, Bson filter);`
  - 다음과 같이 Filters 클래스의 연산자 메소드들을 통해서 filter 설정이 가능하다.
    ```java
    public final class Filters {
      private Filters() {
      }

      public static <TItem> Bson eq(@Nullable final TItem value) { … }

      public static <TItem> Bson ne(final String fieldName, @Nullable final TItem value) { … }

      public static <TItem> Bson gt(final String fieldName, final TItem value) { … }

      public static <TItem> Bson in(final String fieldName, final Iterable<TItem> values) { … }
      ...
    }
    ```

- aggregate 
  - aggregate? -> pipeline을 생성하고 mongo shard 전체에 대하여 필터, 집계, 그룹 등의 연산을 수행할 수 있다. (사진)
    - ex) 여러개의 mongo db 인스턴스로 분산 되어있는 도큐먼트들에 대하여 필터, 집계 등 한번에 연산을 수행하여 결과를 가져올 수 있음
  - `AggregatePublisher<TDocument> aggregate(ClientSession clientSession, List<? extends Bson> pipeline);`
  - Aggregates 클래스를 통해서 aggregate pipeline을 제공한다.
    ```java
    public final class Aggregates {
      public static Bson addFields(final List<Field<?>> fields) { … }

      public static Bson set(final List<Field<?>> fields) { … }

      public static Bson count(final String field) { … }

      public static Bson match(final Bson filter) { … }

      public static Bson project(final Bson projection) { … }

      public static Bson sort(final Bson sort) { … }
      ...
    }
    ```

- watch
  - watch? -> document, collection 에 변화가 일어났을 때 감지할 수 있는 연산
  - `ChangeStreamPublisher<Document> watch(ClientSession clientSession, List<? extends Bson> pipeline);`
  - Aggregates 클래스를 통해서 aggregate pipeline을 제공한다. (addFields, match, redact, set, ..)
  - ChangeStreamPublisher를 반환하고 해당 Publisher를 subscribe하는 구조
    - ChangeStreamDocument를 onNext로 전달함
    - resumseToken, 변경사항이 발생한 document 혹은 _id를 반환 가능함
      ```java
      public interface ChangeStreamPublisher<TResult> extends Publisher<ChangeStreamDocument<TResult>> {}
      ```

      ```java
      public final class ChangeStreamDocument<TDocument> {
        @BsonId()
        private final BsonDocument resumeToken;
        private final BsonDocument namespaceDocument;
        private final BsonDocument destinationNamespaceDocument;
        private final TDocument fullDocument;
        private final BsonDocument documentKey;
        private final BsonTimestamp clusterTime;
        @BsonProperty("operationType")
        private final String operationTypeString;
        @BsonIgnore
        private final OperationType operationType;
        private final UpdateDescription updateDescription;
        private final BsonInt64 txnNumber;
        private final BsonDocument lsid;
        ..
      }
      ```

- bulkWrite
  - Delete, Insert, Replace, Update 등을 모아서 한 번에 실행하는 operation
  - BulkWriteOptions 클래스를 통해서 option 을 설정할 수 있음
  - `Publisher<BulkWriteResult> bulkWrite(ClientSession clientSession, List<? extends WriteModel<? extends TDocument>> requests, BulkWriteOptions options);`
  - WriteModel
    - DeleteManyModel: 조건을 만족하는 document를 모두 삭제
    - DeleteOneModel: 조건을 만족하는 document를 최대 1개만 삭제
    - InsertOneModel: 하나의 document를 추가
    - ReplaceOneModel: 조건을 만족하는 document를 최대 1개만 대체
    - UpdateManyModel: 조건을 만족하는 document를 모두 수정
    - UpdateOneModel: 조건을 만족하는 document를 최대 1개만 수정
      ```java
      public final class BulkWriteOptions {
        // requests를 순차적으로 진행할것인지 여부
        private boolean ordered = true; 

        // insert, update할때 validation을 하지않고 넘어가는지에 대한 여부
        private Boolean bypassDocumentValidation; 

        private BsonValue comment;   
        private Bson variables;     
      }
      ```

- insert
  - 하나 혹은 여러 document를 추가하는 operation
  - `Publisher<InsertOneResult> insertOne(ClientSession clientSession, TDocument document, InsertOneOptions options);`
  - `Publisher<InsertManyResult> insertMany(ClientSession clientSession, List<? extends TDocument> documents, InsertManyOptions options);`
  - InsertOneOptions 혹은 InsertManyOptions를 통해서 validation 우회 여부를 결정함
    - InsertManyOptions라면 insert의 순서를 보장할지 결정함
  - InsertOneResult, InsertManyResult는 wasAcknowledged와 getInsertedIds 메소드를 통해서 write이 성공했는지, write된 id들을 제공함

- MongoCollection - update
  - 하나 혹은 여러 document를 수정하는 operation
  - `Publisher<UpdateResult> updateOne(ClientSession clientSession, Bson filter, List<? extends Bson> update, UpdateOptions options);`
  - `Publisher<UpdateResult> updateMany(ClientSession clientSession, Bson filter, List<? extends Bson> update, UpdateOptions options);`
  - Filters helper 클래스를 통해서 filter 설정 가능
  - Updates helper 클래스를 통해 update 설정 가능
  - UpdateOptions를 통해서 upsert, hint, collation, variables 등 제공

    ```java
    public class UpdateOptions {
      private boolean upsert;
      private Boolean bypassDocumentValidation;
      private Collation collation;
      private List<? extends Bson> arrayFilters;
      private Bson hint;
      private String hintString;
      private BsonValue comment;
      private Bson variables;
    }
    ```

- atomic
  - findOneAndDelete, findOneAndReplace, findOneAndUpdate 등 find와 write를 묶어서 한번에 진행하기 때문에 atomic한 operation 제공
  - `Publisher<TDocument> findOneAndDelete(ClientSession clientSession, Bson filter, FindOneAndDeleteOptions options);`
  - `Publisher<TDocument> findOneAndReplace(ClientSession clientSession, Bson filter, TDocument replacement, FindOneAndReplaceOptions options);`
  - `Publisher<TDocument> findOneAndUpdate(ClientSession clientSession, Bson filter, List<? extends Bson> update, FindOneAndUpdateOptions options);`

- index
  - Collection에서 특정 필드들에 대한 index 생성, 조회, 삭제 가능
  - `Publisher<String> createIndexes(ClientSession clientSession, List<IndexModel> indexes, CreateIndexOptions createIndexOptions);`
  - `ListIndexesPublisher<Document> listIndexes(ClientSession clientSession);`
  - `Publisher<Void> dropIndex(ClientSession clientSession, Bson keys, DropIndexOptions dropIndexOptions);`
  - Indexes helper 클래스를 통해서 다양한 index 제공
    ```java
    public final class Indexes {
      public static Bson ascending(final List<String> fieldNames) { … }
      public static Bson descending(final List<String> fieldNames) { … }
      public static Bson geo2dsphere(final List<String> fieldNames) { … }
      public static Bson geo2d(final String fieldName) { … }
      public static Bson text(final String fieldName) { … }
      public static Bson hashed(final String fieldName) { … }
    }
    ```

### Mongo Document



## 📣 Reference
[패스트캠퍼스 - Spring Webflux 완전정복 : 코루틴부터 리액티브 MSA 프로젝트까지](https://fastcampus.co.kr/dev_online_webflux)<br/>


