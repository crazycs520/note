* LSM树介绍：

http://www.open-open.com/lib/view/open1424916275249.html

https://juejin.im/post/5a58542b6fb9a01c9064ce44

* badger 一个高性能的LSM K/V store

  http://colobu.com/2017/10/11/badger-a-performant-k-v-store/

* bolt DB 源码分析

  https://lrita.github.io/2017/05/22/boltdb-persistence-3/

  http://www.d-kai.me/boltdb%E4%B9%8Bbucket%E4%B8%80/





# 数据库选型



## mongoDB

三大特色：

1. JSON文档模型，
2. 自带高可用，自动主从切换（副本集）
3. 自带水平分片（分片），内置了路由，配置管理。应用只要连接路由，对应用来说是透明的。

局限

1. 多表关联，仅仅支持 left outer join
2. SQL语句支持：查询为主，部分支持
3. 不支持多表原子事务
4. 不支持多文档原子事务
5. 16MB文档大小限制
6. 不支持中文排序
7. 服务端 JavaScript 性能欠佳

什么时候用mongoDB?

1. 数据量有亿万级，或者需要不断扩容
2. 需要 每秒2000~3000 以上的读写
3. 新应用，需求会变，数据模型不确定
4. 需要多个外部数据资源
5. 大量的地理位置查询