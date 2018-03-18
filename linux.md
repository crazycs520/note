# 进程

## 将进程放到后台运行

1. ctrl +z //暂停当前正在运行的进程。
   再执行：bg
2. &




## 软件源

https://mirrors.aliyun.com/help/centos

```Shell
#备份
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup

#下载阿里云的源
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```





### wget  断点续传，限速

```shell
wget -c --limit-rate=300k http://mirrors.163.com/ubuntu-releases/9.10/ubuntu-9.10-desktop-amd64.iso 
```





#### 查看系统配置

https://linux.cn/article-861-1.html

```
rpm -q centos-release		#查看centos版本
# uname -a # 查看内核/操作系统/CPU信息
# head -n 1 /etc/issue # 查看操作系统版本
# cat /proc/cpuinfo # 查看CPU信息
# hostname # 查看计算机名
# lspci -tv # 列出所有PCI设备
# lsusb -tv # 列出所有USB设备
# lsmod # 列出加载的内核模块
# env # 查看环境变量

# free -m # 查看内存使用量和交换区使用量
# df -h # 查看各分区使用情况
# du -sh <目录名> # 查看指定目录的大小
# grep MemTotal /proc/meminfo # 查看内存总量
# grep MemFree /proc/meminfo # 查看空闲内存量
# uptime # 查看系统运行时间、用户数、负载
# cat /proc/loadavg # 查看系统负载

# mount | column -t # 查看挂接的分区状态
# fdisk -l # 查看所有分区
# swapon -s # 查看所有交换分区
# hdparm -i /dev/hda # 查看磁盘参数（仅适用于IDE设备）
# dmesg | grep IDE # 查看启动时IDE设备检测状况

# ifconfig # 查看所有网络接口的属性
# iptables -L # 查看防火墙设置
# route -n # 查看路由表
# netstat -lntp # 查看所有监听端口
# netstat -antp # 查看所有已经建立的连接
# netstat -s # 查看网络统计信息

# w # 查看活动用户
# id <用户名> # 查看指定用户信息
# last # 查看用户登录日志
# cut -d: -f1 /etc/passwd # 查看系统所有用户
# cut -d: -f1 /etc/group # 查看系统所有组
# crontab -l # 查看当前用户的计划任务

# rpm -qa # 查看所有安装的软件包
```





## vmware 设置固定IP

http://www.up4dev.com/2016/10/15/vmware-fusion-static-ip/

https://willwarren.com/2015/04/02/set-static-ip-address-in-vmware-fusion-7/

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

* 解压.tar.xz 

  * xz -d ***.**tar.xz**

    tar -xvf  ***.tar

* curl

  * ```sh
    curl -o /dev/null -s -w %{http_code} www.baidu.com    	#测试网站是否正常
    ```




## shell

### shell设置工作目录

```shell
work_path=$(dirname $(readlink -f $0))                                                                                                                                                                                                            echo $work_path                                                                                                                                                                                                                                   
cd ${work_path} 
```









##c100K长连接

*  [单机服务器支持千万级并发长连接的压力测试](http://blog.csdn.net/lijinqi1987/article/details/74545851)
*  http://colobu.com/2015/05/22/implement-C1000K-servers-by-spray-netty-undertow-and-node-js/


# 问题

1. 发现系统存在大量TIME_WAIT状态的连接，通过调整内核参数解决，

vi /etc/sysctl.conf

编辑文件，加入以下内容：
`net.ipv4.tcp_syncookies = 1net.ipv4.tcp_tw_reuse = 1net.ipv4.tcp_tw_recycle = 1net.ipv4.tcp_fin_timeout = 30`

``然后执行 `/sbin/sysctl -p` 让参数生效。

2. SYN flooding处理--内核调优

   http://blog.51cto.com/xujpxm/1846095

   ```shell
   # vim /etc/sysctl.conf 
   net.ipv4.tcp_syncookies = 1
   net.ipv4.tcp_tw_reuse = 1
   net.ipv4.tcp_tw_recycle = 1
   net.ipv4.tcp_fin_timeout = 300
   net.ipv4.tcp_max_syn_backlog = 2048
   保存退出后，执行：
   # sysctl -p
   ```

   http://blog.csdn.net/gaojinshan/article/details/40895767

   ​



**net.ipv4.tcp_syncookies = 1** 表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭；

**net.ipv4.tcp_tw_reuse = 1** 表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；

**net.ipv4.tcp_tw_recycle = 1** 表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭。

**net.ipv4.tcp_fin_timeout** 修改系統默认的 TIMEOUT 时间



发现大量的TIME_WAIT 已不存在，mysql进程的占用率很快就降下来的，各网站访问正常！！

​       以上只是暂时的解决方法，如果不是被攻击了，应该检查新更改的代码，查看程序代码中是不是忘记关闭连接了mysql.colse()，才导致大量的mysql  TIME_WAIT  



## Flame Graphs 火焰图

http://www.brendangregg.com/flamegraphs.html

https://github.com/brendangregg/FlameGraph

### Supervisor安装使用

https://www.jianshu.com/p/ff915e062f86





## perf

```
yum update && yum install perf 
```



### 更新时间

```
ntpdate time.windows.com && hwclock -w  
```





### **Error: Cannot retrieve repository metadata (repomd.xml) for repository: **

http://www.cnblogs.com/kevingrace/p/6252659.html

```
解决办法：（或者把/etc/yum.repos.d下的文件全部删除，然后将能正常使用yum的同类服务器的这个目录下的文件全部拷贝过来，然后yum clean all 和yum makecache 即可）
下载新的CentOS-Base.repo 到/etc/yum.repos.d/
[root@bastion-IDC src]# cd /etc/yum.repos.d/
其实就是将yum源更改为阿里云的yum源,操作如下：

1）centos5.*的下载连接：
[root@bastion-IDC yum.repos.d]# wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-5.repo

2）centos6.*的下载连接：
[root@bastion-IDC yum.repos.d]# wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo

3）centos7.*的下载连接：
[root@bastion-IDC yum.repos.d]# wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```



