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

## 初探Nginx

* nginx在启动后，会有一个master进程和多个worker进程。master进程主要用来管理worker进程，包含：接收来自外界的信号，向各worker进程发送信号，监控worker进程的运行状态，当worker进程退出后(异常情况下)，会自动重新启动新的worker进程。
* 多个worker进程之间是对等的，他们同等竞争来自客户端的请求，各进程互相之间是独立的。一个请求，只可能在一个worker进程中处理。
* worker进程的个数是可以设置的，一般会设置与机器cpu核数一致。
* 所有worker进程的listenfd会在新连接到来时变得可读，为保证只有一个进程处理该连接，所有worker进程在注册listenfd读事件前抢accept_mutex，抢到互斥锁的那个进程注册listenfd读事件，在读事件里调用accept接受该连接。
* nginx采用了异步非阻塞的方式来处理请求。
* nginx在做4个字节的字符串比较时，会将4个字符转换成一个int型，再作比较，以减少cpu的指令数。