# start

## install

[http://www.runoob.com/redis/redis-install.html](http://www.runoob.com/redis/redis-install.html)

[https://redis.io/download](https://redis.io/download)

### Mac install

```
1. 官网<http://redis.io/> 下载最新的稳定版本,这里是3.2.0
```

```
2. sudo mv 到 /usr/local/
```

```
3. sudo tar -zxf redis-3.2.0.tar 解压文件
```

```
4. 进入解压后的目录 cd redis-3.2.0
```

```
5. sudo make test 测试编译
```

1. sudo make install 

或者用  `brew install redis`

## 启动redis服务器

1. 默认配置启动

```bash
redis-server
```

​	redis 默认端口是6379

2. 配置文件启动

```
redis-server /usr/local/redis-4.0.2/redis.conf
```

Redis目录下有一个默认`redis.conf`配置文件，可以通过复制这个模板文件修改相关配置来启动redis

## Redis命令行客户端

```shell
redis-cli	#redis 客户端，连接本地redis
redis-cli -h host -p port -a password	#连接远程redis , password 用 "" 包括，默认port 是 6379 , 默认host是127.0.0.1
```

* 停止redis 服务

```
redis-cli shutdown
```

## 全局命令

1. 查看所有键

   ```
   keys *
   ```

   `keys` 会遍历所有键，时间复杂度是O(n)，线上环境禁止使用

2. 键总数

   ```
   dbsize		#不会遍历所有键，而是直接获取redis的内置键总数变量，所以时间复杂度为 O(1)
   ```

3. 查看键是否存在

   ```
   exists [key]
   ```

4. 删除键

   ```shell
   del key [key ...]		#支持删除多个键
   ```

5. 键过期设置

   ```
   expire key seconds		#设置key键过期时间，单位是秒
   ttl key 				#返回键的剩余过期时间，大于等于0：键的剩余时间；-1：键没有设置过期时间; -2:键不存在
   ```

6. 键的数据结构类型

   ```
   type key
   ```

7. 查看底层内部编码

   ```
   object encoding key
   ```

   ​





Redis is an in-memory , non-relational(NoSQL)  database; used as a database, cache and message broker.

特点：

* 单线程机制
* 支持5种数据类型：string，hash(散列) , lists , sets , zset(sorted sets），每种类型有不同的内部编码实现，


# API理解和使用

## 数据类型

### string

 二进制安全的，可以包含任何数据，如jpg图片或序列化对象。一个string键的值最大存储512MB

内部编码有3种：

* int : 8字节的长整型
* embstr :  小于等于39个字节的字符串
* raw : 大于39个字节的字符串

1. 设置值

   ```
   set key value [ex seconds] [px milliseconds] nx|xx
   ```

   * ex : 设置秒级过期时间
   * px : 毫秒级过期时间
   * nx: 键必须不存在才设置成功
   * xx : 与nx 相反

   ```
   setex key seconds value
   setnx key value
   ```

   setnx 可以作为分布式锁的一种实现方案，参考官方：https://redis.io/topics/distlock

2. 获取值

   ```
   get key
   ```

3. 批量设置值

   ```
   mset key value [key value ...]
   ```

4. 批量获取值

   ```
   mget key [key ...]
   ```

批量操作可以提高效率，节省网络消耗时间。

5. 计数

   ```
   incr key
   decr key
   incrby key increment
   decrby key decrement
   incrbyfloat key increment
   ```

   * key的值只能是整数，否则返回错误
   * 返回自增后的结果
   * 键若不存在，则按照值为0 增加

6. 追加值

   ```
   append key value
   ```

7. 字符串长度

   ```
   strlen key
   ```

8. 设置并返回原值

   ```
   getset key value
   ```

9. 设置指定位置的字符

   ```
   setrange key offeset value
   ```

10. 获取部分字符串

   ```
   getrange key start end
   ```

#### 典型应用

1. 缓存功能

   redis作为缓存层，mysql 做为存储层，比如缓存一个用户信息

   ```C
   Userinfo getUserinfo(long id){
     userRedisKey = "user:info:"+id
     value = redis.get(userRedisKey)
     Userinfo userinfo;
     if value != null{
       userinfo = deserialize(value)	//将值反序列化为 Userinfo 
     }else{
       userinfo = mysql.get(id)
       if userinfo != null {
         redis.setex(userRedisKey,3600,serialize(userinfo))
       }
     }
     return userinfo
   }
   ```

   缓存跟新：

   参考：[缓存更新的套路](https://coolshell.cn/articles/17416.html)

2. 计数

   如点赞，视频播放次数等

   ```C
   long incrVideoCounter(long id){
     key = "video:playCount:"+id
     return redis.incr(key)
   }
   ```

3. 共享session

   防止用户在访问同一网站不同的服务器时需要重新登录

4. 限速

   如限制用户每分钟获取验证码频率，一个IP不能在一秒内访问超过n 次

   ```C
   phoneNum = "152xxxxxxxxx"
   key = "shortMsg:limit:"+phoneNum
   isExists = redis.set(key,1,"EX 60","NX")
   if(isExists != null || redis.incr(key) <= 5){
     //通过
   }else{
     //限速
   }
   ```

### Hash

 键值本身又是一个键值对结构，是string 类型的field 和 value 的映射表，适合存储对象

每个 hash 可以存储2^32 -1 键值对。

内部编码有3种

* ziplist : 压缩列表，节省内存，但性能下降
* hashtable :  读写速度快，时间复杂度为 O(1)
* quicklist : 结合前两种的优势
  *  当field 个数比较少，且没有大的value时，内部编码为ziplist
  *  当value大于64字节，内部编码由ziplist自动变为hashtable
  *  当field 超过512个时，内部编码由ziplist自动变为hashtable



1. 设置，获取，删除值

   ```
   hset key field value
   hget key field
   hdel key field [field ...] #可以删除多个field
   ```

2. 计算field 个数

   ```
   hlen key
   ```

3. 批量设置和获取

   ```
   hmget key field [field ...]
   hmset field value [field value ...]
   ```

4. 判断field 是否存在

   ```
   hexists key field
   ```

5. 获取所有的field 

   ```
   hkeys key
   ```

6. 获取所有的value

   ```
   hvals key
   ```

7. 获取所有的 field-value

   ```
   hgetall key
   ```

   如果hash 元素比较多，会阻塞redis, 如果只需要部分，推荐使用 `hmget`。如果的确要全部的field-value，建议用渐进式遍历hash的命令 `hscan`

8. 计数

   ```
   hincrby key field
   hincrbyfloat key field increment
   ```

9. 计算value 字符串长度

   ```
   hstrlen key field
   ```

#### 应用场景

1. 缓存用户信息（对象类型）

   ```C
   Userinfo getUserinfo(long id){
     userRedisKey = "user:info:"+id
     value = redis.hgetall(userRedisKey)
     Userinfo userinfo;
     if value != null{
       userinfo = transferMapToUserinfo(value)	//将映射关系转为userifo
     }else{
       userinfo = mysql.get(id)
       if userinfo != null {
         redis.hmset(userRedisKey,transferUserinfoToMap(userinfo))
         redis.expire(userRedisKey,3600)
       }
     }
     return userinfo
   }
   ```

   ​

###List

 字符串列表，按照插入顺序排序，可以添加一个元素到列表头或尾（左或右）。

列表最多存储 2^32 -1 元素

内部编码：

* ziplist ： 压缩表

* linkedlist: 链表

* quicklist

  * 当元素个数过少且没有大元素时，为ziplist 

  * 当元素个数超过512或某个元素超过512时，变为linkedlist

    ​

1. 添加操作

   * 从右边或左边插入元素

     ```
     rpush key value [value ...]
     lpush key value [value ...]
     ```

   * 在某个元素前后插入元素

     ```
     linsert key befor|after pivot value #在pivot元素前|后插入value
     ```

2. 查找

   * 获取指定范围内的元素列表

     ```
     lrange key start end
     ```

   * 获取指定索引下标的元素

     ```
     lindex key index
     ```

   * 获取列表长度

     ```
     llen key
     ```

3. 删除

   * 左侧或右侧弹出

     ```
     lpop key
     rpop key
     ```

   * 删除指定元素

     ```
     lrem key count value
     ```

     找到列表中等于value的元素进行删除，并根据value的不同分三种行为

     * count  > 0  : 从左到右，最多删除count个元素
     * count  < 0  : 从右到左，最多删除-count个元素
     * count = 0 : 删除所有等于value的元素

   * 按照索引范围修剪列表

     ```
     ltrim key start end
     ```

4. 修改

   ```
   lset key index newValue
   ```

5. 阻塞操作

   ```
   blpop key [key ...] timeout
   brpop key [key ...] timeout
   ```

   * key [key …] :  多个列表的键
   * timeout : 阻塞时间
     * 如果列表为空，timeout=3，那个客户端要等到3秒后返回，如果timeout =0 ,那个客户端一直阻塞下去，直到列表不为空
     * 列表不为空，客户端立即返回
   * 使用brpop的两点注意：
     * 如果是多个键，brpop会从左到右遍历键，一旦有一个能弹出元素，客户端立即返回
     * 如果多个客户端对同一个键执行brpop,那么最先执行的brpop的客户端可以获取弹出值。后面的客户端继续阻塞

#### 使用场景

1. 消息队列

   lpush + brpop实现阻塞队列

2. 文章列表

   每篇文章用hash结构存储，然后向文章列表添加文章

```
lpush + lpop = Stack (栈)
lpush + rpop = Queue(队列)
lpush + ltrim = Capped Collection (有限集合)
lpush + brpop = Message Queue(消息队列)
```

### Set

保存多个字符串类元素，一个set最多保存2^32-1个元素。支持集合内增删改查，还支持多集合交集，并集，差集。

内部编码：

* intset : 整数集合，当集合中的元素都是整数且元素个数小于 `set-max-intset-entries`配置（默认512）时。
* Hashtable: 不满足intset条件时 就用 hashtable 实现。

#### 集合内操作

1. 添加元素

   ```
   sadd key element [element ....]
   ```

2. 删除操作

   ```
   srem key element [element ...]
   ```

3. 计算元素个数

   ```
   scard key 		#时间复杂度为O(1),不会遍历集合所有元素
   ```

4. 判断元素是否在集合中

   ```
   sismember key element
   ```

5. 随机从集合中返回指定个数的元素

   ```
   srandmember key [count]
   ```

   count 是可选参数，不写默认为1

6. 从集合中随机弹出元素(随机删除)

   ```
   spop key [count]
   ```

   弹出后会删除元素

7. 获取所有元素

   ```
   smembers key
   ```

   会遍历整个key ，慎用

#### 集合间操作

1. 交集

   ```
   sinter key [key ...]
   ```

2. 并集

   ```
   suinon key [key ...]
   ```

3. 差集

   ```
   sdiff key [key ...]
   ```

4. 将交集，并集，差集结果保存

   ```
   sinterstore destination_key key [key ...]
   suinonstore destination_key key [key ...]
   sdiffstore destination_key key [key ...]
   ```

   将结果保存在 destination_key 中

#### 使用场景

1. tag 标签

```
sadd = Tagging (标签)
spop/srandmember = random item (随机生成数，抽奖)
sadd + sinter = Social Graph(社交需求)
```



### zset(sorted set)

  同set一样，不同的是每个元素都会关联一个double类型的分数（score）。redis正是通过分数来为集合中的成员进行从小到大的排序。

zset的成员是唯一的,但分数(score)却可以重复。

内部编码

* ziplist : 压缩列表，当集合中的元素个数小于 `zset-max-ziplist-entries`配置（默认128）且 每个元素的值都小于`zset-max-ziplist-value`配置（默认64字节）时，redis 使用 ziplist 实现。
* skiplist : 当ziplist 不满足条件时，有序集合会使用skiplist 实现。

#### 集合内操作

1. 添加成员

   ```
   zadd key score member [score member ...]
   ```

   Redis 3.2后为 `zadd`添加了4个选项：

   * nx : member 必须不存在，用于添加
   * xx: member 必须存在，用于更新
   * ch : 此次操作后，有序集合元素和分数发生变化的个数
   * incr ： 对 score 做增加，相当于命令 `zincrby`

   有序集合相对于集合提供了排序字段，代价是`zadd`的时间复杂度为 O( log(n) ) , `sadd` 为O(1)

2. 计算成员个数

   ```
   zcard key
   ```

3. 计算成员分数

   ```
   zscore key member
   ```

4. 计算成员排名

   ```
   zrank key member		#分数从低到高排
   zrevrank key member		#分数从高到底排
   ```

5. 删除成员

   ```
   zrem key member [member ...]
   ```

6. 增加成员分数

   ```
   zincrby key increment member
   ```

7. 返回指定排名范围的成员

   ```
   zrange key start end [withscores]   #低到高
   zrevrange key start end [withscores]
   ```

   withscores 参数可选，加上后同时返回成员分数

8. 返回指定分数内的成员

   ```
   zrangebyscore 		key min max [withscores] [limit offset count]
   zrevrangebyscore	key min max [withscores] [limit offset count]
   ```

   limit offset count 限制输出的起始位置和个数

   min max 同时支持开区间（小括号 () ） 和闭区间（方括号[] ）

   -inf 和 +inf 代表无限小和无限大

9. 返回指定分数范围成员个数

   ```
   zcount key min max
   ```

10. 删除指定排名内的升序元素，

   ```
   zremrangebyrank key start end
   ```

11. 删除指定分数范围的成员

    ```
    zremrangebyscore key min max
    ```

#### 集合间的操作

1. 交集

   ```
   zinterstore destination_key numkeys key [key ...] [weights weight [weight ...]] [aggregate sum|min|max]
   ```

   * destination_key ： 结果保存到这个键
   * numkeys : 需要做交集计算键的个数
   * key [key …] ： 需要做交集计算的键
   * Weights weight [weight …] 每个键的权重，做交集计算时，每个键的member 会将分数score 乘以 权重，默认是1
   * aggregate sum|min|max : 计算交集后，分值可以按sum , min , max 做汇总，默认是sum 

2.  并集

   ```
   zunionstore destination_key numkeys key [key ...] [weights weight [weight ...]] [aggregate sum|min|max]
   ```

#### 使用场景

1. 排行榜系统


##键管理

###单个键管理

1. 键重命名

   ```
   rename key newkey
   renamenx key newkey		#newkey不存在时才重命名
   ```

   * 重命名期间会执行del 命令删除旧的键，如果键对应的值比较大，可能会阻塞redis
   * 如果 key 和 newkey 相同，redis 3.2前会提示错误，redis3.2 后返回成功

2. 随机返回一个键

   ```
   randomkey
   ```

3. 键过期

   ```shell
   expire key seconds   #键在seconds 秒之后过期
   expireat key timestamp  #键在秒级时间戳timestamp 后过期
   #redis 2.6之后提供了毫秒级别的过期命令
   pexpire key milliseconds
   pexpireat key milliseconds-timestamp
   ```

   * 如果 key 不存在，返回结果为0
   * 如果过期时间为负值，键被立即删除
   * `persist` 可以将键的过期时间清除
   * **对于字符串类型的串，执行set 命令时会去掉过期时间**。
   * redis 不支持二级数据结构（如 hash , list ） 内部元素的过期功能。如不能对列表的一个元素设置键过期时间
   * `setex` 命令是 `set+expire` 的组合，是原子操作，还减少了一次网络通讯

   查询键的过期剩余时间

   ```
   ttl key
   pttl key 	#毫秒级	
   ```

4. 迁移键

   把数据从一个Redis迁移到另一个Redis

   * dump + restore

     ```
     dump key
     restore key ttl value
     ```

     dump+restore 可以在不同的redis 实例之间数据迁移，整个迁移过程分两步：

     * 在源redis 上，dump 命令会将键值序列化，采用RDB格式
     * 在目标redis 上，restore 命令将上面序列化的值进行复原，其中 `ttl`参数代表过期时间，等于0代表没有过期时间

     注意：

     * 整个迁移过程并非原子性的，需要客户端分步完成
     * 迁移过程开启了两个客户端之间的连接，所以dump 的结果不是在 源redis和目标redis之间进行传输。

   * **migrate**

     ```
     migrate host port key|"" destination_db timeout [copy] [replace] [keys key [key ...]]
     ```

     参数说明：

     * host: 目标redis 的IP地址
     * port :目标redis的端口号
     * key|"" : 在Redis 3.0.6之前只支持一个键，所以此处是要迁移的键，但在Redis 3.0.6之后支持多个键迁移，若要迁移多个键，此处为 空字符串""
     * destination_db：目标 Redis 的数据库索引，如迁移到0号数据库，此处就为0
     * timeout ： 迁移超时时间，单位为毫秒。
     * [copy] : 添加此选项后，迁移后不删除源键
     * **[replace]** : migrate 不管目标redis 是否存在该键都会正常迁移并进行数据覆盖。
     * [keys key [key …]] :  迁移多个键，如：`keys key1 key2 key3`

     migrate 也用于redis实例之间迁移数据，实际上migrate 命令是将`dump + restore + del` 三个命令的组合，且具有原子性，在 Redis3.0.6版本后支持迁移多个键的功能，提高了迁移效率。

     注意：

     * migrate 整个过程是原子执行的，不需要在多个redis 实例间开启客户端，只需要在源redis上执行migrate命令
     * migrate 命令的数据传输直接在源redis 和 目标 redis 之间传输完成。
     * 目标redis 完成`restore`后会发送 `OK`给源redis, 源redis接收后会根据migrate对应的选项是否删除对应的键

   * `move`

     ```
     move key db
     ```

     `move`命令用于Redis内部数据库之间数据迁移。

   ​

### 键遍历

1. 全量遍历键

   ```
   keys pattern
   ```

   `pattern`使用 glob 风格的通配符 

   * `*`  代表哦匹配任意字符
   * `?`代表匹配一个字符
   * `[]代表匹配部分字符，如[1,3]`匹配1和3，`[1-10]`匹配1到10的数字
   * `\x`转义，如要匹配 `* ?`

   不建议在生产环境上使用 `keys` , 如确实需要，建议：

   * 在一个不对外提供服务的redis从节点上执行，这样不会阻塞客户端请求，但会影响主从复制
   * 如果确认键值总数很少，可以执行
   * 使用`scan`命令渐进式遍历所有键，防止阻塞

2. 渐进式遍历

   ```
   scan cursor [match pattern] [count number]
   ```

   * cursor ： 游标，第一次遍历从0开始，每次scan完成后会返回当前游标值
   * match pattern : 类似于 keys pattern的pattern
   * count number : 表明每次要遍历的个数，默认是10，可适当增大

   除`scan` 之外，Redis 提供了面向 hash , set , zset 的扫描遍历命令，解决`hgetall , smembers , zrange `可能产生的阻塞问题，相对应的命令是`hscan , sscan , zscan` , 用法基本和scan类似。

   渐进式`scan`可以解决`keys`可以产生的阻塞问题，但是scan 过程中如果有键发生变化（增，删，改），则遍历效果可能会：

   * 新增的键可能没遍历到
   * 遍历出重复的键等情况

###数据库管理

1. 切换数据库

   ```
   select dbindex
   ```

   Redis一个实例 默认配置有16个数据库，下标是0~15。

   不建议使用一个实例的多个数据库，原因：

   * Redis 是单线程的，使用多个数据库之间会受到影响

   如果非要用到多个数据库的功能，可以在一台服务器上部署多个Redis实例，每个实例用不同端口号区分，这样既能保证多个数据库之间不受影响，也能合理使用多核CPU的资源。

2. flushdb 和 flushall

   用于清楚数据库，两者的区别在于：flushdb只清除当前数据库；flushall 会清除所有数据库

   注意：

   * 防止误操作，万一误操作也要会误操作后快速恢复，留坑
   * 如果当前数据库的键值对比较多，可能会阻塞Redis 。



# 小功能大用处

## 慢查询分析

* 预设阀值

  * `slowlog-log-slower-than    `       预设阀值，单位是微妙，默认是10 000 ， 
    * 如果设置为0，会记录所有命令，如果 <0 ,不记录
  * `slowlog-log-max-len`         设置最多记录慢查询命令条数，实际上命令记录在一个先入先出列表里面，当列表处于最大长度时，会移除最早插入的命令。 

* Redis 修改配置

  * 修改配置文件

  * `config  set` 命令动态修改

    ```shell
    config set slowlog-log-slower-than 20000
    config set slowlog-log-max-len 1000
    config rewrite	#如果要将配置持久化到配置文件中，需要执行 config rewrite ,前提是服务器是用配置文件启动的,且有配置文件的写权限
    ```

* 获取慢查询日志

  ```
  slowlog get
  ```

* 获取慢查询日志当前长度

  ```
  slowlog len
  ```

* 慢查询日志重置( 清理 )

  ```
  slowlog reset
  ```

* 实践

  * `slowlog-log-max-len` 建议在线上调大，例如设置为1000
  * `slowlog-log-slower-than` 默认是10ms ， 需要根据redis并发量调整该值，如果命令执行实践在1ms以上，则redis最多支撑不超过1000，因此对于高并发场景设置为1ms
  * 慢记录查询只记录命令执行时间，不包括网络传输和命令排队时间
  * 如果慢查询比较多，可以定期执行`slow get` 将慢查询日志持久化到其他存储中，比如MYSQL，然后通过可视化界面查询工具查询，如Redis的私有云CacheCloud 提供这样的功能，好的工具可以事半功倍。

## redis shell

### redis-cli

```
redis-cli -help 	
```

下面介绍一些重要参数

* -r

  重复执行命令多次

  ```shell
  redis-cli -r 3 ping		#执行三次ping命令
  ```

* -i 

  必须和`-r` 搭配使用，代表每隔几秒执行一次命令。注意单位是秒

  ```shell
  redis-cli -r 3 -i 1 ping      #执行3次，没猜错执行间隔1秒
  redis-cli -r 3 -i 0.01 ping   #执行3次，没猜错执行间隔10毫秒
  redis-cli -r 10 -i 1 info | grep used_memory_human	#每隔一秒输出内存使用量，一共执行3次。
  ```

* -x

  代表从标准输出stdin 读取数据作为 redis-cli 的最后一个参数

  ```
  echo "world" | redis-cli -x set hello 
  ```

* -a

  如果redis配置了密码，用这个选项后不用手动输入 auth 命令

* `--scan 和 --pattern`

  指定扫描模式的键，相当于`scan`命令

* `--slave`

  把当前客户端模拟为当前Redis 的从节点，可以获取当前Redis 节点的更新操作。

* `--rdb`

  请求 Redis 实例生成 RDB 持久化文件，保存在本地。可用作持久化文件定期备份。

* `--bigkey`

  使用`scan`命令找到内存占用比较大的键值，这些键可能是系统的瓶颈。

* `--eval` 执行指定 lua 脚本

* `--latency` 检测网络延时，有三个命令

  * `--latency`  测试客户端到目标Redis 的网络延迟
  * `--latency-history` 执行结果只有一条，用与分时段了解延迟信息，默认是15秒输出一次，可以通过 `-i`控制间隔时间
  * `--latency-dist` 使用统计图表的方式从控制台输出

* `--stat` 实时获取Redis重要的统计信息，虽然`info`的信息更全，但`--stat`能实时看到些增量信息

* `--raw 和 --no-raw`

  ```
  redis-cli set hello "你好"
  redis-cli get hello #返回二进制格式
  redis-cli --raw get hello
  ```

### redis-server

* `--test-memory` 检测当前操作系统能否稳定的分配指定的内存容量给redis

  ```
  redis-server --test-memory 1024   #检测当前操作系统能否提供1G内存给Redis
  ```

  检测时间比较长，当输出`passed this test`说明检测完毕

### redis-benchmark

​	为redis 做基准测试。

* `-c`(client)

  代表客户端的并发数，默认是50

* `-n` (requests)

  代表客户端请求总量，默认是100 000

  ```
  redis-benchmark -c 100 -n 20000 	#代表100个客户端同时请求redis，一共执行20000次。
  ```

* `-q`显示requests per second 信息

  ```
  redis-benchmark -c 100 -n 20000 -q	
  ```

* `-r`插入随机键

  `redis-benchmark` 执行后会有三个键存在redis中，如果想插入更多随机键，使用-r

  ```
  redis-benchmark -c 100 -n 20000 -r 10000
  ```

  -r 后的10000代表对只对后四位做随机处理，不是随机个数。

* `-p`

  每个请求 pipeline 的数据量

* `-k`代表客户端是否使用 keepalive , 1为使用，0为不使用，

* `-t` 对指定命令做基准测试

  ```
  redis-benchmark -t get,set -q
  ```

* `--csv` 将执行结果按csv输出，便于后续处理，如导出excel 等

  ```
  redis-benchmark -t get,set --csv
  ```

## pipeline

客户端执行一条指令分为4个过程：

1. 发送命令
2. 命令排队
3. 命令执行
4. 返回结果

其中 1 和4 的时间称为 Round trip time ( RTT , 往返时间 )

​	redis提供了批量操作可以有效节省RTT，但大部分命令是不支持批量操作的。

​	Pipeline 机制能改善上述问题，将一组 redis 命令组装，然后通过一次 RTT 传输给 Redis ，再将这组命令的执行结果按顺序返回给客户端。

​	虽然更多实际情况通常使用高级语言客户端中的 pipeline 。

原生批量命令与Pipeline 命令对比

* 原生批量命令是原子的，pipeline 不是原子的
* 原生批量是一个命令对应多个key , pipeline 支持不同命令
* 原生批量是redis 服务器实现，而pipeline 需要服务器与客户端共同实现。

每次pipeline的命令个数要有节制，否则一方面会增大客户端等待时间，二是会造成网络阻塞。

## 事务与Lua

### 事务

​	Redis提供了简单的事务功能，将一组要执行的命令放在`multi 和 exec`之间， `multi` 代表事务开始，`exec`代表事务结束，他们之间的命令是原子执行的。

```shell
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set a 3
QUEUED
127.0.0.1:6379> set b 4
QUEUED
127.0.0.1:6379> exec
1) OK
2) OK
```

在multi  和 exec 之间的命令输入后命令并没有开始执行，只有执行exec后，才会顺序执行之间的命令。

如果要停止事务功能，用`discard` 代替 `exec`命令即可。

如果事务中的命令出现错误：

1. 命令错误

   如命令写错等语法错误，会造成整个事务无法执行，对原数据不影响

2. 运行上错误

   发生运行时错误时，redis **不支持回滚**功能

   有些场景需要在事务前，确保事务中的key 没被其他客户端修改过，才执行，否则不执行。Redis提供了watch命令来解决这类问题。



redis 只提供了简单的事务功能，之所以简单，是因为不支持回滚特性，同时无法实现命令之间的逻辑特性。此时需要用Lua脚本实现更强大事务功能。