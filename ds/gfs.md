![gfs](http://github.com/crazycs520/images/blob/master/gfs.png?raw=true)

![gfs](http://github.com/crazycs520/images/blob/master/gfs1.png?raw=true)

# 基础

## Overview

* small random read 优化

  Batch and sort small reads

* master 存储3个主要的Metadata:

  1. the file and chunk namespaces
  2. the mapping from files to chunks
  3. the locations of each chunk’s replicas

  * 第1，2种metadata永久保存在operation log 中，第3种信息是master 启动和chunkserver 加入cluster 时向chunkserver 轮询其所存储的Chunk 的location数据，然后通过 HeartBeat 监控ChunkServer 的状态 ，这样可以保证Mater服务器能够拥有最新的数据信息

* 64M的Chunk对应的 元数据 需要不到64 bytes ，采用prefix compression 算法压缩。

* Operation Log  :   stored on the master’s local diskan d replicated on remote machines.

  * record of critical metadata changes
  * a **logical time line** that defines the order of concurrent operations.

* checkpoint , a compact B-tree like form ,  can be directly mapped into memory

  * The master checkpoints its state whenever the log grows beyond a certain size so that it can recover by loading the latest checkpoint from local disk and replaying only the limited number of log records after that.
  * The master switches to a new log file and creates the new checkpoint in a separate thread. The new checkpoint includes all mutations before the switch.

* Consistency Model

  ​

  ![gfs](http://github.com/crazycs520/images/blob/master/gfs2.png?raw=true)

  * A file region is **consistent** if all clients will always see the same data, regardless of which replicas they read from. 
  * A region is **defined** after a file data mutation if it is consistent and clients will see what the mutation writes in its entirety.

* Write ,  appends?

  * A write causes data to be written at an **application-specified file offset**
  * A record append causes data (the “record”) to be appended atomically  **at least once** even in the presence of concurrent mutations, but at an **offset of GFS’s choosing**

* after a successful mutations, the mutated file region is guaranteed to be defined

  * applying mutations to a chunkin the same order on all its replicas
  * using  **chunk version** numbers to detect any replica that has become stale

## System Interactions

​	**minimize the master's involvement in all operation**

* ![gfs](https://raw.githubusercontent.com/crazycs520/images/master/gfs3.png)
* snapshot
  * The snapshot operation makes a copy of a file or a directory tree (the “source”) almost instantaneously
  * use standard copy-on-write techniques to implement snapshots.
  * When the master receives a snapshot request, it first revokes any outstanding leases on the chunks in the files it is about to snapshot.

* 如何选择一个副本的ChunkServer?
  1. 磁盘利用率低
  2. 限制最新数据块的写入数量，避免单个ChunkServer成为热点
  3. 跨机架跨数据中心






# 问题

* Describe a sequence of events that result in a client reading stale data from the [Google File System](https://pdos.csail.mit.edu/6.824/papers/gfs.pdf)
  * Since clients cache chunkl ocations, they may read from a stale replica before that information is refreshed. Moreover, as most of our files are append-only, a stale replica usually returns a premature end of chunk  rather than outdated data. When a reader retries and contacts the master, it will immediately get current chunk locations.





# 总结

* GFS特点
  1. AP and weak consistency
  2. 存储大文件，而小文件虽然也支持，但并不对其做优化
  3. Append new data rather than overwrite existing data