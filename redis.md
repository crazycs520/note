****Redis** is an in-memory , non-relational(NoSQL)  database; used as a database, cache and message broker.**

特点：

- 单线程机制
- 支持5种数据类型：string，hash(散列) , lists , sets , zset(sorted sets），每种类型有不同的内部编码实现，

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

 键值本身又是一个键值对结构，是string 类型的 field 和 value 的映射表，适合存储对象

每个 hash 可以存储2^32 -1 键值对。

内部编码有3种

* ziplist : 压缩列表，节省内存，但性能下降
* hashtable :  读写速度快，时间复杂度为 O(1)
  * 当field 个数比较少，且没有大的value时，内部编码为ziplist
  * 当value大于64字节，内部编码由ziplist自动变为hashtable
  * 当field 超过512个时，内部编码由ziplist自动变为hashtable
* quicklist : 结合前两种的优势



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


###List

 字符串列表，按照插入顺序排序，可以添加一个元素到列表头或尾（左或右）。

列表最多存储 2^32 -1 元素

内部编码：

* ziplist ： 压缩表
* linkedlist: 链表
  * 当元素个数过少且没有大元素时，为ziplist 
  * 当元素个数超过512或某个元素超过512时，变为linkedlist
* quicklist


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

# 客户端

```shell
client list     #列出与redis服务端相连接的所有客户端连接信息
info clients    #列出连接总数，最大输出缓冲和输入缓冲，执行速度比client list 快
info memory
info stats
```

1. 输入缓存：qbuf , qbuf-free 分别指输入缓冲的总容量和剩余容量
   * 每个客户端的输入缓冲区不能超过1G，超过会关闭连接
   * 输入缓冲区不受 maxmemory 控制，如果已经存储了2G数据，输入缓冲使用了3G，超过了maxmemory 4G的限制，可能会产生数据丢失，键值淘汰，oom等情况。
   * 实际中输入缓冲区出现问题的概率较低，主要注意减少 bigKey ，减少redis阻塞，合理的监控警报。
2. 输出缓冲 ：obl , oll , omen : obl固定缓冲区长度， oll动态缓冲区列表长度，omen 使用字节数。
   * `client-output-buffer-limit <class> <hard limit> <soft limit> <soft seconds>` 
     * `<class>` 类型分三种：normal  , slave , pubsub ,表示普通客户端，slave 复制客户端，发布订阅客户端。
     * `hard limit` 输出缓冲指超过此限制客户端立即关闭
     * `soft limit soft limit` 指输出缓冲超过 soft limit  且持续超过 soft time ， 客户端会关闭。
   * 输出缓冲也不受 maxmomery 限制，不过可以通过上述配置来关闭一些连接。
   * 如何预防输出缓冲出现异常？
     * `info clients`命令监控输出缓冲区列表 最大对象数，超过阈值及时处理。
     * `client-output-buffer-limit normal 20mb 10mb 120`
     * 适当增大slave 客户端的输出缓冲，在高并发的情况下避免输出缓冲溢出被kill导致复制重连接
     * 谨慎使用monitor 命令
     * 客户端存活状态：age ， idle ： 已经连接时间和最近一次空闲时间
     * 限制`maxclients ` ` timeout` 参数，如果客户端idle时间超过timeout ，就会被关闭。
3. 客户端的相关配置
   *  timeout
   *  maxclients
   *  tcp-keepalice : 建议为60 ，则redis会每60秒对创建的TCP连接进行活性检测，防止大量死连接。
   *  tcp-backlog : 默认是511，但会受系统配置`/proc/sys/net/core/somaxconn`影响 
     * `echo 511 > /proc/sys/net/core/somaxconn`
4. 客户端常见异常
   * 无法从连接池获取连接：
     * 客户端原因：高并发下连接池设置过小，或者客户端没有及时释放连接，或者存在慢查询，导致归还连接的速度慢
     * 服务端原因：可能在执行过程中阻塞。
   * 客户端读写超时：
     * 读写超时时间设置过短
     * 命令本身比较慢
     * 客户端与服务端的网络连接
     * redis自身阻塞
   * 客户端连接超时
     * 连接超时时间设置过短
     * redis阻塞
     * 网络异常
   * 客户端缓存异常
     * 输出缓冲区慢
     * 长时间闲置服务被断开
     * 不正常的并发读写
   * Lua脚本正在执行
   * Redis正在加载持久化文件
   * Redis使用内存超过 maxmemory 配置
   * 客户端连接数过大
5. 实例分析
   1. redis内存陡增
      * `dbsize` 查看主节点，从节点键个数
      * `info clients` 查看客户端缓冲区是否异常
      * `redis-cli client list | grep -v "omen=0"` 查看具体哪个连接异常
      * `client kill ` 关闭异常连接。

# 持久化

## RDB

RDB持久化是把当前进程数据的内存生产快照保存到硬盘。

* `bgsave` 后台手动执行RDB，执行流程如下：
  * 执行 bgsave 命令，redis父进程判断有无子进程正在执行，如 RDB/AOF子进程，如果有，则返回。
  * redis父进程fork子进程，fork操作过程中父进程会阻塞，可以通过`info stats`命令查看 `latest_fork_usec` 最近一个fork操作的耗时，单位微妙。
  * fork 完成后bgsave命令返回，不再阻塞父进程。
  * 子进程创建RDB文件，根据父进程内存生成快照文件，完成后对原文件进行原子替换。`lastsave`命令可以获取最后一次生成RDB的时间
  * 子进程发送信号给父进程表示完成，父进程更新统计信息，具体见 `info persistence` 下的rdb_*命令。
* RDB文件处理
  * 建议开启`rdbcompression yes` 参数对RDB文件进行LZF算法压缩，方便保存和网络传输。
* RDB的优点
  * 是一个压缩的二进制文件，适合备份，全量复制的场景
  * redis加载RDB文件恢复数据比AOF快
* RDB缺点
  * 属于重量级操作，频繁执行成本太高
  * 不能做到实时持久化/秒级持久化
  * 老版本redis可能无法兼容新版本的RDB文件格式。

## AOF

AOF（append only file ）以独立日志的方式记录每次写命令，重启时重新执行AOF中的命令达到数据恢复的效果。

* 开启AOF : `appendonly yes` ， 默认不开启。AOF的工作流程：
  * 所有的写命令追加到aof_buf 缓冲区。
  * AOF根据`appendfsync`参数控制向硬盘做同步操作
  * 随着AOF越来越大，要定期对AOF文件进行重写
  * redis服务器重启时可以加载AOF文件进行数据恢复。 
* `appendfsync` 文件同步策略：
  * always : 命令写入aof_buf后调用fsync同步磁盘
  * everysec ： fsync 由专门的线程每秒调用一次。（最常用），服务器宕机后最多丢失2秒的数据。不是1秒。
  * no : 同步硬盘由操作系统负责，通常同步周期最长30秒。
* 重写机制
  * 进程内已超时的数据不再写入文件
  * 合并多条命令，删除无效命令
  * 更快的被redis加载
* AOF重写机制触发
  * 手动触发 ： `bgrewriteaof` 命令
  * 自动触发 ： 根据
    * `auto-aof-rewrite-min-size`  ： 表示AOF重写时AOF文件最小体积，默认64M
    * `auto-aof-rewrite-percentage`  ：代表当前AOF文件空间和上次重写后AOF文件空间的比值。
    * 触发时机 = aof_current_size > auto-aof-rewrite-min-size && ( aof_current_size - aof_base_size ) / aof_base_size >= auto-aof-rewrite-percentage 。
    * aof_current_size 和  aof_base_size 可以在info persistence 统计信息中查看。
* AOF重写流程：
  * 执行AOF重写请求 ，如果有正在执行的子进程，则返回
  * 父进程fork 子进程，开销等同于bgsave 
  * 父进程fork 完后继续响应其他命令，所有修改命令依然写入 aof_buf 缓冲并根据 appendfsync 策略同步到磁盘，保证原有的aof文件正确性。
  * 由于fork操作用了写时复制技术，redis用"AOF重写缓冲区" 保存这部分新数据。防止新的AOF文件生产期间丢失这部分数据。
  * 子进程根据内存快照按照命令合并规则写入新的AOF文件，每次批量写入硬盘数据量由`aof-rewrite-incremental-fsync` 控制，默认32MB , 防止单次刷盘数据过多造成硬盘阻塞
  * 新AOF写入完成后，子进程发送信号给父进程，父进程更新统计信息，具体见 info persistence 下的aof_*相关统计
  * 父进程把AOF重写缓冲区里面的数据写入新的AOF文件
  * 新AOF文件代替旧文件，完成AOF重写
* 重启加载
  * AOF持久化开启且存在AOF文件时，优先加载AOF文件
  * AOF关闭或AOF文件不存在时，加载RDB文件。

## 问题定位和优化

* fork操作

  * RDB和AOF重写都会用到fork操作，虽然有写时复制技术，但是会复制父进程的空间内存页表。
  * fork耗时问题：正常情况下fork的耗时是每Gb耗时20毫秒左右，可以在 info stats 统计中查看`latest_fork_usec` 获取最近一次fork操作耗时，单位，微妙
  * 如何改善fork耗时：
    * 尽量采用物理机或高效支持fork的虚拟技术，避免使用Xen
    * 控制redis实例的最大可用内存，fork操作和内存成正比，建议线上控制在10GB以内
    * 合理配置Linux内存分配策略
    * 降低fork操作频率

* 子进程开销监控和优化：子进程负载RDB和AOF重写，主要涉及CPU，内存和硬盘

  * cpu

    * 子进程把进程内的内存分批写入文件，这属于cpu密集型操作，通常子进程对单核cpu使用率接近90%
    * cpu消耗优化：开启了RDB和AOF的redis实例不要绑定单核CPU。
    * 不要和其他cpu密集型应用部署在一起

  * 内存

    * Linux有写时复制技术，主要消耗有AOF重写缓存

    * 如果部署了多个redis实例，保证同一时间只有一个子进程在工作。

    * 避免大量写入时做子进程重写操作。

    * Linux关闭THP（Tansparent Huge Page） ,  默认开启。开启时可以加速fork创建速度，但是会大幅增加重写期间父进程内存消耗。所以建议关闭。

      * ```
        sudo echo never > /sys/kernel/mm/transparent_hugepage/enabled
        ```

  * 硬盘

    * 子进程主要是把内存数据写入磁盘，肯定对磁盘有写入压力。根据redis重写AOF和RDB文件写入的数据量，结合系统工具如 sar , iostat , iotop 等分析重写期间的硬盘负载情况。
    * 不要和其他高硬盘负载服务部署在一起，如存储服务，消息队列
    * AOF重写期间会消耗大量IO, 可以开启配置`no-appendfsync-on-rewrite` ,默认关闭。表示AOF重写期间不做Fsync操作。开启后极端情况下可能导致丢失整个AOF重写期间的数据，需要更具数据安全性决定是否配置。
    * 当开启AOF功能的redis在高流量写入场景时，如果是机械硬盘，则此时瓶颈主要在AOF同步硬盘上
    * 对于单机配置多个redis实例的情况，可以配置不同的实例分盘存储AOF文件，分摊磁盘写入压力

* AOF追加阻塞

  * 开启AOF持久化操作时，通常同步硬盘策略是everysec ， redis用另一条线程每秒执行fsync操作同步磁盘。当系统磁盘繁忙时，会造成redis阻塞，分析如下：
    * 主线程负责写入AOF缓冲区
    * AOF每秒执行一次同步磁盘操作，并记录最近一次同步时间
    * 主线程对比上次AOF同步时间：
      * 如果距上次同步成功2秒内，主线程直接返回
      * 如果距上次同步成功超过2秒，主线程阻塞，直到同步完成。
  * everysec   磁盘同步策略最多每秒丢失2秒的数据，不是1秒。
  * 如果fsync缓慢，会导致redis主线程阻塞。

## 多实例部署

`info persistence` 

当一台机器部署多个redis实例，多个实例都开启AOF重写后，彼此之间会产生对CPU和IO的竞争。

采取的策略是通过外部程序轮询控制AOF重写操作的执行。

# 复制

## 配置

* `slaveof {masterHost} {masterPort}` : slaveof 命令本身是异步命令，节点只保存主节点信息后返回。后续复制流程在节点内部异步执行
* `info replication` 查看复制相关状态
* `slaveof no one` 断开复制
* 安全性，主节点可能配置了 requirepass 参数进行密码验证，这是所有客户端访问必须使用auth 命令进行校验。从节点的复制需要配置从节点的 masterauth 参数与主节点的密码保持一致，才能发起复制流程。
* 从节点默认是只读。不建议修改。配置参数: `slave-read-only=yes`
* 传输延迟：`repl-disable-tcp`-nodelay参数控制是否关闭TCP_NODELAY，默认关闭：
  * 关闭时，主节点产生的命令数据无论大小都会及时的发送给从节点，这样延迟会减小，但增加了网络带宽；适用于主从机之间网络良好的场景。如同机架或同机房
  * 开启时，主节点会合并较小的TCP包而节省带宽，默认发送时间取决于Linux内核，一般默认40ms 。适用于主从机网络环境复杂或带宽紧张的场景，如跨机房部署。

## 拓扑

1. 一主一从，当写命令并发量高且需要持久化时，可以只在从节点上开启AOF 。需要注意的是，主节点关闭持久化后，如果主节点脱机要避免自动重启，因为主节点没有持久化数据导致重启后数据为空，这时从节点继续复制主节点会导致从节点数据也被清空。安全的做法是从机执行 `slaveof no one` 断开复制链接，再重启主机点。
2. 一主多从，适用于读较多的场景。对于写并发量也高德场景，多个从节点会导致主节点写命令的多次发送给从机导致过度消耗网络带宽，也加重了主节点的负载。
3. 树状主从结构：降低主节点的压力，缺点是比较复杂。

## 原理

* 复制过程有6个过程：

  1. 保存主节点的信息
  2. 从节点内部通过每秒运行的定时任务维护复制相关逻辑，发现新的主节点后，尝试与主节点建立网络连接，建立socket套接字。
  3. 发送ping命令，如果没有收到主节点恢复的pong 或者超时，从节点会断开复制连接，下次定时任务会发起重新连接。
     * 检测主从机之间的网络套接字是否可用
     * 检测主节点是否可用
  4. 权限验证。如果主节点设置了requirepass 参数，则需要密码验证。
  5. 同步数据集，首次建立复制的场景会先进行一次全量复制。
  6. 命令持续复制。

* 数据同步，psync命令完成主从同步数据

  * 全量复制：用于第一次复制的场景。
  * 部分复制

* psync完成需要：

  * 主从节点各自的复制偏移量
  * 主节点的复制积压缓冲区
  * 主节点的运行ID

* 可以通过对比主从节点的复制偏移量判断主从数据节点数据是否一致。

* 复制积压缓冲区：保存在主节点上的一个固定队列，默认是1M，主节点响应写命令时，不但会把命令发送给从节点，还会写入复制积压缓冲区。

* 主节点ID ： redis节点启动后都有一个40位16进制的字符串作为运行ID 。如果用IP+PORT来识别主节点，那么当主节点重启变更了数据集后是不安全的。故当运行ID变化后会从节点会做全量复制。

  * redis重启后运行ID会改变，如果不想改变运行ID , 可以用命令debug reload 命令重启。

* psync命令：用来完成全量复制和部分复制功能。命令格式：`psync {runId} {offset}` ,从 第一次不知道runI时用 ? 代替，offset 第一次是-1 ， 代表全量复制。

  * 主节点回复：
    * +FULLRESYNC {runId} {offset} ，从节点执行全量复制流程
    * +CONTINUE ，从节点将触发部分复制
    * +ERR ，说明主节点版本低于redis 2.8 , 不能识别 psync命令。

* 全量复制：从节点第一次复制是全量复制，流程如下

  * 从节点发送 `psync ? -1` 

  * 主节点恢复+FULLRESYNC {runId} {offset}

  * 从节点保存runid  和偏移量 offset 

  * 主节点执行 bgsave 保存RDB到本地

  * 主节点发送RDB文件给从节点

    * 如果数据量较大，RDB从文件创建到完成传输的总耗时，如果总时间超过repl-timeout 所配置的值（默认60秒）， 从节点会放弃接受RDB文件并清理已下载的临时文件，导致全量复制失败。
    * 所以对于数据量较大的节点，要调大 `repl-timeout` 

  * 从节点开始接受RDB到接受完成，主节点任然响应读写命令，且把写命令保存在**复制客户端缓冲区内**。当从节点加载RDB文件完成后，主节点再把缓冲内的数据发送给从节点，保证主从之间的一致性。

    * 高流量写入场景下很容易造成主节点复制客户缓冲区溢出，默认配置为 

      ```
      client-output-buffer-limit slave 256mb 64mb 60
      ```

      超出缓冲后会关闭连接导致全量复制失败，故该参数要进行合适的调整。

  *  从节点接收完来自主节点的全部数据后，会清除自身旧数据

  * 从节点开始加载RDB文件

  * 从节点加载完RDB后，如果从节点开启了AOF持久化功能，则立即执行bgrewriteaof操作。

* 部分复制 ： 发送`psync {runId} {offset}` 命令

  * 主节点核对 runId 是否一致
  * 主节点会在复制积压缓冲区内存这部分根据offset 查找数据，找到就发送.

* 心跳判断机制

  * 主节点默认每10秒对从节点发送ping命令，判断从节点的存活性和连接状态，可以配置`repl-ping-slave-slave-period` 控制发送频率。
  * 从节点在主线程中每隔一秒发送replconf ack {offset} 命令给主节点，作用如下
    * 实时监测主从节点的网络状态
    * 上报自身复制偏移量，检查复制数据是否丢失，如果丢失，则要从主节点的复制积压缓冲区重新拉取数据
    * 实现保证从节点的数量和延迟性功能。
  * 主节点根据replconf 命令判断从节点超时时间，体现在 info replication 统计中的lag查看。lag表示与从节点最后一次通信延迟的秒数。正常延时在0，1之间。如果超过`repl-timeout` 的配置（默认60）秒，则判定从节点下线断开复制客户端连接。

##开发运维中的问题

1. 读写分离 : 分摊主节点的读流量
   * 复制数据延迟
   * 读到过期数据，惰性删除和定时删除
   * 从节点故障
2. 主从配置不一致
   * 主节点可以关闭AOF而在从节点开启。
   * 但是内存相关的配置必须一致
3. 规避全量复制
   * 第一次建立复制，这不可避免，应尽量在低峰时操作，
   * 节点运行ID不匹配：如果主节点故障重启后，从节点发现主节点运行ID不匹配，会认为自己复制的是一个新的主节点从而进行全量复制。这种情况应该从架构上避免，如提供故障转移，主节点故障后，提升从节点为主节点或者采用支持自动故障转移的哨兵方案或集群方案。
   * 复制积压缓冲区不足：在写流量高德场景下，应增大复制积压缓冲区。`repl-backlog-size` ，默认是1mb
4. 规避复制风暴
   * 避免大量从节点对同一主节点或同一台机器的多个主节点短时间内发起全量复制的过程。
   * 采用树状结构，减轻主节点负担，但带来了运维的复杂性
   * 单机器复制风暴
     * 把主节点分散到多台机器上
     * 主节点故障后提供故障转移



#  理解内存

##内存消耗

`info memory`

```shell
127.0.0.1:6379> info memory #查看内存使用统计
used_memory:1014304		#redis 内存分配的内存总量
used_memory_human:990.53K
used_memory_rss:2088960	#从操作系统的角度显示redis进程占用的物理内存
used_memory_rss_human:1.99M
used_memory_peak:1014304	#内存使用最大值
used_memory_peak_human:990.53K
used_memory_lua:37888		#lua 引擎消耗的内存
used_memory_lua_human:37.00K
mem_fragmentation_ratio:2.06 	#内存碎片率
mem_allocator:libc		#redis 使用的内存分配器
```

mem_fragmentation_ratio：used_memory_rss / used_memory 

* 当 > 1 时，used_memory_rss - used_memory   如果相差的很大，说明碎片率严重 ；  
* **当 < 1 时， 这一般出现在操作系统把redis 内存交换( swap )到 硬盘导致，需要特别注意**

### 内存消耗的划分

* 自身内存
* 对象内存
  * Key 对象，故避免使用过长的键
  * value 对象
* 缓冲内存
  * 客户端缓冲
    * 输入缓冲，不可控，最大为 1G
    * 输出缓冲，通过`client-output-buffer-limit` 参数控制
  * 复制积压缓冲，用于 实现部分复制功能，根据`repl-backlog-size` 参数控制，默认1M，可以设置较大，如100M，因为所有主节点只有一个，所有从节点共享此缓冲
  * AOF缓冲，用于AOF重写期间保存近期写入命令，不可控，取决于AOF重写时间和写入命令量
* 内存碎片

**used_memory =  自身内存 + 对象内存 + 缓冲内存**

**used_memory_rss - used_memory = 内存碎片**

**内存碎片**

以下操作容易出现高内存碎片

* 频繁的更新操作，如 append , setrange
* 大量过期键删除

解决方法

* 数据对其，尽量采用数字类型或者固定长度的字符串
* 安全重启，常用于高可用架构，如sentinel 或 cluster ， 把内存过高的主节点转换为从节点。

**子进程内存消耗**

主要指AOF/RDB 时创建的子进程。因为linux 具有写时复制 （ copy-on-write） ，父子进程会共享相同的内存页。

注意，建议关闭linux 中的 transparent huge page ( THP )  机制，虽然开启会加快 fork子进程的速度，但 copy-on-write 期间，复制内存页的单位会从 4K 变为 2M , 如果写入命令较多，可能会造成过度内存损耗。

## 内存管理

**设置内存上限**

`maxmemory` 

* 用于缓存场景，当内存超出maxmemory 后，使用LRU等策略释放内存
* 防止所用内存超过服务器物理内存

**动态调整内存上限**

```
config set maxmemory 4GB
```

**内存回收策略**

* 删除到达过期时间的键对象
  * 惰性删除
  * 定时任务删除，默认每秒运行10次（通过配置hz 控制）
* 内存使用超过 maxmemory 触发内存溢出控制策略，由`maxmemory-policy`参数控制，`config set maxmemory-policy {policy}`动态配置
  * noeviction : 默认策略，不删除任何数据，拒绝所有写入操作并返回客户端错误信息 OOM
  * volatile-lru : 根据LRU算法删除超时属性的键，直到腾出空间
  * allkeys-lru : 根据LRU算法删除所有键，直到腾出空间
  * Allkeys-random : 随机删除所有键，直到腾出空间
  * volatile-random : 随机删除超时属性的键，直到腾出空间
  * volatile-ttl : 根据键值对象的ttl属性，删除最近要过期的数据，如果没有，退回到noeviction

当redis一直工作在内存溢出的状态下且设置非noeviction 策略时，会频繁触发回收内存操作，影响redis性能。

## 内存优化

**redisObject对象**

value（值）对象由redisObject 来封装

```C
typedef struct redisObject {
    // 对象的类型，string ，hash , list , set , zset 
    unsigned type:4;
    // 内部编码类型
    unsigned encoding:4;
  // LRU计时时钟:记录对象最后一次访问时间，辅助LRU算法
    unsigned lru:22; /* lru time (relative to server.lruclock) */
    // 引用计数器
    int refcount;
    // 数据指针，如果是整数或是长度<=39字节的字符串，内部编码为embint，则直接存储，这样只要一次内存分配操作。
    void *ptr;	
} robj;
```

可以使用 `scan + object idletime` 命令查询长时间不访问的键进行清理

**如果是整数或是长度<=39字节的字符串，内部编码为embint，则直接存储，这样只要一次内存分配操作。**

**缩减键值对象**

* key :键值越短越好，如 `u:{uid}:fs:nt:{fid}`
* value ，存储时去掉不必要的属性，选择高效的序列化工具
  * golang序列化工具：https://github.com/smallnest/gosercomp ,可以选择**MessagePack**和**gogo/protobuf**都可以;
  * 如果存储json 等文本数据时，内存紧张情况下可以考虑压缩后存入，压缩工具推荐：google的 [snappy](https://github.com/google/snappy)

**共享对象池**

redis 内部有 [0-9999]的整数对象池，创建大量整数类型时redisObject 存在内存开销。除了整数值对象，list ,  hash , set , zset 内部元素也可以使用整数对象池。

当设置了 	`maxmemory`并启用了LRU相关的淘汰策略算法时，redis  会禁止使用共享对象池。

**字符串优化**

Redis 自己实现了 简单动态字符串( simple dynamic string   , SDS) 字符串结构。

```C
struct SDS{
  int len;
  int free;
  char buf[];
};
```

* O(1)时间复杂度获取字符串长度，未用长度，已用长度
* 可用来保存字节数组
* 预分配机制，故尽量少用 append , setrange  ，改为直接用 set 修改字符串，避免预分配带来的内存碎片化
  * 第一次创建len 属性等于实际大小
  * 修改后如果已有free数据空间不足1M，每次预分配1倍容量，比如：原有 `len=60,free=0`; 追加60 byte后， 预分配120 byte :`len=60+60+120`
  * 如果已有free数据空间大于1M，每次预分配 1M 。
* 惰性删除机制，字符串缩减后不释放，作为预分配空间保留

**字符串重构**

比如把 json 数据用hash 结构，同时使用hmget , hmset 修改部分字段。

合理设置内部编码转换的限制参数，比如某个属性的字符串最长长度是65，而 `hash-max-ziplist-value`默认是64,修改此参数为64 用ziplist 编码可以节省内存，但不要将此限制设置太大，因为ziplist 编码在长度过长后性能会下降。

**编码优化**

**控制编码类型**

编码类型转换只能是从小内存转向大内存，如果重新设置参数后，需要 redis 重启加载数据才能完成转换。

**hash**

`hash-max-ziplist-value` 和  `hash-max-ziplist-entries` 

* ziplist 
* hashtable

**list**

3.2版本前

`list-max-ziplist-value` 和  `list-max-ziplist-entries` 

* ziplist
* linkedlist

3.2版本及以后，新的编码：**quicklist**

`list-max-ziplist-size` ：表示最大压缩空间和长度，最大空间使用 [-5-1] , 默认是 -2 ， 表示8K，正整数表示最大压缩长度

`list-compress-depth` : 表示最大压缩深度，默认为0 ，不压缩

**set**

`set-max-intset-entries`

* intset
* hashtable

**zset**

`zset-max-ziplist-value` 和  `zset-max-ziplist-entries` 

* ziplist
* skiplist



**ziplist编码**

采用内存线性连续的结构，适合存储小对象和长度有限的数据

| zlbytes | 记录整个压缩列表的长度 |
| ------- | ----------- |
| zltail  | 记录距离尾节点的偏移量 |
| zllen   | 记录压缩链表的节点数量 |
| entry-1 | 记录具体的节点     |
| entry-2 |             |
| ...     |             |
| zlend   | 记录链表结尾，1个字节 |

entry结构

| pre_entry_bytes_length |      |
| ---------------------- | ---- |
| encoding               |      |
| Contents               |      |

**intset编码**

内部表现为，存储有序，不重复的整数集

| encoding | 表示整数类型，有三种：int-16 , int-32 , int-64 |
| -------- | ----------------------------------- |
| length   | 表示集合元素个数                            |
| contents | 整数数组                                |

使用时尽量保持整数范围一致，提前预估，防止个别大整数触发集合升级操作，产生内存浪费。

**控制键数量**

不要把大量使用 get/set 这种api , 可以考虑hash结构， 把大量键分组映射到hash结构中，降低键的个数。这种做法非常适合于存储小对象的场景，故注意采用ziplist 编码，采用 hashtable 反而会增加内存消耗；

