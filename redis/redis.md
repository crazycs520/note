# 基本概念

Redis is an in-memory , non-relational(NoSQL)  database; used as a database, cache and message broker.

特点：

* 单线程机制
* 支持5种数据类型：string，hash(散列) , lists , sets , zset(sorted sets）



## 数据类型

* **string** , 是二进制安全的，可以包含任何数据，如jpg图片或序列化对象。

```Shell
127.0.0.1:6379> set myname "crazycs"	# 设置键名为 myname , 值为 "crazycs"
OK
127.0.0.1:6379> get myname
"crazycs"
```

**注意**：一个键最大能存储512MB

* **Hash** ， 键名对集合，是string 类型的field 和 value 的映射表，适合存储对象

  每个 hash 可以存储2^32 -1 键值对。

  ```
  hset
  hget
  hgetall
  hdel
  ```

  ​

* **List** , 字符串列表，按照插入顺序排序，可以添加一个元素到列表头或尾（左或右）。

  列表最多存储 2^32 -1 元素

  ```
  lpush
  rpush
  lpop
  rpop
  lrange 
  lindex
  ```

  ```Shell
  127.0.0.1:6379> lpush runoob redis
  (integer) 1
  127.0.0.1:6379> lpush runoob mongodb
  (integer) 2
  127.0.0.1:6379> lpush runoob mysql
  (integer) 3
  127.0.0.1:6379> lrange runoob
  (error) ERR wrong number of arguments for 'lrange' command
  127.0.0.1:6379> lrange runoob 0 10
  1) "mysql"
  2) "mongodb"
  3) "redis"
  ```

  ​

* **Set** , string 类型的无序集合(没有重复元素)，通过哈希表实现，故添加，删除，查找的复杂度都是O(1)。

  ```
  sadd key member
  smembers
  sismember
  serm
  ```

  ```Shell
  127.0.0.1:6379> sadd db redis
  (integer) 1
  127.0.0.1:6379> sadd db mongodb
  (integer) 1
  127.0.0.1:6379> sadd db mysql
  (integer) 1
  127.0.0.1:6379> sadd db mysql
  (integer) 0
  127.0.0.1:6379> smembers db
  1) "mongodb"
  2) "mysql"
  3) "redis"
  ```

  ​

* **zset(sorted set)** ,  同set一样，不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。

  zset的成员是唯一的,但分数(score)却可以重复。

  ```Shell
  ZADD 			#Adds member with the given score to the ZSET
  ZRANGE 			#Fetches the items in the ZSET from their positions in sorted order
  ZRANGEBYSCORE 	#Fetches items in the ZSET based on a range of scores
  ZREM 			#Removes the item from the ZSET, if it exists
  ```

  ​

  ```shell
  zadd key score member 
  ```

  ```Shell
  127.0.0.1:6379> zadd students 1 Bob
  (integer) 1
  127.0.0.1:6379> zadd students 2 Marry
  (integer) 1
  127.0.0.1:6379> zadd students 3 Jim
  (integer) 1
  127.0.0.1:6379> zadd students 2 Bim
  (integer) 1
  127.0.0.1:6379> zrangebyscore students 0 10
  1) "Bob"
  2) "Bim"
  3) "Marry"
  4) "Jim"
  ```


##install

[http://www.runoob.com/redis/redis-install.html](http://www.runoob.com/redis/redis-install.html)

[https://redis.io/download](https://redis.io/download)

###Mac install

    1. 官网<http://redis.io/> 下载最新的稳定版本,这里是3.2.0

    2. sudo mv 到 /usr/local/

    3. sudo tar -zxf redis-3.2.0.tar 解压文件

    4. 进入解压后的目录 cd redis-3.2.0

    5. sudo make test 测试编译

   6. sudo make install 

或者用  `brew install redis`

##启动redis服务器

##默认配置启动

```Bash
redis-server
```

​	redis 默认端口是6379

###配置文件启动

```
redis-server /usr/local/redis-4.0.2/redis.conf
```

Redis目录下有一个默认`redis.conf`配置文件，可以通过复制这个模板文件修改相关配置来启动redis

## Redis命令行客户端

```shell
redis-cli	#redis 客户端，连接本地redis
redis-cli -h host -p port -a password	#连接远程redis , password 用 "" 包括，默认port 是 6379 , 默认host是127.0.0.1
```

### 停止redis 服务

```
redis-cli shutdown
```

##全局命令

1. key

```shell
set mykey crazycs
get mykey
del mykey
```



