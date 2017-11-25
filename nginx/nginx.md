# 简介

## 准备工作

1. 必备软件

   * `yum install -y gcc`
   * `yum install -y gcc-c++`
   * `yum install -y pcre pcre-devel`   : PCRE库，支持正则表达式
   * `yum install -y zlib zlib-devel`   ： 压缩
   * `yum install -y openssl openssl-devel`  : 支持在SSL协议上传输HTTP

2. 内核参数优化

   * 修改 `/etc/sysctl.conf` 

     ```Shell
     #表示进程可以同时打开文件句柄的最大数量
     fs.file-max = 999999
     #开启TCP连接复用功能，允许将time_wait状态的 sockets重新用于新的TCP连接（主要针对time_wait连接）
     net.ipv4.tcp_tw_reuse = 1  
     #表示当keepalive启用的时候，TCP发送keepalive消息的频度（单位：秒）
     net.ipv4.tcp_keepalive_time = 30 
     #如果套接字由本端要求关闭，这个参数决定了它保持在FIN-WAIT-2状态的时间。对端可以出错并永远不关闭连接，甚至意外当机。缺省值是60 秒。2.2 内核的通常值是180秒，你可以按这个设置，但要记住的是，即使你的机器是一个轻载的WEB服务器，也有因为大量的死套接字而内存溢出的风险，FIN- WAIT-2的危险性比FIN-WAIT-1要小，因为它最多只能吃掉1.5K内存，但是它们的生存期长些。
     net.ipv4.tcp_fin_timeout = 15 
     #timewait 套接字数量的最大值
     net.ipv4.tcp_max_tw_buckets = 5000  
     #标识TCP三次握手建立阶段接受SYN请求队列的最大长度，默认是1024，高并发情况下可以设置大些，不至于丢失客户端发起的连接请求
     net.ipv4.tcp_max_syn_backlog = 262144  
     #本地对外连接端口范围
     net.ipv4.ip_local_port_range = 1024 61000
     #TCP读buffer,TCP接收滑动窗口的最小值，默认值，最大值
     net.ipv4.tcp_rmem = 4096 32768 262142
     #TCP写buffer，TCP发送滑动窗口的最小值，默认值，最大值
     net.ipv4.tcp_wmem = 4096 32768 262142 

     #禁用包过滤功能 
     net.ipv4.ip_forward = 0  
     #启用源路由核查功能 
     net.ipv4.conf.default.rp_filter = 1  
     #禁用所有IP源路由 
     net.ipv4.conf.default.accept_source_route = 0  
     #使用sysrq组合键是了解系统目前运行情况，为安全起见设为0关闭
     kernel.sysrq = 0  
     #控制core文件的文件名是否添加pid作为扩展
     kernel.core_uses_pid = 1  
     #开启SYN Cookies，当出现SYN等待队列溢出时，启用cookies来处理
     net.ipv4.tcp_syncookies = 1  
     #每个消息队列的大小（单位：字节）限制
     kernel.msgmnb = 65536  
     #整个系统最大消息队列数量限制
     kernel.msgmax = 65536  
     #单个共享内存段的大小（单位：字节）限制，计算公式64G*1024*1024*1024(字节)
     kernel.shmmax = 68719476736  
     #所有内存大小（单位：页，1页 = 4Kb），计算公式16G*1024*1024*1024/4KB(页)
     kernel.shmall = 4294967296  
     #开启有选择的应答
     net.ipv4.tcp_sack = 1  
     #支持更大的TCP窗口. 如果TCP窗口最大超过65535(64K), 必须设置该数值为1
     net.ipv4.tcp_window_scaling = 1  
     #为TCP socket预留用于发送缓冲的内存默认值（单位：字节）
     net.core.wmem_default = 8388608
     #为TCP socket预留用于发送缓冲的内存最大值（单位：字节）
     net.core.wmem_max = 16777216  
     #为TCP socket预留用于接收缓冲的内存默认值（单位：字节）  
     net.core.rmem_default = 8388608
     #为TCP socket预留用于接收缓冲的内存最大值（单位：字节）
     net.core.rmem_max = 16777216
     #每个网络接口接收数据包的速率比内核处理这些包的速率快时，允许送到队列的数据包的最大数目
     net.core.netdev_max_backlog = 262144  
     #web应用中listen函数的backlog默认会给我们内核参数的net.core.somaxconn限制到128，而nginx定义的NGX_LISTEN_BACKLOG默认为511，所以有必要调整这个值
     net.core.somaxconn = 262144  
     #系统中最多有多少个TCP套接字不被关联到任何一个用户文件句柄上。这个限制仅仅是为了防止简单的DoS攻击，不能过分依靠它或者人为地减小这个值，更应该增加这个值(如果增加了内存之后)
     net.ipv4.tcp_max_orphans = 3276800  
     #时间戳可以避免序列号的卷绕。一个1Gbps的链路肯定会遇到以前用过的序列号。时间戳能够让内核接受这种“异常”的数据包。这里需要将其关掉
     net.ipv4.tcp_timestamps = 0  
     #为了打开对端的连接，内核需要发送一个SYN并附带一个回应前面一个SYN的ACK。也就是所谓三次握手中的第二次握手。这个设置决定了内核放弃连接之前发送SYN+ACK包的数量
     net.ipv4.tcp_synack_retries = 1  
     #在内核放弃建立连接之前发送SYN包的数量
     net.ipv4.tcp_syn_retries = 1  
     #开启TCP连接中time_wait sockets的快速回收
     net.ipv4.tcp_tw_recycle = 1  
     #1st低于此值,TCP没有内存压力,2nd进入内存压力阶段,3rdTCP拒绝分配socket(单位：内存页)
     net.ipv4.tcp_mem = 94500000 915000000 927000000     
     ```
     * ```shell
       #重新导入配置文件参数
       sysctl -p  
       ```

3. 安装

   ```shell
   wget https://nginx.org/download/nginx-1.12.2.tar.gz
   tar -zxvf nginx-1.12.2.tar.gz
   cd nginx-1.12.2
   ./configure
   make
   make install
   ```

4. 常用命令

   ```shell
   nginx - s stop	#fast shutdown , 立即退出进程
   nginx - s quit  #graceful shutdown   ， 关闭监听端口，停止接收新的连接，把当前正在处理的连接处理完，然后退出
   nginx - s reload #reloading the configuration
   nginx - s reopen #reopening the log files ， 可以把当前日志文件改名或者移动到其他目录进行备份， reopen 重新打开时会生成新的日志文件。
   ps -ef | grep nginx

   ```

   ​

## 初探Nginx

* nginx在启动后，会有一个master进程和多个worker进程。master进程主要用来管理worker进程，包含：接收来自外界的信号，向各worker进程发送信号，监控worker进程的运行状态，当worker进程退出后(异常情况下)，会自动重新启动新的worker进程。
* 多个worker进程之间是对等的，他们同等竞争来自客户端的请求，各进程互相之间是独立的。一个请求，只可能在一个worker进程中处理。
* worker进程的个数是可以设置的，一般会设置与机器cpu核数一致。可以把每个worker进程绑定一个cpu。
* 所有worker进程的listenfd会在新连接到来时变得可读，为保证只有一个进程处理该连接，所有worker进程在注册listenfd读事件前抢accept_mutex，抢到互斥锁的那个进程注册listenfd读事件，在读事件里调用accept接受该连接。
* nginx采用了异步非阻塞的方式来处理请求。
* nginx在做4个字节的字符串比较时，会将4个字符转换成一个int型，再作比较，以减少cpu的指令数。

**基础概念**

* connection : 是对TCP的封装，包括Socket , 读事件，写事件。
  * nginx 中每个进程都有一个连接数的最大上限制，这个上限与操作系统对fd的限制不一样。操作系统可以用`ulimit -n` 查看一个进程可以打开的最大fd的数量。每个socket连接都会占用一个fd。nginx通过设置`worker_connections`来设置每个进程支持的最大连接数。最终能打开的连接数取系统和nginx设置的最小值。
  * nginx 通过一个 free_connections 的链表保存所有的空闲连接，每次获取新的连接时，从空闲链表中获取，用完后放回空闲链表。
  * 一个nginx能建立的最大连接数，应该是worker_connections * worker_processes。而如果是HTTP作为反向代理来说，最大并发数量应该是worker_connections * worker_processes/2。因为作为反向代理服务器，每个并发会建立与客户端的连接和与后端服务的连接，会占用两个连接。
  * nginx 会根据每个worker进程空闲链表的大小来维持负载均衡，空闲链表越多，获取新的请求处理的机会就越大。
* request  ： 指 http请求，包括请求行，请求头，请求体，响应行，响应头，响应体。
* Keepalive :  长连接
* pipe  ： 异步的发起多个请求
* Lingering_close  :  延迟关闭。先关闭写，延时一段时间再关闭连接的读。

## Nginx 配置系统

* 由一个主配置文件和其他一些辅助的配置文件构成。全部位于nginx安装目录下的conf目录下。

**指令概述**

* 配置指令是一个字符串，可以用单引号或者双引号括起来，也可以不括。有空格一定要用引号。

**指令参数**

* 多个参数用空格或TAB分隔。指令参数有一个或多个TOKEN 串组成。

* TOKEN串分为简单字符串或者复合配置块。复合配置块是由大括号括起来的一堆内容，其中可能包含若干其他配置指令。

* 对于简单配置项，以分号结尾，比如：

  ```
  error_page   500 502 503 504  /50x.html;
  ```

* 复合配置块示例：

  ```
  location / {
      root   /home/jizhao/nginx-book/build/html;
      index  index.html index.htm;
  }
  ```

**指令上下文**

* 配置文件根据逻辑意义进行分类，也就是分成了多个作用域，即为配置指令的上下文。常见的指令上下文如下：
  * main  : nginx 运行时与具体业务功能（比如http服务器或者email服务器）无关的参数，比如工作进程数，运行身份等等。
  * http ： http服务相关的配置，如是否使用 `keepalive` ， 是否使用gzip进行压缩。
  * server   :   http服务支持若干虚拟机，每个虚拟主机对应的 server 配置项，包括该虚拟主机的相关配置。
  * location  ： http 服务中，某些特定的URL对应的一系列配置项。
  * mail  ：  实现email 相关的SMTP/IMAP/POP3代理时，共享的一些配置。
* 指令上下文可能有包含的情况出现。例如：http 和 mail 一定出现在main上下文里。在一个上下文里，可能包含另一个类型的上下文多次。如：http 服务， 支持多个虚拟主机，则会包含多个server 上下文。

```
user  nobody;
worker_processes  1;
error_log  logs/error.log  info;

events {
    worker_connections  1024;
}

http {
    server {
        listen          80;
        server_name     www.linuxidc.com;
        access_log      logs/linuxidc.access.log main;
        location / {
            index index.html;
            root  /var/www/linuxidc.com/htdocs;
        }
    }

    server {
        listen          80;
        server_name     www.Androidj.com;
        access_log      logs/androidj.access.log main;
        location / {
            index index.html;
            root  /var/www/androidj.com/htdocs;
        }
    }
}

mail {
    auth_http  127.0.0.1:80/auth.php;
    pop3_capabilities  "TOP"  "USER";
    imap_capabilities  "IMAP4rev1"  "UIDPLUS";

    server {
        listen     110;
        protocol   pop3;
        proxy      on;
    }
    server {
        listen      25;
        protocol    smtp;
        proxy       on;
        smtp_auth   login plain;
        xclient     off;
    }
}
```

上面的示例配置中：

* 存在于main 上下文的配置指令有：
  * user
  * worker_processes
  * error_log
  * events
  * http
    * server
      * listen
      * server_name
      * access_log
      * location
        * index
        * root
  * mail
    * server
      * protocol
      * proxy
      * smtp_auth
      * xclient
    * auth_http
    * imap_capabilities

## nginx 模块化体系系统

* nginx 的内部结构是由Nginx Core 核心部分和一系列功能模块组成。
* Nginx提供了web服务器的基础功能，web服务反向代理，email服务反向代理功能。
* nginx core实现了底层的通讯协议，为其他模块和nginx进程构建了基本的运行时环境，并且构建了其他各模块的协作基础。

**模块概述**

* nginx 将各功能模块组织成一条链，当有请求到达时，请求依次经过这条链上的部分或全部模块，进行处理，每个模块实现特定的功能。
* 两个模块比较特殊：http 模块  和  mail 模块 。它们处于nginx core 核心和各功能模块的中间。它们在nginx core 之上实现了另外一层抽象，处理与HTTP协议和email 协议相关的协议有关的事件。并且确保这些事件能被以正确的顺序调用其他一些功能模块。

**模块的分类**

* event  module   ：  搭建了独立于操作系统的时间处理机制框架，及提供了各种事件的处理。如 ： ngx_events_module ,  ngx_event_core_module 和 ngx_epoll_module 。
* phase handler   :    这种类型的模块可直接称为 handler  模块。主要负责客户端请求并产生待响应的内容。如 ngx_http_static_module 。
* output filter      :   也称filter  模块，主要负责对输出内容进行处理，可以对输出进行修改。如实现对所有的html页面增加定义的footbar ， 或者对输出的图片url  进行替换之类的工作。
* upstream    :  实现反向代理的功能，将真正的请求转向后端服务器。并从后端服务器上读取响应，发回客户端。 upstream 是一种特殊的 handler ，只是响应内容不是真的由自己产生，而是从后端服务器上读取。
* Load-balance   :  负载均衡模块。

##nginx 的请求处理

* nginx 使用**多进程**模型对外提供服务，其中一个master 进程，多个worker进程。master进程负载管理nginx本身和其他worker进程。
* 一个简单的请求处理流程如下：
  1. 操作系统提供的机制（如epoll ）产生相关的事件。
  2. 接收和处理这些事件。如是接收到数据，则产生更高层的request 对象。
  3. 处理request 的header 和 body。
  4. 产生响应，发送回客户端。
  5. 完成request 的处理。
  6. 重新初始化定时器及其他的组件。

**请求的处理流程**

以一个HTTP Request 为例，从nginx  内部来看，一个HTTP Request 的处理过程涉及以下几个阶段。

* 初始化HTTP Request ，即生成HTTP Request  请求对象。
* 处理请求头
* 处理请求体
* 如果有的话，调用与此请求（URL或Location） 关联的handler 。
* 依次调用各 Phase handler 进行处理。

Phase handler 中的phase , 阶段。即包含若干处理阶段的一些handler 。每一个阶段可能有若干个handler 。

一个phase handler 通常执行以下几项任务：

* 获取location  配置
* 产生适当的响应
* 发送 response header 
* 发送 response body 

当nginx 读到一个HTTP Request 的header 的时候，nginx 首先查找与这个请求关联的虚拟主机的配置。如果找到了虚拟主机的配置，那么通常情况下，这个HTTP Request 将会经过以下几个阶段的处理：

* 读取请求内容


* Server请求地址重写


* 配置查找
* Location 请求地址重写
* 请求地址重写
* 访问权限检查
* 访问权限提交
* 配置项 try_files 处理
* 内容产生阶段   ：  交给一个合适的content handler 去处理，如果这个Request 对应的location 在配置文件里明确指定了一个content handler，那么通过对location匹配，直接找到并把请求交给这个handler 。 如果没有直接配置，那么就依次尝试：
  * 如果location 里面有配置 random_index_on ， 那么随机选择一个文件发送给客户端。
  * 如果location 里面有配置 index 指令，那么发送index指令指明的文件。
  * 如果location 里面有配置 autoindex on ，那么就发送请求地址对应的服务器端路径下的文件列表给客户端。
  * 如果request 对应的location上有设置gzip_static on ，那么查找是否有对应的.gz 文件存在，有就发送这个给客户端。
  * 请求的URI如果对应一个静态文件，static module 就发送静态文件内容给客户端。
* 日志模块处理阶段

内容阶段生产完后，生成的输出会被传递到filter 模块中进行处理。filter 模块也与location有关。所有的filter都被组织成一条链，输出依次穿越所有的filter ，直到有一个filter 模块的返回值表明处理已完成。常见的filter 有

* server-side includes 
* XSLT filter 
* 图像缩放
* gzip压缩

所有filter中，有几个需要关注一下：按调用顺序依次如下

* write ： 写输出到客户端，实际上是写到对应的socket上
* postpone  :  filter 是负责subrequest 的 ，也就是子请求的。
* copy  ： 将一些需要复制的buf 文件或者内存重新复制一份交给剩余的body filter 。

# handler模块

## 简介

* handler 就是接受来自客户端的请求并产生输出的模块。
* location 指令可以指定content handler ；每个handler 都有一次机会关联到对应的location上，如果有多个handler都关联了同一个location ， 那么实际上只有一个handler 模块真正起作用。应避免这种情况发生。
* handler 模块的处理结果通常3种情况：处理成功，处理失败，拒绝处理。拒绝处理的情况下就会由默认的handler模块进行处理。如关联到某个location 的handler 模块拒绝请求，就会由默认的 ngx_http_static_module 处理。

## handler模块的基本结构

### 配置结构



