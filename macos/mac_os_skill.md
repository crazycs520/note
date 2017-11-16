# software

## iterm2+oh my zsh+tmux

https://github.com/zsh-users/zsh-autosuggestions

## 终端走代理翻墙

http://www.iosugar.com/2017/02/19/Mac-terminal-environment/

https://www.goodspb.net/mac-%E4%BD%BF%E7%94%A8-golang-%E7%BF%BB%E5%A2%99%E5%AE%9E%E5%BD%95/

```shell
#安装privoxy
brew install privoxy
#打开privoxy配置文件/usr/local/etc/privoxy/config,在最下面添加：
#1080是Shadowsocks代理的端口，这个改成你的端口。8118是开启http代理的端口。使用0.0.0.0即可在局域网内使用此代理，如只想本机使用，使用127.0.0.1。
listen-address 0.0.0.0:8118
forward-socks5 / 127.0.0.1:1080 .
#开启privoxy
/usr/local/sbin/privoxy /usr/local/etc/privoxy/config
#查看是否成功开启8118端口进行监听
netstat -na | grep 8118
#设置代理服务器,为了避免每次开机都能生效,可以将上面两句配置添加到.zshrc/.bashrc_profile中
export http_proxy='http://localhost:8118'
export https_proxy='http://localhost:8118'
```



## iStats

https://github.com/Chris911/iStats

终端查看CPU，温度

## Today Scripts

https://sspai.com/post/40169

通知中心运行脚本



## sublime

* 命令行打开sublime 

  1. 设置alias

  在你的 `.bashrc` 或者 `.zshrc` 中添加下面的设置:

  ```
  alias subl=\''/Applications/Sublime Text.app/Contents/SharedSupport/bin/subl'\

  ```

  2. 设置软链接

  ```
  sudo ln -s "/Applications/Sublime Text.app/Contents/SharedSupport/bin/subl" /usr/local/bin/subl
  ```

  ​

# command

## diskutil

```
diskutil list
```

```shell
diskutil eraseDisk JHFS+ iDisk disk2
```

JHFS+为格式名称，代表Mac OS X扩展(日志式)
iDisk 为格式化U盘后分区的名称，可以修改
disk2 为U盘名称，这里请替换直接的U盘代号

# upgrade ssd

1. Use **Disk Utility** Erase your new ssd , Mac OS Extended(Journaled) , GUID Partition Map

2. Download** [SuperDuper](http://www.shirt-pocket.com/SuperDuper/SuperDuperDescription.html)**to clone your old to new ssd, **Using **Backup -all files**

3. Untii now , may be ok in macbook , so just replace old ssd to new ssd.

   **but this time in hackintosh** ,  It can not start up with new ssd , so i **copy old EFI to new SSD EFI**, it work now:

   ```shell
   diskutil list #find your old ssd and new ssd
   diskutil mount /dev/disk0s1
   diskutil mount /dev/disk1s1
   # then open the finder , you can copy the old ssd EFI to new SSD EFI.
   ```




