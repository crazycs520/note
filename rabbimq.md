# 简介

* AMQP : 高级消息队列协议
* RabbitMQ的消息包含两部分内容：
  * 有效荷载  ： 想要传输的数据
  * 标签（lable） ： rabbitMQ根据标签路由消息。
* 客户端连接RabbitMQ： AMQP信道，其建立在TCP连接上。一个AMQP连接中可以有多个信道。  

## 队列

* AMQP消息路由必须有三部分，生产者把消息发布到交换器上，消息最终到达队列，并被消费者接收。
  * 交换器
  * 队列
  * 绑定
* 队列是AMQP消息通信的基础模块：
  * 为消息提供处所，消息在此等待消费
  * 对负载均衡来说，只需一堆消费者，RabbitMQ会以循环的方式均匀分配发来的消息（不能考虑高低配置机器间的差异性）。
* 消费者通过以下两个方式从特定的队列中接收消息：
  * 通过AMQP的`basic.consume`命令订阅。这样会将信道置为接收模式，直到取消对队列的订阅。消息一到达队列时就自动接收。
  * 如果只想获得单条消息而不是持续订阅，用AMQP的`basic.get`命令实现。不应该把`basic.get`放在一个循环里代理`basic.consume`，这样会影响RabbitMQ的性能。大致上，`basic.get` 命令会订阅消息，获得单条消息，然后取消订阅。消费者应该用`basic.consume`来实现高吞吐量。
* 如果有多个消费者订阅到同一队列上，每个消息只会发送给一个订阅的消费者。队列以循环的方式选接收的消费者。
* 消费者收到每一条消息都必须进行确认。消费者必须通过AMQP的 `basic.ack` 命令显示地向RabbitMQ发送一个确认，或者在订阅队列时将`auto_ack参数设置为true`。
* 如果消费者收到一条消息后，在确认之前与RabbitMQ 断开（即取消订阅），RabbitMQ 会认为这条消息没有分发，然后重新分发给下一个订阅的消费者。
* 如果消费者收到一条消息后没有（忘记）确认，那么RabbitMQ将不会给这个消费者发更多的消息了，因为在上一条消息确认前，RabbitMQ会认为这个消费者并没有准备好接受下一条消息。可以利用这一点，如果消费者处理消息非常耗时，则应用程序可以延迟确认该消息，直到消息处理完成，这样可以防止RabbitMQ持续不断的发送消息导致应用程序过载。
* 收到消息后想明确拒绝而不是确认（比如处理时本机出现错误），怎么办？
  * 把消费者从RabbitMQ断开，这会让RabbitMQ重新把消息发送给下一个消费者。但是这会额外增加RabbitMQ的负担。
  * 在RabbitMQ2.0.0版本之后，可以用AMQP的`basic.reject` 命令来拒绝。如果把`reject` 命令的`requeue`参数设置成 `true`  ， RabbitMQ会将消息重新发送给下一个消费者; 如果设置为 `false` ，RabbitMQ会把消息从消息队列中移除，不会发送给下一个订阅的消费者。也可以通过对消息的确认`basic.ack`来忽略该消息，这对检测到一条格式错误的消息而言非常有用。
    * 。如果把`reject` 命令的`requeue`设置为 `false` ，RabbitMQ会把消息从消息队列中移除，而加入死信`dead letter` 队列，如果应用程序有用到死信队列，则应该使用 `basic.reject` 并把`requeue`参数设置为 false 。
* 如何创建队列？消费者和生产者都能够使用AMQP的`queue.declare` 来创建队列。但如果消费者在同一条信道上订阅了另一个队列的话，就无法声明队列了，必须先取消订阅，将信道设置为`传输`模式。
  * 创建队列时，需要指定队列名称，订阅、绑定队列时也需要队列名称，如果不指定名称，RabbitMQ会分配一个随机名称并在 `queue.declare` 命令的响应中返回。以下是队列设置的一些有用参数：
    * `exclusive` ：如果设置为 true  , 队列将变为私有的，此时只有你的应用程序才能消费队列消息。当要限制一个队列只有一个消费者时有用。
    * `auto-delete` ： 当最后一个订阅者取消订阅时，队列就会自动移除。常用在临时队列中。
  * 如果声明一个已经存在的队列会怎样？只要声明参数完全匹配现有的队列，RabbitMQ什么也不做，返回成功。如果参数不匹配，队列声明会失败。
    * 如果只想检测队列是否存在，可以设置`queue.declare`的passive 选项为 true ，这样队列存在就返回成功，不存在RabbitMQ也不会创建队列而是返回一个错误。
  * 队列应该由消费者还是生产者创建？在不能忍受消息丢失的情况下，可以生产和消费都尝试去创建；如果可以忍受消息丢失，那么可以只让消费者来声明。

## 交换机和绑定

* 想把消息投递到队列，是通过把消息发送给**交换机**来完成的。然后根据**路由键（Routing Key）**，把消息投递到相关的队列。如果消息不匹配任何路由键，消息将进入`黑洞`。
* 交换机的四种类型：
  * direct  
    *   如果路由键匹配，消息就被投递到对应的队列。
    *   服务器必须实现 direct 类型的交换器，包含一个**空白字符串的默认交换器**。当声明一个队列时，他会自动绑定到默认交换器，并以 队列名称作为路由键。可以用以下代码发送消息到声明的队列中（前提是要获得信道）：
      * `channel->basic_publish($msg,'' ,'queue-name')`
        * 第一个参数是消息内容
        * 第二个参数空字符串指定了默认交换器
        * 第三个参数是路由键，这个路由键就是之前声明队列的队列名
    *   可以声明自己的交换器：发送 `exchange.declare` 命令并设置相关参数即可。
  * fanout 
    *  将接收到的消息广播到绑定的队列上。fanout 会把收到的消息投递给所有附加在此交换器上的队列。比如用户上传新图片，要做两件事：清除缓存和奖励用户积分。那么可以把两个队列绑定到图片上传的fanout 交换器上。
  * topic
    * 实现来自不同源头的消息能够到达同一个队列。如日志系统，将不同消息级别：error , info , warning 投递到响应队列，每个队列通过绑定不同的路由键（可以用正则表达式匹配）实现更改一个队列接收不同级别的错误消息。（`*` 可以匹配任意字符，`#`可以匹配所有路由键）
  * headers

## 虚拟机和隔离

* 每个RabbitMQ服务器都能创建虚拟主机（vhost） 。每个vhost 的本质是一个mini版本的RabbitMQ，拥有自己的队列，交换器和绑定，还有**权限机制**。这个以将多个应用程序间实现隔离，将同一Rabbit 的客户区分开，又可以避免队列和交换机的命名冲突。
* RabbitMQ包含默认的`vhost:"/"` ，通过使用缺省的guest用户名和密码就能访问默认的vhost。为安全起见，应该更改它。
* RabbitMQ中的权限控制是以vhost为单位的。
* 考虑架构中通用功能群组，并为他们分配各自的vhost。这样方便迁移。
* vhost 不能通过AMQP协议创建。需要用安装目录中的`rabbitmqctl` 工具创建:`rabbitmqctl add_vhost [vhost_name]` ； 删除一个vhost : `rabbitmqctl delete_vhost [vhost_name]` ; 查看RabbitMQ服务器上运行着哪些vhost ：`rabbitmqctl list_vhosts` 。

## 持久化

* 默认情况下队列和交换器重启后就丢失，因为每个队列和交换机的`durable`属性默认为 false。如果为`true`，服务器重启后会重新创建队列和交换机。
* 想要恢复消息，要持久化消息
  * 把它的`投递模式`（dilivery mode）选项设置为2来把消息标记为持久化
  * 并且发布到持久化交换器，交换器会提交日志后才发送响应。
  * 到达持久化队列 来完成消息持久化。 
* 从持久化队列中消耗一条消息，RabbitMQ会在持久化日志中把这个消息标记为等待垃圾收集。
* 持久化消息对性能有影响，所以只对需要持久化的消息才做持久化。
* AMQP事务：当需要继续处理其他任务前，必须确保代理接收到了消息，并把消息路由给所有匹配的订阅队列时，就需要事务功能。
  * 把信道设置成事务模式，通过信道发送想要确认的消息，之后还有很多其他的AMQP命令，这些命令的执行取决于第一条消息是否发送成功，一旦发送完所有的命令，就可以提交事务了。
  * 事务对性能也有影响，而且会使生产者应用程序产生同步，这样不好。
* 发送方确认模式 和事务相仿，将信道设置成`confirm`模式，而且只能通过重新创建信道来关闭该设置。一旦信道进入`confirm`模式，所有在信道上发布的消息都会指派一个ID号，一般消息被投递给队列后，信道会发送一个确认通知生产者。如果消息和队列是持久化的，确认会在队列将消息写入磁盘后才发出。
  * 发送方确认模式最大的好处是它们之间是异步的，一旦发布了一条消息，生产者应用就可以在等待确认的同时继续发送下一条 。当确认消息最终受到时，生产者应用通过回调方法就会触发来处理该确认消息。
  * 如果RabbitMQ内部发送错误导致消息丢失，RabbitMQ会发送一条`nack`（not acknowledged）消息，告知生产方消息丢失了。
  * 由于没有消息回滚的概念（同事务相比），因此发送方确认模式更加轻量级，同时对Rabbitß性能的影响很小。

## 一条消息的一生

* 发布者代码步骤
  * 连接到RabbitMQ
  * 获取信道
  * 声明交换器
  * 创建消息
  * 发布消息
  * 关闭信道
  * 关闭连接
* 消费者代码步骤
  * 连接到RabbitMQ
  * 获取信道
  * 声明交换器
  * 声明队列
  * 把队列绑定到交换机
  * 消费消息
  * 关闭信道
  * 关闭连接

# 运行和管理RabbitMQ

## 服务器管理

1. 启动节点
   * `rabbitmq-server -detched`   以守护程序的方式方式在后台运行
2. 停止节点
   * 干净的关闭
     * `rabbitmqctl stop` ： rabbitmqctl 会和本地节点通信并指示其干净的关闭。
     * `rabbitmqctl stop -n rabbit@[hostname]` ： 关闭远程节点
3. 重启应用程序
   * 停止RabbitMQ
     * `rabbitmqctl stop_app`
4. Rabbit配置文件
   * 典型地，该配置文件位于`/etc/rabbitmq/rabbitmq.config`  ， 文件位置可以通过rabbitmq-server 的脚本对CONFIG-FILE环境变量进行设置。
   * mnesia 指的是Mnesia 数据库配置选项（Mnesia 是RabbitMQ用来存储交换器和队列元数据）。
   * 每个配置选项的形式： `{[option_name],[option_value]}` ，每个配置选项之间用 `,`  隔开。
   * RabbitMQ中的每个队列，交换器和绑定的元数据（除了消息的内容）都是保存到Mnesia 的。Mnesia 是Erlang 内建的NOSQL型数据库。 
   * Mnesia配置选项
     * Dump_log_write_threshold ：默认值100 ，将日志内容转存到真实数据库文件的频率，指定达到多少条日志后就转存到数据库中。
     * tcp_listeners : 默认值`[{"0.0.0.0",5672}]`  ，定义了RabbitMQ应该监听的非SSL加密通信的IP地址和端口。
     * `ssl_listeners`  : 默认值 空 ， 定义了RabbitMQ应该监听的SSL加密通信的IP地址和端口。
     * `ssl_options`  ：  默认值 空  ， 指定SSL相关选项。
     * `vm_memory_high_watermark` : 默认值 0.4 ，控制RabbitMQ允许小号的内存，0.4=40%
     * `msg_store_file_size_limit` : 默认值 16777216  ，RabbitMQ垃圾收集存储内容之前，消息存储数据库的最大大小，单位，字节
     * `queue_index_max_journal_entries` ：默认值 262144 , 在转存到消息数据库并提交之前，消息存储日志里的最大条目数。

## 请求许可

* RabbitMQ权限工作原理：用户可以连接到RabbitMQ主机的应用程序设置不同级别的权限（读，写，和 配置）。
* 单个用户可以跨越多个vhost进行授权。

1. 管理用户
   * 创建新用户：`rabbitmqctl add_uer [user_name] [passwd]`
   * 删除用户 ： `rabbitmqctl delete_user [user_name]`
   * 查看Rabbit服务器上有哪些用户：`rabbitmqctl list_user`
   * 更改用户密码 ： `rabbitmqctl change_password [user_name] [new_passwd]`
2. Rabbit的权限系统
   * 读，写，配置 三种权限。
     * 读：有关消费消息的任何操作，包括“清除” 整个队列（同样需要绑定操作成功）
     * 写：发布消息（同样需要绑定操作成功）
     * 配置：队列和交换机器的创建与删除
   * 访问控制条目是无法跨越vhost 的。
   * 设置user在vhost上的权限：`rabbitmqctl set_permissions -p [vhost_name] [user_name] ".*" ".*" ".*"` ： 授予某用户在某vhost 的完全权限，命令解析
     * `-p [vhost_name] ` ：这个应用条目应用到vhost_name 的vhost上
     * `user_name` 被授予权限的用户
     * ` ".*" ".*" ".*"` 授予的权限，分别映射到配置，写，读。
       * `".*"` 是正则表达式，它意味着匹配任何队列或者交换器，这是指代所有权限。
   * `rabbitmqctl list_permissions -p [vhost]`   : 列出vhost上的权限
   * `rabbitmqctl list_user_permissions [user_name]` ，列出用户在所有vhost上的权限。
   * 删除user 在 vhost上所有的权限 `rabbitmqctl clear_permissions -p [vhost_name] [user_name]` 

## 检查

1. 查看数据统计
   * 列出队列和消息数目 ：`rabbitmqctl list_queues` : 此命令列出的是默认vhost上的队列和消息
     * `rabbitmqctl list_queues -p [vhost_name]` ：列出特定vhost上的队列和消息数目
     * `rabbitmqctl list_queues name messages consumers memory durable auto_delete` ：列出默认vhost上的更多信息
   * 查看交换器和绑定
     * 获取默认交换机信息：`rabbitmqctl list_exchanges`
     * 获取默认绑定信息：`rabbitmqctl list_bindings `
2. RabbitMQ日志
   * 在文件系统中读日志
     * `LOG_BASE` 环境变量，日志存放在此目录中。此目录中有两个日志文件，RABBIT_NODENAME-sasl.log 和  ，RABBIT_NODENAME.log 。
       * sasl日志是库的集合，作为Erlang-OTP发行版的一部分，为开发Erlang应用程序提供一系列标准。其中之一是日志记录格式。故 当Rabbit 记录Erlang相关信息时，将其写入 RABBIT_NODENAME-sasl.log文件中。
       * RABBIT_NODENAME.log 记录服务器发生的事件。
   * 轮换日志
     * 服务器启动时会重新创建日志并在旧日志文件后添加一个数字。如果想手动转换日志文：
       * `rabbitmqctl rotate_logs suffix`  : suffix通常是数字，如：`rabbitmqctl rotate_logs .1` 添加到被轮换的日志文件的末尾。
   * 通过AMQP实时访问日志
     * 当使用`rabbitmqctl list_exchanges` 列出交换器列表时可以看到有一个 `amp.rabbitmq.log` 的topic 交换器。RabbitMQ把日志信息发布到该交换器上，并以严重级别作为路由键：`error` ，`warning` ， `info` 。
     * 可以创建一个消费者来监听日志并作出相应的反应。

## 修复Rabbit：疑难解答

1. 由 `badrpc  , nodedown , ` 和其他Erlang 引起的问题
   * `badrpc , nodedown` 这样的消息是由Erlang 虚拟机产生的；常发生于使用`rabbitmqctl` 命令。
   * Erlang cookie
     * rabbitmqctl 命令的工作原理： `rabbitmqctl` 会启动Erlang 节点，并从那里使用Erlang 分布式系统尝试连接RabbitMQ节点。这需要两样东西：
       * 合适的Erlang cookie ： Erlang 节点通过交换作为秘密令牌的Erlang cookie 以获得认证。令牌存储在名为.erlang.cookie的文件，该文件通常在用户的home目录下。为了让rabbitmqctl 连接RabbitMQ节点，因此需要共享相同的cookie，如果运行RabbitMQ和执行rabbitmqctl命令的是同一个用户，则不会有问题。但在产品环境中，可能需要创建rabbitmq用户并用此用户运行Rabiit服务器。这意味着必须和rabbitmq用户共享cookie，或者切换到rabbitmq用户执行`rabbitmqctl`。
       * 合适的Erlang节点名称： 当启动Erlang 节点时，可以给他两个互斥的节点名选项，`name` 和 `sname` ，节点名可长可短，这也是sname中s(hort) 的含义。如果用长名启动节点，则会像 `rabbit@hostname.network.tld`这样，如果是短名的话，会像：`rabbitmq@hostname` 这样。短名启动是默认方式。
         * 当用`rabbitmqctl`连接上RabbitMQ时，必须使得这些参数两边都匹配。即都用sname或都用name。
   * Mnesia 和主机名
     * Mnesia是Erlang的内置NoSQL数据库，RabbitMQ用Mnesia 存储队列，交换器，绑定等信息。RabbitMQ启动时做的第一件事就是启动Mnesia数据库。如果Mnesia启动失败，RabbitMQ也会失败，常见原因有：
       * 最常见的是 MNESIA_BASE 目录的权限问题。运行RabbitMQ服务器的用户需要对该文件夹有写权限。
       * 另一个常见原因是：如果主机名改了，或是服务器运行在集群模式下，无法在启动的时候连接到其他节点，这都会导致失败。
         * 为什么要关注主机名呢，因为Mnesia会基于机器的主机名创建数据库schema ，如果主机名改了，那么Mnesia就无法载入旧的schema。
         * 同时记住：RabbitMQ使用rabbit这个单词作为节点名。如果用Erlang sname更改了它，也会出现同样的问题。

# 解决Rabbit相关问题：编码与模式

* 解耦，方便扩展，Rabbit轮询负载均衡。选择合适的编程语言做合适的事，用AMQP提供API。
* 私有队列（声明队列时指定exclusive 参数）和发送确认：使用消息来发回应答，在每个AMQP消息头里有个字段叫做reply_to 。消息的生产者可以通过该字段来确认队列的名称。

# 集群并处理失败

## RabbitMQ集群

* 允许消费者和生产者在Rabbit节点崩溃的情况下继续运行
* 添加更多的节点来线性扩展消息通信吞吐量
* 当一个Rabbit集群节点崩溃时，该节点上队列的消息也会消失。因为RabbitMQ默认不将队列的内容复制到整个集群上。如果想要复制，需要进行配置。

## 集群架构

* RabbitMQ会始终记录一下四种元数据：
  * 队列元数据 ：队列的名称和其属性（是否持久化，是否自动删除）
  * 交换器数据：交换器名称，类型和属性（可持久化等）
  * 绑定元数据：一张简单的表格展示如何将消息路由到队列
  * vhost 元数据： 为vhost中的队列，交换器，绑定提供命名空间和安全属性。
* 单一节点内，RabbitMQ会把所有这些信息存在内存中，同时把标记为可持久化的队列和交换器以及绑定存储到硬盘。以确保重启后的重建。

1. 集群中的队列

   * 不是每个节点都有所有队列的完全拷贝。单一节点只有所有关于队列的信息（元数据，状态和内容）。
   * 在集群中新建队列，集群只会在单个节点而不是所有节点上创建完整的队列信息（元数据，状态，内容）。其他节点只存有队列的元数据和指向该队列的那个节点的指针。
   * 如果重新创建的队列被标记为持久化了，那么在其他节点重新声明时会得到 `404 NOT_FOUND` 错误。
   * 为什么不将队列的内容存储到所有的节点上？存储空间 和 性能。

2. 分布交换器

   * 交换器不同于队列拥有自己的进程，交换器只是一个名称和一个队列绑定列表。
   * 集群上所有节点都拥有所有交换器的所有信息。

3. 内存节点和磁盘节点

   * 集群中可以选择配置部分节点为内存节点，部分为磁盘节点。内存节点速度快~
   * 当在集群中声明队列，交换器，绑定时，这些操作会直到所有节点都成功提交元数据后才返回。
   * RabbitMQ要求在集群中至少有一个磁盘节点，其他的可以是内存节点。
   * 当节点加入或离开集群时，要将变更通知至少到一个磁盘节点。
   * 如果集群中的唯一磁盘几点挂了，那么集群可以继续路由，但不能做一下操作：
     * 创建队列，交换器，绑定
     * 添加用户
     * 更改权限
     * 添加或删除集群节点

4. 镜像队列和保留消息

   ​












