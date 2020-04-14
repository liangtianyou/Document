# 如何制作飞腾NetONE启动盘

本文将指导用户自行制作飞腾NetONE启动盘, 在下面的例子里面, 我们采用CF卡作为飞腾系统盘, 如果您采用硬盘, 其实操作的差别也不大.

1. 准备一张CF卡, 容量不小于1G
2. 在linux下对CF卡进行分区, 建议把所有空间都分成一个区, 主分区(primary), 设置成LINUX类型--(83). (man fdisk)
3. 格式化这个分区到ext4 (man mkfs.ext4)
4. mount CF卡分区到/mnt
	```bash
    mount -t ext4 /dev/sdb1 /mnt)
  ```
5. 从这里下载[netone-baresys](../static/dl/netone-baresys_20160819_aarch64.tar.gz)
6. 将netone-baresys解压缩到CF卡 
	```bash
	tar zxf netone-baresys_20160819_aarch64.tar.gz -C /mnt
	```
7. 卸载CF卡
	```bash
    umount /mnt
  ```
8. CF卡制作完成.
**注意**: 这张CF卡上的系统是按照 [如何启动一个最简的飞腾系统(NetONE)](arm-ft-how-to-start-a-bare-system.md) 制作的, 从 [如何启动一个最简的飞腾系统(NetONE)](arm-ft-how-to-start-a-bare-system.md) 您可以了解到缺省的网络设置和服务.

