# vultr VPS安装shadowsocks和BBR

[参考](https://medium.com/@zoomyale/%E7%A7%91%E5%AD%A6%E4%B8%8A%E7%BD%91%E7%9A%84%E7%BB%88%E6%9E%81%E5%A7%BF%E5%8A%BF-%E5%9C%A8-vultr-vps-%E4%B8%8A%E6%90%AD%E5%BB%BA-shadowsocks-fd57c807d97e)

[参考](http://boblogs.com/index.php/2017/09/14/vultr_ss/)

1. 安装shadowsocks

   ```Shell
   wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh
   chmod +x shadowsocks-all.sh
   ./shadowsocks-all.sh 2>&1 | tee shadowsocks-all.log
   ```

   参考：[Shadowsocks 一键安装脚本（四合一）](https://shadowsocks.be/11.html)

2. 安装BBR

   ```Shell
   wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh
   chmod +x bbr.sh
   ./bbr.sh
   ```

   更多参考：[一键安装最新内核并开启 BBR 脚本](https://teddysun.com/489.html)

   ​

