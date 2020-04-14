# 如何升级基于ipk的飞腾系统(NetONE)
[TOC]

目前提供的基于飞腾的Linux系统共有两种, 一种基于debian系统, 采用debian自带的apt/dpkg作为包管理系统, 另一种则是基于ipk的专用系统.

基于ipk的系统升级方式如下:

**注意**： 以下的例子都是基于BEX01A主板,每种主板有自己的发布目录, 升级前, 请自行替换下面例子中的发布目录到主板对应发布目录:
|主板型号|操作系统发布路径|备注|
|--|--|--|
|VX100|http://ipk.syan.com.cn/v3/aarch64-vx0100| -|
|BEX01A|http://ipk.gxwatek.com/v3/aarch64-bex01a| -|
|CEX01A |http://ipk.gxwatek.com/v3/aarch64-cex01a| -|

## 1 升级到最新系统
```bash
# 使用ssh或者串口登录飞腾系统
echo "src netone http://ipk.syan.com.cn/v3/aarch64-vx0100" > /root/ipkg.conf
ipkg-cl -f /root/ipkg.conf update
ipkg-cl -f /root/ipkg.conf upgrade
```
## 2 安装/升级某个ipk
ipk包的命名规则是 NAME_VERSION-RELEASE_ARCH.ipk, 从这里您可以浏览目前已经发布的安装包.

您也可以不升级整个系统, 而是升级/安装某个ipk包(依赖包将会自动安装):
```bash
# 使用ssh或者串口登录飞腾系统
echo "src netone http://ipk.syan.com.cn/v3/aarch64-vx0100" > /root/ipkg.conf
ipkg-cl -f /root/ipkg.conf update
ipkg-cl -f /root/ipkg.conf install NAME_OF_IPK
# 例如, 单独安装numactl可以使用下面的命令
# ipkg-cl -f /root/ipkg.conf install numactl
```
## 3 更换ipk仓库/降级
每次NetONE系统升级, 我们都会把原先的系统备份在这里. 如果您的系统需要降级, 请
1. 首先按照如何制作飞腾系统盘(NetONE)制作系统盘;
2. 用新的系统盘替换原有系统盘;
3. 重启
4. 更新到存档版本, 例如使用20160922的版本作为ipk来源
```bash
# 使用ssh或者串口登录飞腾系统
echo "src netone http://ipk.syan.com.cn/archive/20160922/aarch64-bex01a/" > /root/ipkg.conf
ipkg-cl -f /root/ipkg.conf update
ipkg-cl -f /root/ipkg.conf upgrade
```
## 4 专家模式
如果觉得重做系统盘比较麻烦, 还有一种专家模式, 不需要重做系统盘, 但是我们只建议在我方人员的指导下进行:
```bash
# 使用ssh或者串口登录飞腾系统
sed -i -e 's/4.7.5/1.0.0/g' /usr/lib/ipkg/status   #将当前系统记录的内核版本降到1.0.0, 欺骗包管理系统, 使得可以通过升级方式完成降级
echo "src netone http://ipk.syan.com.cn/v3/aarch64-vx0100" > /root/ipkg.conf
ipkg-cl -f /root/ipkg.conf update
ipkg-cl -f /root/ipkg.conf upgrade
```