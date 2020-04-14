# 如何安装dpdk(NetONE)
1. 通过ssh或者console进入系统shell
2. 首先, 请确定您使用的是飞腾NetONE系统, 检查的方法是
```bash
uname -a | grep ex01a
# 如果有如下的输出, 即表示是NetONE系统
# Linux netone 4.4-bex01a #1 SMP Mon Aug 15 00:55:38 UTC 2016 aarch64 GNU/Linux
```
3. 然后, 请参照[如何升级基于ipk的飞腾系统(NetONE)](arm-ft-how-to-upgrade-with-ipk)完成当前系统的升级, 确保当前系统包含所需软件包;
4. 接下来安装dpdk, 请执行:
```bash
ipkg-cl -f /root/ipkg.conf install dpdk  # 仅仅安装内核模块, 是dpdk的最小安装
ipkg-cl -f /root/ipkg.conf install dpdk-tools  # 安装dpdk-devbind, testpmd等dpdk基本配置和测试工具
 ipkg-cl -f /root/ipkg.conf install dpdk-tests  # 在/usr/local/bin/下安装dpdk examples下的部分测试程序
```
5. 重启
6. /usr/share/dpdk下, 可以查看setup.sh等dpdk官方工具。

## 以下内容仅供参考:
1. /etc/init.d/S30dpdk启动脚本
```bash
#!/bin/sh

echo "starting dpdk...."
/bin/mkdir -p /mnt/huge
/bin/mount -t hugetlbfs nodev /mnt/huge
echo 2048 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
```
2. /root/dpdk.sh
```bash
#!/bin/sh

# 如果使用4.4的内核, 请uncomment下一行, 使用igb_uio
# DRIVER=igb_uio
# 如果使用4.7的内核, 请uncomment下一行, 使用vfio-pci
# DRIVER=vfio-pci

# SLOTS的内容可以通过"dpdk-devbind --status"获取, 在本示例里面(bex01a), 09:00.0对应的是eth4, 09:00.1对应的是eth5
SLOTS="09:00.0 09:00.1"

/sbin/modprobe ${DRIVER}
for t in ${SLOTS}; do
      dpdk-devbind --force --bind=${DRIVER} ${t}
done
dpdk-devbind --status
```
3. 进入飞腾系统, 运行testpmd, 进行测试