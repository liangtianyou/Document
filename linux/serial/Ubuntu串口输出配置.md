# Ubuntu 串口输出配置
[TOC]
## 1 修改grub
增加：
```bash
GRUB_CMDLINE_LINUX="console=tty0 console=ttyS0,115200n8"
GRUB_TERMINAL=serial
GRUB_SERIAL_COMMAND="serial --speed=115200 --unit=0 --word=8 --parity=no --stop=1"
```
如下：
```bash
root@ubuntu:~# vi /etc/default/grub 
GRUB_DEFAULT=0
GRUB_HIDDEN_TIMEOUT=10
GRUB_HIDDEN_TIMEOUT_QUIET=true
GRUB_TIMEOUT=10
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
GRUB_CMDLINE_LINUX="console=tty0 console=ttyS0,115200"
GRUB_TERMINAL=serial
GRUB_SERIAL_COMMAND="serial --speed=115200 --unit=0 --word=8 --parity=no --stop=1"
```

## 2 更新grub
执行```update-grub```
```bash
root@ubuntu:~# update-grub
Generating grub configuration file ...
Warning: Setting GRUB_TIMEOUT to a non-zero value when GRUB_HIDDEN_TIMEOUT is set is no longer supported.
Found linux image: /boot/vmlinuz-3.13.0-161-generic
Found initrd image: /boot/initrd.img-3.13.0-161-generic
Found memtest86+ image: /boot/memtest86+.elf
Found memtest86+ image: /boot/memtest86+.bin
done
```

## 3 修改ttyS0配置文件
```/etc/init/ttyS0.conf```
如下：
```bash
root@ubuntu:~# vi /etc/init/ttyS0.conf 
start on stopped rc RUNLEVEL=[2345] and (
            not-container or
            container CONTAINER=lxc or
            container CONTAINER=lxc-libvirt)
 
stop on runlevel [!2345]
 
respawn
exec /sbin/getty -h -L -w  115200 ttyS0 vt100
```
如果不需要从VGA显示，在grub里删除console=tty0即可。

[原文链接](https://blog.csdn.net/weixin_42414349/article/details/83510909)
