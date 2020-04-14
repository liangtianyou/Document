# 如何启动一个最简的飞腾系统

1. 首先, 确保您已经把飞腾系统更新到最新的版本
2. 通过console或者ssh进入shell, 执行
```bash
touch /etc/.baresys
```
3. 注意, 此时如果您重启系统, 将得到一个没有网络配置, 没有ssh配置的linux环境
4. 为了在重启后依然可以有网络, 可以有ssh, 请创建/etc/init.d/S50baresetup, 该文件包含下面的内容:
```bash
#!/bin/sh

IPADDRETH0=192.168.10.60 # 替换成你期望的网络地址
GATEWAY=192.168.10.1 #替换成你期望的缺省网关地址
SSHKEY=/usr/local/conf/dropbear.pem
NAMESERVER=114.114.114.114

/sbin/ip link set lo up
/sbin/ifconfig eth0 ${IPADDRETH0}
/sbin/ifconfig eth0 up
/sbin/ip route add default via ${GATEWAY}
if [ ! -f ${SSHKEY} ]; then
mkdir -p /usr/local/conf
/usr/bin/dropbearkey -t rsa -f ${SSHKEY} -s 1024
fi
/usr/sbin/dropbear -r ${SSHKEY} -p 22
# 设置dns服务器
echo "nameserver ${NAMESERVER}" > /etc/resolv.conf
```
5. 执行
```bash
chmod +x /etc/init.d/S50baresetup
# 设置root用户密码
passwd
# 如果dropbear尚未安装, 请执行
ipkg-cl -f /root/ipkg.conf install dropbear
```
6. 重启系统
7. 如果打算回到原先的系统启动结果, 执行
```bash
rm /etc/.baresys
mv /etc/init.d/S50baresetup /root
```