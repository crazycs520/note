* [单机使用 Docker Compose 快速构建集群](https://github.com/pingcap/docs-cn/blob/master/op-guide/docker-compose.md)
* http://chuansong.me/n/720316151966
* 分布式追踪
  * https://blog.openshift.com/openshift-commons-briefing-82-distributed-tracing-with-jaeger-prometheus-on-kubernetes/
  * http://www.jdon.com/48551
  * http://www.cnblogs.com/zhengyun_ustc/p/55solution2.html
  * http://www.cnblogs.com/zhengyun_ustc/p/m2m.html









# tidb相关资料

视频：https://www.youtube.com/watch?v=s_abZtGLlfY&index=15&list=PLx_Mc4dJcQbl4qPWbVu86u6owZeiwsErR

​	https://www.youtube.com/watch?v=xAiuLJtCZOg

​	https://www.youtube.com/watch?v=v7dF5jJuntE

​	https://www.youtube.com/watch?v=dijsN0bddck

​	

http://www.infoq.com/cn/articles/how-to-build-a-distributed-database

https://www.jianshu.com/p/69fd2ca17cfd

https://zhuanlan.zhihu.com/p/33711664

https://zhuanlan.zhihu.com/p/34419461

https://mp.weixin.qq.com/s?__biz=MzI3NDIxNTQyOQ==&mid=2247484187&idx=1&sn=90a7ce3e6db7946ef0b7609a64e3b423&chksm=eb162471dc61ad679fc359100e2f3a15d64dd458446241bff2169403642e60a95731c6716841&scene=4  join优化， pipeline







### 准备

* gRPC
* SQL 如 left join , inner join ...
* index selection , 优先选择区分度大的索引，比如 name 和 sex , 优先选 name 
* 行列混合存储，智能统计使用数据后将数据转为更合适的结构存储，列适合OLAP, 行适合 OLTP
* OLAP 和 OLTP 的 work load 分离
* 只读，不参与投票 的 raft 副本，   供OLAP读，因为OLAP 一般允许一定延迟的数据
* 单元测试覆盖率  http://blog.xcatliu.com/2017/03/12/test_coverage_for_github/
* Falut injection
* Jepsen
* Namazu
* 一切都要自动化~
* 监控利器之 Prometheus——grafana 可视化  https://www.jianshu.com/p/339db34e4afe
* 对比 CockroachDB