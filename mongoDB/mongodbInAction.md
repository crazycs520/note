# MongoDB简介

1. 特点
   * 读，写吞吐量高
   * 扩容简单，自动失效备援
   * Document data model  文档数据模型。适用于数据模型多变，加速应用开发
     * 不用分表。MySQL会把一个大表拆分为多个小表格。
     * 多层次数据表示，查询时省去了关系型数据中的 多表的 join 操作。
     * 内部存储格式是BSON ,Binary JSON
     * collections ， a group of documents。MySQL keeps its data in tables of rows, while MongoDB keeps its data in documents of collections
     * B-Tree Indexs and 允许二级索引，每个collections 不超过64个索引。支持 ascending, descending, unique, compound-key, hashed, text, and even geospatial indexes	
   * Replication.  automate failover  and scale database read.
   * Speed and durability : user control the speed and durability trade-off by chosing write semantics and deciding whethere enable journaling.
     * As a consequence, anyone planning to disable journaling for performance should run with replication.
   * Scaling.  Range-based , hash ， tag-based partitioning mechanism.
2. - ​

# 官方文档阅读记录

## Introduction

1. database and Collections , Bson document

   ```
   use myNewDB
   db.myNewCollection1.insertOne( { x: 1 } )
   ```

2. Views

   * Starting in version 3.4, MongoDB adds support for creating **read-onl**y views from existing collections or other views.

   * ```
     db.createView(<view>, <source>, <pipeline>, <collation> )
     ```

   * Index Use  . but , [Views](https://docs.mongodb.com/manual/core/views/) do not support text search.

   * Drop a View   : ` db.collection.drop()`

3. BSON Types

   * ObjectID : *consist of 12 bytes,* a unique [_id](https://docs.mongodb.com/manual/reference/glossary/#term-id) field that acts as a [primary key](https://docs.mongodb.com/manual/reference/glossary/#term-primary-key). 
     - *a 4-byte value representing the seconds since the Unix epoch,*
     - *a 3-byte machine identifier,*
     - *a 2-byte process id, and*
     - *a 3-byte counter, starting with a random value.*
   * TimeStamps : 8 bytes
     * *the first 32 bits are a `time_t` value (seconds since the Unix epoch)
     * *the second 32 bits are an incrementing *`*ordinal*`* for operations within a given second.*
   * Date   :  8 bytes
     * represents the number of  **milliseconds** since the Unix epoch (Jan 1, 1970). 

##Aggregation

* Aggregation Pipeline
* Map-Reduce
  *  *provide great flexibility compared to the aggregation pipeline,* in general, *map-reduce is less efficient and more complex than the aggregation pipeline.*
* Single purpose Aggregation Operation

## Text Search

* *A collection can only have ***one*** text search index, but that index can cover multiple fields.*

```shell
#create
db.stores.createIndex( { name: "text", description: "text" } )
#find
db.stores.createIndex( { name: "text", description: "text" } )
#Exzat Phrase
db.stores.find( { $text: { $search: "java \"coffee shop\"" } } )
#Term Exclusion
db.stores.find( { $text: { $search: "java shop -coffee" } } )
#sort
db.stores.find( { $text: { $search: "java \"coffee shop\"" } } )
```

## Geospatial Queries

* MongoDB store geospatial data as GeoJSON .

  ```
  location: {
        type: "Point",
        coordinates: [-73.856077, 40.848447]
  }
  ```

  - Valid longitude values are between `-180` and `180`, both inclusive.
  - Valid latitude values are between `-90` and `90` (both inclusive).

* Geospatial Index

  * `2dsphere`   :  support queries that calculate [geometries on an earth-like sphere](https://docs.mongodb.com/manual/geospatial-queries/#geospatial-geometry).
    * `db.collection.createIndex( { <location field> : "2dsphere" } )`
  * `2d `     :  support queries that calculate [geometries on a two-dimensional plane](https://docs.mongodb.com/manual/geospatial-queries/#geospatial-geometry). 
    * `db.collection.createIndex( { <location field> : "2d" } )`

## data models

### Data model Introduction

1. [Document Structure](https://docs.mongodb.com/manual/core/data-modeling-introduction/#document-structure)
   * Reference Data ----Normalized data models  ： use case  and weakness
     * when embedding would result in duplication of data but would not provide sufficient read performance advantages to outweigh the implications of the duplication.
     * to represent more complex many-to-many relationships.
     * to model large hierarchical data sets.
     * weakness ： must issue follow-up queries to resolve the references. In other words, normalized data models can require more round trips to the 
   * Embedded Data----“denormalized” models   : allow applications to retrieve and manipulate related data in a single database operation.
     * you have “contains” relationships between entities. See [Model One-to-One Relationships with Embedded Documents](https://docs.mongodb.com/manual/tutorial/model-embedded-one-to-one-relationships-between-documents/#data-modeling-example-one-to-one).
     * you have one-to-many relationships between entities. In these relationships the “many” or child documents always appear with or are viewed in the context of the “one” or parent documents. See [Model One-to-Many Relationships with Embedded Documents](https://docs.mongodb.com/manual/tutorial/model-embedded-one-to-many-relationships-between-documents/#data-modeling-example-one-to-many).
2. Atomicity of write Operations
   * write operations are atomic at the [document](https://docs.mongodb.com/manual/reference/glossary/#term-document) level,
     * Embedded Data atomic write operations since a single write operation can insert or update the data for an entity. 
     * Reference Data  would split the data across multiple collections and would require multiple write operations that are not atomic collectively.
3. Document Growth
   *  For MMAPv1, if the document size exceeds the allocated space for that document, MongoDB will relocate the document on disk. the default use of the [Power of 2 Sized Allocations](https://docs.mongodb.com/manual/core/mmapv1/#power-of-2-allocation) .
   * if your applications require updates that will frequently cause document growth to exceeds the current power of 2 allocation, you may want to refactor your data model to use references between data in distinct documents rather than a denormalized data model.
   * You may also use a *pre-allocation* strategy to explicitly avoid document growth.
4. Data Use and Performance 
   * if your application only uses recently inserted documents, consider using [Capped Collections](https://docs.mongodb.com/manual/core/capped-collections/)
   * if your application needs are mainly read operations to a collection, adding indexes to support common queries can improve performance.

### Document Validation

* MongoDB provides the capability to validate documents during **updates and insertions**. Validation rules are specified on a per-collection basis using the `validator` option.

  ```json
  db.createCollection( "contacts",
     { validator: { $or:
        [
           { phone: { $type: "string" } },
           { email: { $regex: /@mongodb\.com$/ } },
           { status: { $in: [ "Unknown", "Incomplete" ] } }
        ]
     }
  } )
  ```

* MongoDB also provides the **`validationLevel`** option, which determines how strictly MongoDB applies validation rules to existing documents during an update, and the **`validationAction`** option, which determines whether MongoDB should `error` and reject documents that violate the validation rules or `warn` about the violations in the log but allow invalid documents.

  * By default, `validationLevel` is `strict` and MongoDB applies validation rules to all inserts and updates. 
  * Setting `validationLevel` to `moderate` applies validation rules to inserts and to updates to existing documents that fulfill the validation criteria. With the `moderate` level, updates to existing documents that do not fulfill the validation criteria are not checked for validity.

* Accept or Reject Invalid Documents.

  * The `validationAction` option determines how MongoDB handles documents that violate the validation rules.
  * By default, `validationAction` is `error` and MongoDB rejects any insertion or update that violates the validation criteria. 
  *  When `validationAction` is set to `warn`, MongoDB logs any violations but allows the insertion or update to proceed.

* Bypass Document Validation

  * Users can bypass document validation using the `bypassDocumentValidation` option. 

### Operational factor and Data Model

* Document Growth
* Atomicity
  *  No **single** write operation can change more than one document. 
  * Ensure that your application stores all fields with atomic dependency requirements in the same document. If the application can tolerate non-atomic updates for two pieces of data, you can store these data in separate documents.
* Sharding 
  * MongoDB uses [sharding](https://docs.mongodb.com/manual/reference/glossary/#term-sharding) to provide horizontal scaling.
  * To distribute data and application traffic in a sharded collection, MongoDB uses the [shard key](https://docs.mongodb.com/manual/core/sharding-shard-key/#shard-key). Selecting the proper [shard key](https://docs.mongodb.com/manual/core/sharding-shard-key/#shard-key) has significant implications for performance, and can enable or prevent query isolation and increased write capacity. 
* Indexes
  * MongoDB automatically creates a unique index on the `_id` field.
  * As you create indexes, consider the following behaviors of indexes:
    * Each index requires at least 8 kB of data space.
    * Adding an index has some negative performance impact for write operations. For collections with high write-to-read ratio, indexes are expensive since each insert must also update any indexes.
    * Collections with high read-to-write ratio often benefit from additional indexes. Indexes do not affect un-indexed read operations.
    * When active, each index consumes disk space and memory. This usage can be significant and should be tracked for capacity planning, especially for concerns over working set size.
* Large Number of Collections
  * In certain situations, you might choose to store related information in several collections rather than in a single collection.
* Collection Contain Large Number of Small Documents
* Storage Optimization for Small Document
  * Use the `_id` field explicitly.
  * Use shorter field names. (not necessary )
* Data LifeCycle Management
  * expire data from collections by setting TTL