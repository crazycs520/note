# 简介

LevelDB 是一个 key-value 持久化存储引擎，类似于单个 [Bigtable tablet (section 5.3)](http://research.google.com/archive/bigtable.html) 的实现。

LevelDB 提供了简洁的接口：`Put`，`Delete`， `Get`。


# 接口实现

## Put 

```c
Status DB::Put(const WriteOptions& opt, const Slice& key, const Slice& value) {
  WriteBatch batch;
  batch.Put(key, value);
  return Write(opt, &batch);
}
```

`Put` 接口内部将写入 key-value 封装成一个 `WriteBatch` 后，调用 `Write` 接口。

### WriteBatch

`WriteBatch` 将 key-value 存储到一个连续的 byte 数组里面，编码如下：

![WriteBatch 编码](../images/leveldb01.jpg)

* sequence_number: 写入序列号，每次写入一个 key-value 后加 1 单调递增。
* count: 一个 `WriteBatch` 中写入 key-value 对的个数。
* type 是写入类型，有 2 种写入类型：
  * kTypeValue：写入 key-value 操作。
  * kTypeDeletion：删除 key 操作。其对应的 value_length 是0， value 是空。
* key_size: key 的大小，用`PutVarint32` 编码。
* key: 用户写入的 key 的值。
* value_size: value 的大小。
* value: 用户写入 value 的值。

可以使用 `WriteBatch`单次写入多个 key-value 来提升写入性能。

### Write

简单的写入流程如下：

1. 检查是否需要 compact
2. 分配 sequence
3. 写 WAL 日志。
4. 如果 `WriteOptions` 中的 `sync` 是 true，则需要等待 WAL 日志 flush 完成。
5. 写入 memtable , 即 skip-list。

第 5 步写入前，需要将用户 key-value 和 sequence 一起编码成一个 byte数组后再写入 memtable，编码如下：

![写入 key-value 编码](../images/leveldb02.jpg)

* Internal_key_size: internal key 的大小，它等于用户 key 的大小 + 8，其中 7 个字节存  `sequence_number`，一个字节存 `type`。

### memtable 的实现

memtable 内部实现是 [skip-list](https://homepage.cs.uiowa.edu/~ghosh/skip.pdf)。

![写入 key-value 编码](../images/leveldb03-skip-list.jpg)

插入 key 流程如下：

1. `FindGreaterOrEqual`, 找到一个大于等于 key 的节点，同时记录查找过程中各层索引信息。
   * 由于每个写入的 key 的 sequence 肯定不一样，所以肯定不会找到相等的 key。
2. 新建 node 并随机分配一个索引层高。
3. 插入 node 并更新各层的索引信息。



skip-list 不支持并发写入，但是支持 1 个写入的同时多个并发读。skip-list 中 node 的各层索引的更新和读取是采用的原子操作，使用 `release-acquire` 的内存顺序。



## Delete

`Delete` 实现几乎和 `put` 一样，只是写入类型是 `kTypeDeletion`，其 value_length 是 0， value 是空。

## Get









# 参考资料

[LevelDB 官方文档](https://github.com/google/leveldb/tree/master/doc)

[levelDB 实现解析]([https://yuerblog.cc/wp-content/uploads/leveldb%E5%AE%9E%E7%8E%B0%E8%A7%A3%E6%9E%90.pdf](https://yuerblog.cc/wp-content/uploads/leveldb实现解析.pdf))

 [skip-list](https://homepage.cs.uiowa.edu/~ghosh/skip.pdf)

[如何理解 C++11 的六种 memory order](https://www.zhihu.com/question/24301047)

