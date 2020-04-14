# 如何从CF卡/硬盘启动飞腾系统

[TOC]

## 1 硬盘启动说明
1. 飞腾主板断电的情况下取出CF卡；
2. 接上硬盘数据线、电源线；
3. 飞腾主板连上串口, 接通电源，在串口console有输出的情况下, 敲打任意键，等待出现字符“FT1500a#”时，键入如下命令：
```bash
setenv bootargs console=ttyS1,115200 earlyprintk=uart8250-32bit,0x28001000 root=/dev/sda1 initrd=0x95000000,16M rootwait rw KEYBOARDTYPE=pc KEYTABLE=us;

setenv scsi_load_kern ext4load scsi 0:1 0x90100000 boot/uImage;

setenv scsi_load_initrd ext4load scsi 0:1 0x95000000 boot/initrd.m.gz;

setenv scsi_load_fdt ext4load scsi 0:1 0x90000000 boot/dtb;

setenv scsi_boot run scsi_load_kern\;run scsi_load_fdt\;run scsi_load_initrd\;eq close c0\;eq close c1\;eq close c4\;eq close c5\;pci enum\;bootm 0x90100000    0x95000000:0x1000000 0x90000000;

setenv bootcmd run scsi_boot;

saveenv

run scsi_boot
```
**注**：每条命令逐个执行。

执行完最后一步操作，硬盘启动系统，登录用户名：root 密码为空。

## 2 CF卡启动说明
1. 飞腾主板断电拔除硬盘数据线、电源，接回CF卡；
2. 飞腾主板连上串口, 接通电源，在串口console有输出的情况下, 敲打任意键，等待出现字符“FT1500a#”时，键入如下命令：
```bash
setenv bootargs console=ttyS1,115200 earlyprintk=uart8250-32bit,0x28001000 root=/dev/sda1 initrd=0x95000000,16M rootwait rw KEYBOARDTYPE=pc KEYTABLE=us;

setenv cf_load_kern ext4load usb 0:1 0x90100000 boot/uImage;

setenv cf_load_initrd ext4load usb 0:1 0x95000000 boot/initrd.m.gz;

setenv cf_load_fdt ext4load usb 0:1 0x90000000 boot/dtb;

setenv cf_boot run cf_load_kern\;run cf_load_fdt\;run cf_load_initrd\;eq close c0\;eq close c1\;eq close c4\;eq close c5\;pci enum\;bootm 0x90100000    0x95000000:0x1000000 0x90000000;

setenv bootcmd run cf_boot;

saveenv

run cf_boot
```
**注**：每条命令逐个执行。

执行完最后一步操作，通过CF卡启动系统；


## 3 多块硬盘(CF卡)并存的情况下的启动说明
1. 如果多块硬盘并存, 那么增加或者减少硬盘的时候, 有可能会发生识别顺序漂移的现象, 例如上一次启动的时候, 启动盘是/dev/sda, 插上一块新硬盘后, 启动盘变成了/dev/sdb, 如果还是从/dev/sda去启动, 这种漂移就有可能会造成启动失败.
2. 另外, 由于B/CEX01A的CF卡是通过USB模式挂载的, 在硬盘和CF卡同时存在的情况下, 在debian下, 也有可能发生挂载次序的漂移问题.
3. 解决这个问题的办法是通过UUID.
4. 第1步, 获取启动盘的UUID. 执行：
```bash
blkid
# 会有类似下面的输出:
/dev/sda1: UUID="4fba3652-ac13-4086-97d4-3356537732f6" TYPE="ext4"
/dev/sdb1: UUID="45559bc9-add1-4d6a-bd53-8ae0b5b043e3" TYPE="ext4"
# 记录UUID之后的内容, 如上例所示, 本机的root盘是/dev/sda1, 那么就记录下4fba3652-ac13-4086-97d4-3356537732f6
```
5. 第2步, 重启设备, 进入uboot. (进入uboot的方法是在串口连接的情况下, 在uboot启动过程中敲打键盘)
6. 第3步, 在uboot下输入如下命令:
```bash
FT1500a# setenv bootargs console=ttyS1,115200 earlyprintk=uart8250-32bit,0x28001000 root=UUID=4fba3652-ac13-4086-97d4-3356537732f6 initrd=0x95000000,16M rootwait rw KEYBOARDTYPE=pc KEYTABLE=us;

FT1500a# saveenv
```
**注意**：上面的命令就是把原先```root=/dev/sda1```替换成```root=UUID=4fba3652-ac13-4086-97d4-3356537732f6```。 如果你的root盘类似/dev/sdb1, 请用对应的UUID替换。

上述操作完成后, linux将始终使用指定UUID的硬盘分区作为root分区, 不论是否有新硬盘或者CF卡增加或者减少. 但是缺点也很明显, 如果你要更换root硬盘, 需要先获取新硬盘root分区的UUID, 并重复上面的步骤.
