# Linux 默认串口数修改方法
[TOC]

Linux系统下默认只支持4个串口，安装多串口卡后，超过四个的将无法识别，可通过如下修改来让Linux支持多串口。

编辑 ```/boot/grub/grub.conf``` 文件，在 kernel 开头的那一行末尾加 ``` 8250.nr_uarts=16 ```，此处16表示最大支持16个，可根据实际串口数量来修改。
```bash
title CentOS 6 (2.6.32-696.el6.x86_64)
        root (hd0,0)
        kernel /vmlinuz-2.6.32-696.el6.x86_64 ro root=/dev/mapper/VolGroup-lv_root rd_NO_LUKS LANG=en_US.UTF-8 rd_NO_MD rd_LVM_LV=VolGroup/lv_swap SYSFONT=latarr cyrheb-sun16 crashkernel=auto rd_LVM_LV=VolGroup/lv_root  KEYBOARDTYPE=pc KEYTABBLE=us rd_NO_DM rhgb console=tty0 console=ttyS0,115200n8 8250.nr_uarts=16 quiet
        initrd /initramfs-2.6.32-696.el6.x86_64.img
```
