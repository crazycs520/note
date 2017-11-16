# 进程

## 将进程放到后台运行

1. ctrl +z //暂停当前正在运行的进程。
   再执行：bg

2. &




# 运维常用命令

* `ifconfig`
  * `ethtoo` 查看网卡的速度和工作模式
  * `ethtool eth0 | egrep -i 'Speed|Duplex' `
* `w`   查看服务器的运行时间，当前用户，运行程序，1分钟，5分钟，10分钟的平均负载
* `df`
  * `df -h` 查看挂载盘的目录，  总容量和使用量，Inode 的总量和使用量，磁盘文件系统类型。
* ps  , 查看进程ID，cpu，mem使用量，可占用的内存空间大小，实际占用内存空间大小等
  * `ps -ef | grep mysqld` 
* `free -h` 查看内存使用情况
  * sysctl 中有一个参数，`vm.swappiness`，默认是60，在内存足够的时候可以设置为0 ， 尽量避免使用swap 
* `vmstat` 查看 swap的I/O情况
* `netstat` 和 `ss`  : 查看网络数据的相关命令
  * `netstat -plnt` 查看本机监听TCP端口号的进程列表输出
  * `netstat -ant` 查看本机所有TCP连接，地址，状态等
  * `netstat -st` 查看服务器的全部连接数
* `iostat` 查看IO相关状态，读写请求数等
* `sar` 查看系统活动等信息
  * `sar -n DEV`查看网络统计信息
  * `sar -r` 内存统计信息
* `mtr` 集成了 `ping ` 和 `traceroute`的工具，默认每秒发送一次并刷新数据界面







# 问题

发现系统存在大量TIME_WAIT状态的连接，通过调整内核参数解决，

vi /etc/sysctl.conf

编辑文件，加入以下内容：
`net.ipv4.tcp_syncookies = 1net.ipv4.tcp_tw_reuse = 1net.ipv4.tcp_tw_recycle = 1net.ipv4.tcp_fin_timeout = 30`

`` 

``然后执行 `/sbin/sysctl -p` 让参数生效。

 

**net.ipv4.tcp_syncookies = 1** 表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭；

**net.ipv4.tcp_tw_reuse = 1** 表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；

**net.ipv4.tcp_tw_recycle = 1** 表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭。

**net.ipv4.tcp_fin_timeout** 修改系統默认的 TIMEOUT 时间



发现大量的TIME_WAIT 已不存在，mysql进程的占用率很快就降下来的，各网站访问正常！！

​       以上只是暂时的解决方法，如果不是被攻击了，应该检查新更改的代码，查看程序代码中是不是忘记关闭连接了mysql.colse()，才导致大量的mysql  TIME_WAIT  