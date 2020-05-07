# Centos 下串口输出配置
[TOC]
## 1 查看系统是否支持串口
```bash
[root@localhost ~]# dmesg |grep tty
console [tty0] enabled
serial8250: ttyS0 at I/O 0x3f8 (irq = 3) is a 16550A
serial8250: ttyS1 at I/O 0x2f8 (irq = 4) is a 16550A
00:09: ttyS0 at I/O 0x3f8 (irq = 3) is a 16550A
00:0a: ttyS1 at I/O 0x2f8 (irq = 4) is a 16550A
```

## 2 CentOS7 串口配置

### 2.1 修改/etc/defaule/grub，增加下面几行：
```bash
GRUB_TERMINAL="console serial"
GRUB_SERIAL_COMMAND="serial --speed=115200 --unit=0 --word=8 --parity=no --stop=1"
GRUB_CMDLINE_LINUX_DEFAULT="console=tty1 console=ttyS0,115200"
```
查看：
```bash
[root@localhost ~]# vi /etc/default/grub 
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="rd.lvm.lv=centos/root rd.lvm.lv=centos/swap biosdevname=0 net.ifnames=0 rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
GRUB_TERMINAL="console serial"
GRUB_SERIAL_COMMAND="serial --speed=115200 --unit=0 --word=8 --parity=no --stop=1"
GRUB_CMDLINE_LINUX_DEFAULT="console=tty1 console=ttyS0,115200"
```
### 2.2 更新grub。
执行```grub2-mkconfig -o /boot/grub2/grub.cfg```
输出如下：
```bash
[root@localhost ~]# grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-862.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-862.el7.x86_64.img
Found linux image: /boot/vmlinuz-0-rescue-bcfdd64db72c4cce8e4ea8a33b2f64a3
Found initrd image: /boot/initramfs-0-rescue-bcfdd64db72c4cce8e4ea8a33b2f64a3.img
done
```
## 3 CentOS6 串口配置
### 3.1 修改/etc/inittab
增加```s0:2345:respawn:/sbin/agetty -L -f /etc/issue.serial 115200 ttyS0 vt100```
如下：
```bash
[root@localhost ~]# vi /etc/inittab 
s0:2345:respawn:/sbin/agetty -L -f /etc/issue.serial 115200 ttyS0 vt100
```

### 3.2 编辑/etc/securetty文件
增加ttyS0，表示可以root登录终端。
如下：
```bash
[root@localhost ~]# vi /etc/securetty 
ttyS0
```

### 3.3 修改 /boot/grub/grub.conf文件
在kernel行最后加```console=tty0 console=ttyS0,115200n8```
如下：
```bash
[root@localhost ~]# vi /boot/grub/grub.conf
title CentOS 6 (2.6.32-696.el6.x86_64)
        root (hd0,0)
        kernel /vmlinuz-2.6.32-696.el6.x86_64 ro root=/dev/mapper/VolGroup-lv_root rd_NO_LUKS LANG=en_US.UTF-8 rd_NO_MD rd_LVM_LV=VolGroup/lv_swap SYSFONT=latarcyrheb-sun16 crashkernel=auto rd_LVM_LV=VolGroup/lv_root  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM rhgb console=tty0 console=ttyS0,115200n8 quiet
        initrd /initramfs-2.6.32-696.el6.x86_64.img
```       

## 4 重启验证

[原文链接](https://blog.csdn.net/weixin_42414349/article/details/83538159)
